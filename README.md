TODO : Add travis configuration, and then badge, eventually gemfury badge too.

# SeedMigration

Harry's Data Migrations are a way to manage changes to seed data in a rails app in a similar way to how schema migrations are handled.


## Intro
A data migration library, similar to rails built-in schema migration. It also auto generates a `db/seeds.rb` file, similar to how schema migrations generate the `db/schema.rb` file.
Using this auto generated seed file makes it quick and easy to setup new environments, usually development or test.

## Installation

Add `gem 'seed_migration'` to your `Gemfile`:

```ruby
gem 'seed_migration', :github => 'harrystech/seed_migration'
```

Note : It'll soon be released on rubygems.org.

## Usage

### Install and run the internal migrations

```ruby
rake seed_migration:install:migrations
rake db:migrate
```

That will create the table to keep track of data migrations.

### Generate a new migration

`SeedMigration` adds a new rails generator :

```ruby
rails g seed_migration AddFoo
```
A new file will be created under `db/data/` using rails migration convention:

```
db/data/20140407162007_add_foo.rb
```

You'll need to implement the `#up` method and if you need to be able to rollback, the `#down` method.

### Running the migrations

To run all pending migrations, simply use

```ruby
rake data:migrate
```

If needed, you can run a specific migration:

```ruby
rake data:migrate MIGRATION=20140407162007_add_foo.rb
```

### Rollback

Rolling back the last migration is as simple as:

```ruby
rake data:rollback
```

You can rollback more than one migration at the same time:

```ruby
rake data:rollback STEPS=3 # rollback last 3 migrations
```

Or rollback a specific migration:

```ruby
rake data:rollback MIGRATION=20140407162007_add_foo.rb
```

### Registering models

By default, `SeedMigration` won't seen any data after running `data:migrate`. You have to manually register the models in the configuration file.

Simply register a model:

```ruby
SeedMigration.register Product
```

You can customize the 'seeded' attribute list:

```ruby
SeedMigration.register User do
  exclude :id, :password
end
```

This will create a `seeds.rb` containing all User and Product in the database:

```ruby
# encoding: UTF-8
# This file is auto-generated from the current content of the database. Instead
# of editing this file, please use the migrations feature of Seed Migration to
# incrementally modify your database, and then regenerate this seed file.
#
# If you need to create see the database on another system, you should be using
# db:seed, not running all the migrations from scratch. The latter is a flawed
# and unsustainable approach (the more migrations you'll amass, the slower
# it'll run and the greater likelihood for issues).
#
# It's strongly recommended to check this file into your version control system.

ActiveRecord::Base.transaction do
  Product.create("id"=>1, "name"=>"foo", "created_at"=>"2014-04-04T15:42:24Z", "updated_at"=>"2014-04-04T15:42:24Z")
  Product.create("id"=>2, "name"=>"bar", "created_at"=>"2014-04-04T15:42:24Z", "updated_at"=>"2014-04-04T15:42:24Z")
  # ...
  User.create("id"=>1, "name"=>"admin", "created_at"=>"2014-04-04T15:42:24Z", "updated_at"=>"2014-04-04T15:42:24Z")
  # ...
end

SeedMigration::Migrator.bootstrap(20140404193326)
```


### Deployment notes

It is recommended to add the `rake data:migrate` to your deploy script, so each new data migrations is ran upon new deploys.
You can enable the `extend_native_migration_task` option to automatically run `rake data:migrate` after `rake db:migrate`.

## Example

```ruby
rails g seed_migration AddADummyProduct
```

```ruby
class AddADummyProduct < SeedMigration::Migration
    def up
        Product.create!({
            :asset_path => "valentines-day.jpg",
            :title => "Valentine's Day II: the revenge!",
            :active => false,
            :default => false,
        }, :without_protection => true)
    end

    def down
        Product.destroy_all(:title => "Valentine's Day II: the revenge!")
    end
end
```

## Configuration

`SeedMigration` can be configured using an initializer file.

### List of available configurations :

- `extend_native_migration_task (default=false)`
- `pending_migrations_warning_level (default=:warn)`
- `ignore_ids (default=false)`
- `migration_table_name (default='seed_migration_data_migrations')`: Override the table name for the internal model that holds the migrations

#### example:

```ruby
# config/initializers/seed_migration.rb

SeedMigration.config do |c|
    c.migration_table_name = 'data_migrations'
    c.extend_native_migration_task = true
    c.pending_migrations_warning_level = :error
end

SeedMigration.register User do
  exclude :id, :password
end
SeedMigration.register Product
```

## Compatibility

At the moment, we rely by default on

```
ActiveRecord::Base.connection.reset_pk_sequence!
```
which is `pg` only.

If you need to use this gem with another database, use the `ignore_ids` configuration.


## Runnings tests


```ruby
RAILS_ENV=test bundle exec rake app:db:reset
bundle exec rspec spec
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
