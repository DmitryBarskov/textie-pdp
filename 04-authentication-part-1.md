# Authentication. Part 1

We'll need some extra gems now.
Add them to Gemfile if you didn't.

```Gemfile
gem "interactor"
gem "jwt"
gem "rack-cors"
```

We will use JWT authentication, so we need the gem.
We'll going to set up access from other domains in this lesson with `rack-cors`.
There is an architecture design pattern called "transaction-script".
We'll implement it using "interactor" gem.

## Setting up variables

We don't want to set environment variables manually every time we start development server or run tests.
There is a gem to automate that - [dotenv-rails](https://github.com/bkeepers/dotenv).
Add it to `Gemfile` in development & test section
(see [the first part of this tutorial](/01-how-to-create-rails-api-application.md#configure-basic-gems)).
Create `.env` file and put `JWT_SECRET=some-default-jwt-token` there.

`.env` can contain sensetive information, so keep it private.
Create `.env.example` with variables defintions and checkout it to your repository instead.
Add public settings and settings for local development there.
```bash
# file: .env.example
DATABASE_URL=postgresql://localhost:5432/textie_development
JWT_SECRET=some-default-jwt-token
```

When deployed to a server the app in the production mode takes these settings from real environment variables.

## Create JWT Codec

Let's create a class that encodes/decodes JWT for us. It should have 2 methods:

* encode - the method takes a hash and returns a string
* decode - the method takes some string produced by method `encode` and return the hash, originally passed to method `encode`

The methods uses `JWT` module from the gem.

```ruby
# file: app/models/jwt_codec.rb
class JwtCodec
  def initialize(secret = ENV.fetch("JWT_SECRET"))
    @secret = secret
  end

  def encode(payload)
    JWT.encode(payload, @secret)
  end

  def decode(token)
    JWT.decode(token, @secret, true, algorithm: "HS256")[0]
  rescue JWT::DecodeError
    nil
  end
end
```

# TODO

## Add routes

For authentication we'll use route `POST /api/v1/sessions`.

```ruby
resources :sessions, only: %i[create]
resource :profile, only: %i[show]
```

`/api/v1/profile` is a private route, we'll test our authentication with it.

## Create SessionsController

```ruby
# file: app/controllers/api/v1/sessinos_controller.rb
module Api
  module V1
    class SessionsController < ApplicationController
      def create
        result = LoginUser.call(session_params)

        if result.success?
          render json: { token: result.token }
        else
          render json: { error: result.error }, status: :unauthorized
        end
      end

      private

      def session_params
        params.permit(:email, :password)
      end
    end
  end
end
```

We have email and password parameters for creating session (i. e. logging in).
Client receives JSON Web Token, stores it and sends it with each request.
By this token we can recognize the user.

To create session we only call `LoginUser` interactor with email and password params,
handle the errors of the result.
Client will receive JWT as a response.

## Creating interactors for logging in users

Let's see `LoginUser` interactor. It's only aggregates other interactors.
First, we need to find the user by given email, then we pass the found user to `AuthenticateByPassword` interactor, if authentication succeeds then we create JWT which is taken from `result` variable in our contorller.

```ruby
# file: app/interactors/login_user.rb
class LoginUser
  include Interactor::Organizer

  organize LoginUser::FindUserByEmail,
           LoginUser::AuthenticateByPassword,
           LoginUser::GenerateJwt
end
```

Other interactors are very simple:
Here we find user by given email and fail the interactor if no user was found.
Interactor failure causes all the chain to fail, it won't be executed further.

```ruby
# file: app/interactors/login_user/find_user_by_email.rb
class LoginUser
  class FindUserByEmail
    include Interactor

    delegate :email, to: :context

    def call
      context.user = User.find_by(email: email)
      fail! unless context.user
    end

    private

    def fail!
      context.fail!(error: I18n.t("error.user_not_found", email: email))
    end
  end
end
```

```ruby
# file: app/interactors/login_user/authenticate_by_password.rb
class LoginUser
  class AuthenticateByPassword
    include Interactor

    delegate :user, :password, to: :context

    def call
      context.user = user.authenticate(password)
      fail! unless user
    end

    private

    def fail!
      context.fail!(error: I18n.t("error.invalid_credentails"))
    end
  end
end
```

```ruby
# file: app/interactors/login_user/generate_jwt.rb
class LoginUser
  class GenerateJwt
    include Interactor

    delegate :user, to: :context

    def call
      context.token = JwtCodec.new.encode(user_attributes)
    end

    private

    def user_attributes
      {
        id: user.id,
        email: user.email,
        full_name: user.full_name
      }
    end
  end
end
```

## Recognizing user by JWT

Create an interactor first.
Here we have top level interactor definition.
We want top-level interactors to be abstract and interactors in modules to be concrete.

```ruby
# file: app/interactors/recognize_user.rb
class RecognizeUser
  include Interactor::Organizer

  organize RecognizeUser::AuthenticateByJwt
end
```

In fact we have an interactor RecognizeUser which is just an alias for AuthenticateByJwt.


```ruby
# file: app/interactors/recognize_user/authenticate_by_jwt.rb
class RecognizeUser
  class AuthenticateByJwt
    include Interactor

    delegate :token, to: :context

    def call
      user_attributes = JwtCodec.new.decode(token)
      fail! unless user_attributes
      context.user = User.new(user_attributes)
    end

    private

    def fail!
      context.fail!(error: I18n.t("error.invalid_jwt"))
    end
  end
end
```

Here we just use our JwtCodec class and fail the authentication process when it fails to decode jwt.

## ApplicationController

It is the base class for all our contorllers.
We add filter `:authenticate_user!` that will run every time a controller receives a request. In the filter we just invoke our RecognizeUser interactor and handle its error.
If it succeeds we set current_user attribute and expose it with attr_reader. So all the controllers can access current_user.
