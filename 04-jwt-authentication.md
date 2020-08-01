# JWT authentication

## Install dependencies

We'll need extra gems here:
```ruby
gem "rack-cors"

gem "jwt"
gem "interactor", "~> 3.1.0"
```

We will use JWT authentication, so we need the gem.
We'll going to set up access from other domains in this lesson with `rack-cors`.
There is an architecture design pattern called "transaction-script".
We'll implement it using "interactor".
