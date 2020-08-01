# Authentication

We'll need some extra gems now.
Add them to Gemfile if you didn't.

```Gemfile
gem "interactor"
gem "jwt"
gem "rack-cors"
```

We're going to organize our business logic in interactors.
For authentication we'll use JWT and `rack-cors` to allow requests from other hosts.

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
