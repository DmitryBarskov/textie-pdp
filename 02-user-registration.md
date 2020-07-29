# 02. User registration

## Configure database

I've removed all the comments and parameters (except for default profile) from `config/database.yml`.
And added single url parameter to default profile.

```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  url: <%= ENV.fetch("DATABASE_URL") %>

development:
  <<: *default

test:
  <<: *default

production:
  <<: *default
```

Now we can configure connection to database by setting `DATABASE_URL` environment variable.

## Setting up variables for development

We don't want to set environment variables manually every time we start development server or run tests.
There is a gem to automate that - [dotenv-rails](https://github.com/bkeepers/dotenv).
Add it to `Gemfile` in development & test section (see [the first part of this tutorial](/01-how-to-create-rails-api-application.md#configure-basic-gems)).
Create `.env` file and put `DATABASE_URL=postgresql://127.0.0.1:5432/textie_development` there.

Probably you need to add your username and password to this url, for example:
```bash
DATABASE_URL=postgresql://user:password@127.0.0.1:5432/textie_development
```

`.env` can contain sensetive information, so keep it private.
Create `.env.example` with variables defintions and checkout it to your repository instead.
Add public settings and settings for local development there. 
```bash
# file: .env.example
DATABASE_URL=postgresql://localhost:5432/textie_development
```

## Create user model

Generate `models/user.rb` and a migration file.

```bash
rails g model User email:citext:uniq full_name password_digest:string
```

`citext` stands for case-insensitive text.
It is a special type in PostgreSQL that behaves as text
but it is processed with `LOWER` function before comparisons.

To use it you need to enable it.
In generated migration file add `enable_extension("citext")` before `create_table` statement.

```ruby
# file: db/migrate/yyyymmddhhmmss_create_users.rb
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    enable_extension("citext")

    create_table :users do |t|
      t.citext :email, null: false
      # ...
```
