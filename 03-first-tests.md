# First tests

## Setting up our tests

For our tests we will need [FactoryBot](https://github.com/thoughtbot/factory_bot)
and [FFaker](https://github.com/ffaker/ffaker).

Set up factory for users:
```ruby
# file: spec/factories/users.rb
FactoryBot.define do
  factory :user do
    sequence(:email) { |n| "user_#{n}@example.com" }
    full_name { FFaker::Name.name }
    password { "123456" }
  end
end
```
Here we use FFaker to generate some plausible names.

Add this line
```ruby
config.include FactoryBot::Syntax::Methods
```
to RSpec configuration, so it's possible to use FactoryBot methods in tests.

## Testing registration

### Positive branch

Let's test our registration feature.
First we describe positive case when registration is successful.
```ruby
RSpec.describe "Api::V1::Users", type: :request do
  describe "POST /api/v1/users" do
    context "with valid attributes" do
      let(:valid_attributes) { attributes_for(:user) }
      let(:request) { post api_v1_users_url, params: { user: valid_attributes } }

      it "creates user" do
        expect { request }.to change(User, :count).by(1)
      end
    end
  end
end
```
We receive `valid_attributes` from our `:users` factory
(`attributes_for` is a FactoryBot method).
Then we describe how to send a request.
In our spec itself we send request and check that user is created.

## Negative branch

Create a context for it inside our `describe "POST ..."` block.
```ruby
context "with invalid attributes" do
  let(:request) { post api_v1_users_url, params: { user: invalid_attributes } }
  let(:body) { JSON.parse(response.body) }
end
```

Then create different context inside for each type of error.
```ruby
context "when email already taken" do
  let(:invalid_attributes) { attributes_for(:user, email: "taken@email.com") }

  before { create(:user, email: "taken@email.com") }

  it "responds with errors" do
    request
    expect(body).to match("email" => [include("taken")])
  end
end

context "when no email provided or invalid email" do
  let(:invalid_attributes) { attributes_for(:user, email: "") }

  it "responds with errors" do
    request
    expect(body).to match("email" => [include("blank"), include("invalid")])
  end
end

context "when no password provided" do
  let(:invalid_attributes) { attributes_for(:user, password: "") }

  it "responds with errors" do
    request
    expect(body).to match("password" => [include("blank")])
  end
end
```

It is quite simple. Here you can see that we can use nested RSpec matchers.
