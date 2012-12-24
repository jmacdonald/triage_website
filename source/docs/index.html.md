### Getting Started

Triage is a web-based application written in Ruby on Rails. It requires a web server, database, and a recent version of Ruby.

#### Requirements

##### Ruby

Triage is tested and support on Ruby 1.9.3. It's likely that it will work on Ruby 1.9.2, but Rails 4 is approaching and is only supported on 1.9.3, so it's best to stick with that. If you plan on running multiple version of ruby on your application server, [rbenv](https://github.com/sstephenson/rbenv) is an excellent choice.

##### Database

Triage supports MySQL, Postgres, and SQLite, although SQLite is only recommended for development.

### Basic Setup

#### Bundler

Once Ruby is installed, you'll need to install [Bundler](gembundler.com) to get Triage's dependencies:

    $ gem install bundler

#### Database Selection

Now that bundler is installed, extract Triage into a serving directory and `cd` into it. Before installing all of the dependencies, you'll need to decide which database you want to use. Edit `Gemfile` and look for the database section:

    # Database Adapters
    ######################################

    # PostgreSQL
    gem 'pg'

    # MySQL
    # gem 'mysql2'

    # SQLite
    # gem 'sqlite3'

    #######################################

Make a point to enable only those database adapters that you plan on using. You'll need development headers for those that you leave enabled, so make sure to install those manually or using your system's package manager.

#### Installing Dependencies

Run the following command to fetch all of Triage's dependencies:

    $ bundle install

If you get errors after running this command, it's likely that you're missing development headers for one or more of the gems (libraries) used by Triage. Read the error messages, they'll give you insight into which headers you're missing.

#### Database Configuration

The last step before starting Triage is to configure the database, which is handled by `config/database.yml`:

    sqlite: &sqlite
      adapter: sqlite3
      database: db/test.sqlite3

    mysql: &mysql
      adapter: mysql2
      username: root
      password:
      database: triage_test

    postgresql: &postgresql
      adapter: postgresql
      username: postgres
      password:
      database: triage_test
      min_messages: ERROR

    defaults: &defaults
      pool: 5
      timeout: 5000
      host: localhost
      <<: *<%= ENV['DB'] || "postgresql" %>

    development:
      adapter: postgresql
      username: postgres
      password:
      database: triage_development
      min_messages: ERROR

    test:
      <<: *defaults
