# Sinatra Activerecord Setup

## Objectives

- Setup a database in a Sinatra application.
- Create and use a Rakefile to run ActiveRecord migrations.
- Use ActiveRecord in a Sinatra application.

## Overview

Sinatra doesn't come with database support out of the box, but it's relatively
easy to configure. In general, we'll be working from templates that have this
pre-built, but it's good to understand what's going on under the hood. We're
going to practice adding a database to our Sinatra applications.

## Instructions

Fork and clone this repository to get started! We have a basic sinatra
application stubbed out with an `app.rb` file acting as the controller.

### Adding Your Gems

Currently, a few gems are already present in our Gemfile:

```ruby
gem 'sinatra'
gem 'thin'
gem 'require_all'


group :development do
	gem 'shotgun'
	gem 'pry'
end
```

To get Sinatra working with ActiveRecord, First, we'll add three gems to allow
us to _use_ ActiveRecord: `activerecord` version `5.2`, `sinatra-activerecord`,
and `rake`. The `activerecord` gem gives us access to the magical database
mapping and association powers. The `rake` gem, short for "ruby make", is a
package that lets us quickly create files and folders, and automate tasks such
as database creation, and the `sinatra-activerecord` gem gives us access to some
awesome Rake tasks. Make sure those three gems are added in your Gemfile:

```ruby
  gem 'sinatra'
  gem 'thin'
  gem 'require_all'
  gem 'activerecord', '5.2'
  gem 'sinatra-activerecord'
  gem 'rake'
```

Into our development group, we'll add two other gems: `sqlite3` and `tux`.
`sqlite3` is our database adapter gem - it's what allows our Ruby application to
communicate with a SQL database. `tux` will give us an interactive console that
pre-loads our database and ActiveRecord relationships for us. Since we won't use
either of these in production, we put them in our `:development` group - this
way, they won't get installed on our server when we deploy our application.

```ruby
  gem 'sinatra'
  gem 'thin'
  gem 'require_all'
  gem 'activerecord', '5.2'
  gem 'sinatra-activerecord'
  gem 'rake'
#Stuff to install but not to be deployed.
  group :development do
    gem 'shotgun'
    gem 'pry'
    gem 'tux'
    gem 'sqlite3', '~> 1.3.6'
  end
```

Our Gemfile is up to date - awesome! Go ahead and run `bundle install` to get
your system up to speed.

### Connecting to the Database

We now have access to all of the gems that we need, but we still need to set up
a connection to our database. Add the following block of code to your
`environment.rb` file (underneath `Bundler.require(:default,
ENV['SINATRA_ENV'])`).

```ruby
configure :development do
  set :database, 'sqlite3:db/database.db'
end
```

This sets up a connection to a sqlite3 database named "database.db", located in
a folder called "db." If we wanted our `.db` file to be called `dogs.db`, we
could simply change the name of this file:

```ruby
configure :development do
  set :database, 'sqlite3:db/dogs.db'
end
```

But for now, `database.db` is a great name. Notice that this didn't actually
create those files or folders yet - that's how Rake will help us.

### Making a Rakefile

As we mentioned, `rake` gives us the ability to quickly make files and set up
automated tasks. We define these in a file called `Rakefile`. First, create a
`Rakefile` in the root of our project directory. In the `Rakefile`, we'll
require our `config/environment.rb` file to load up our environment, as well as
`"sinatra/activerecord/rake"` to get Rake tasks from the `sinatra-activerecord`
gem.

```ruby
require './config/environment'
require 'sinatra/activerecord/rake'
```

In the terminal, type `rake -T` to view all of the available rake tasks. You
should see the following output:

```bash
rake db:create              # Creates the database from DATABASE_URL or config/database.yml for...
rake db:create_migration    # Create a migration (parameters: NAME, VERSION)
rake db:drop                # Drops the database from DATABASE_URL or config/database.yml for t...
rake db:fixtures:load       # Load fixtures into the current environment's database
rake db:migrate             # Migrate the database (options: VERSION=x, VERBOSE=false, SCOPE=blog)
rake db:migrate:status      # Display status of migrations
rake db:rollback            # Rolls the schema back to the previous version (specify steps w/ S...
rake db:schema:cache:clear  # Clear a db/schema_cache.dump file
rake db:schema:cache:dump   # Create a db/schema_cache.dump file
rake db:schema:dump         # Create a db/schema.rb file that is portable against any DB suppor...
rake db:schema:load         # Load a schema.rb file into the database
rake db:seed                # Load the seed data from db/seeds.rb
rake db:setup               # Create the database, load the schema, and initialize with the see...
rake db:structure:dump      # Dump the database structure to db/structure.sql
rake db:structure:load      # Recreate the databases from the structure.sql file
rake db:version             # Retrieves the current schema version number
```

### Testing it Out

Let's test out our handiwork by creating a `dogs` table with two columns: `name`
and `breed`. First, let's create our migration:

```bash
rake db:create_migration NAME=create_dogs
```

You should see the following output:

```bash
=># db/migrate/20150914201353_create_dogs.rb
```

The beginning of the file is a timestamp - yours should reflect the time that
your `create_dogs` file was created! You've now created your first database
migration inside of the `db` folder.

Inside of the migration file, remove the default `change` method (we'll come
back to this), and add methods for `up` and `down`.

```ruby
class CreateDogs < ActiveRecord::Migration[5.2]
  def up
  end

  def down
  end
end
```

**Important:** When we create migrations with ActiveRecord, we must specify the
version we're using just after `ActiveRecord::Migration`. In this case, we're
using `5.2`, so all the examples here will show `ActiveRecord::Migration[5.2]`.
This version may differ depending on the lab. If this number does not match
the version in your `Gemfile.lock`, your migration will cause an error.

Our `up` method should create our table with `name` and `breed` columns. Our
down method should drop the table.

```ruby
class CreateDogs < ActiveRecord::Migration[5.2]
  def up
    create_table :dogs do |t|
      t.string :name
      t.string :breed
    end
  end

  def down
    drop_table :dogs
  end
end
```

Now, run the migration from the terminal with `rake db:migrate`.

```bash
rake db:migrate SINATRA_ENV=development
```

Why add `SINATRA_ENV=development`, you might ask? Well, remember the top line of
`config/environment.rb`? It's telling Sinatra that it should use the
"development" environment for both `shotgun` and the testing suite. Therefore,
we want to make sure our migrations run on the same environment as well, and
specifying `SINATRA_ENV=development` allows us to do that.

You should see the following output:

```bash
== 20150914201353 CreateDogs: migrating =======================================
-- create_table(:dogs)
   -> 0.0019s
== 20150914201353 CreateDogs: migrated (0.0020s) ==============================
```

#### The `change` Method

The change method is actually a shorter way of writing `up` and `down` methods.
We can refactor our migration to look like this:

```rb
class CreateDogs < ActiveRecord::Migration[5.2]
  def change
    create_table :dogs do |t|
      t.string :name
      t.string :breed
    end
  end

end
```

While the rollback (`down`) method is not included, it's implicit in the change
method. Rolling back the database would work in exactly the same way as using
the `down` method.

<p class='util--hide'>View <a href='https://learn.co/lessons/sinatra-activerecord-setup'>ActiveRecord Setup in Sinatra</a> on Learn.co and start learning to code for free.</p>
