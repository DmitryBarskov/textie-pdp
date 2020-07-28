# 02. User authentication

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

