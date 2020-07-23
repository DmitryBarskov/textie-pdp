# 01. How to create Rails API application?

## Initialize an app first

Run `rails new` with options listed below:

```sh
rails new textie --api -d postgresql --skip-keeps -C -T
```

Arguments:

- `--api` minimal API application
- `-d postgresql` use PostgreSQL database
- `--skip-keeps` don't create `.keep` files in empty directories
- `-C` skips websockets support
- `-T` skips test folder, we'll create it manually

## Configure basic gems

- `bootsnap` speeds up server start by caching files
- `puma` a Ruby/Rack applications server
- `pg` PostgreSQL driver
- `bcrypt` password encryption
- `jbuilder` templates for JSON views

- `brakeman` static security analysis
- `bullet` N+1 queries detector
- `bundler-audit` gems security advisor
- `byebug` CLI debugger
- `dotenv-rails` loads environment variables from `.env` file
- `factory_bot_rails` factories for testing
- `ffaker` test data
- `rspec-rails` test framework
    
    Run `rails generate rspec:install` to create `rails_helper.rb`. Look through
    this file and enable prefferred settings. Also `spec_helper.rb` file is created.
    Perform the same procedure for it too.

- `rubocop*` static code analysers/linters

    I ran initialized `.rubocop.yml` with some Flatstack conventions,
    ignored some offences in `config` directory and added rules that
    Rubocop asked me to. After I ran `rubocop -a` to fix all the offences.

- `database_cleaner` cleans database between test runs
- `listen` notifies when files change
- `rails-erd` generates Entity Relationship Diagram (run `bundle exec erd`)
- `spring` preload application in development
- `spring-watcher-listen` makes Spring use Listen

## Ignore files in Git

I took https://github.com/fs/rails-base's `.gitignore` and merged it with default one.
