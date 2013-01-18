### Getting Started

Triage is a web-based application written in Ruby on Rails. It requires a web server, database, and a recent version of Ruby.

#### Requirements

##### POSIX

Triage will run on a variety of platforms, including various flavours of Linux, BSD, and Mac OS X. In other words, Triage doesn't run on Windows. If you're willing to get your hands dirty, you can modify Triage to run on a Windows-friendly web server without too much trouble, but that time would probably be better spent learning how to use Linux. Install a recent distro, follow the succeeding installation instructions, and thank me later.

##### Ruby

Triage is tested and support on Ruby 1.9.3. It's likely that it will work on Ruby 1.9.2, but Rails 4 is approaching and is only supported on 1.9.3, so it's best to stick with that. If you plan on running multiple version of ruby on your application server, [rbenv](https://github.com/sstephenson/rbenv) is an excellent tool for doing just that.

##### Database

Triage supports MySQL, Postgres, and SQLite, although SQLite is only recommended for development.

##### Libraries

Many of the libraries used by Triage have native library dependencies. Please install the following development packages before attempting to install Triage's dependencies using Bundler (you'll get compile errors if they're missing):

* make
* SSL (libssl-dev)
* XML (libxml2-dev)
* XSLT (libxslt1-dev)
* MySQL (libmysqlclient-dev)
* PostgreSQL (libpq-dev)
* SQLite3 (libsqlite3-dev)

You can install all of these libraries in one shot using an apt-based package manager as follows (you'll only need one of the last three arguments):

`sudo apt-get install make libssl-dev libxml2-dev libxslt1-dev libmysqlclient-dev libpq-dev libsqlite3-dev`

### Basic Setup

#### Bundler

Once Ruby is installed, you'll need to install [Bundler](gembundler.com) to get Triage's dependencies:

    $ gem install bundler

#### Installing Dependencies

The following command will install Triage's dependencies. The last 3 arguments indicate which database adapters _not_ to install; make sure to remove one of these for whichever database you plan to use.

    $ bundle install --path vendor/bundle --without development test sqlite mysql postgresql

If you get errors after running this command, it's likely that you're missing development headers for one or more of the gems (libraries) used by Triage. Read the error messages, they'll give you insight into which headers you're missing.

#### Database Configuration

##### Environment Variables

Set the `DB` environment variable to `sqlite`, `mysql`, or `postgresql`, depending on which database you plan on using. Also, set the `RAILS_ENV` variable to `production` to indicate that we want to run Triage in production mode.

##### Configure `database.yml`

    sqlite: &sqlite
      adapter: sqlite3
      database: db/<%= ENV['RAILS_ENV'] %>.sqlite3

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

    production:
      <<: *defaults

If you're using SQLite, there's nothing to do. Otherwise, modify either the postgresql or mysql section with the connection details for your database. If the database is hosted on a separate server, make sure to specify a host (default is localhost).

Once the database is configured, you're ready to set it up with some initial tables and seed data. Run the following command to do just that:

    bundle exec rake db:setup

Provided the configuration provided in `config/database.yml` is correct, the database will be setup and ready to go.

### Production Deployment

#### Running a Development Version

If you've decided to deploy a development version of Triage (technically speaking, anything not explicitly tagged as a release on the repository or offered as download), you'll need a JavaScript runtime on your server. Triage makes use of interpreted languages that depend on a JS runtime for realtime compilation; these resources are pre-compiled for official releases, but you'll need to compile them manually when running a development release.

Install v8 (Google's JS runtime) and a C++ compiler:

    sudo apt-get install libv8-dev g++

add the following line to the `Gemfile` located in the root of the app:

    gem 'therubyracer'

Make sure to put this in the development group (alongside the debugger gem). You'll need to re-run the command in _Installing Dependencies_ to force Bundler to re-read the Gemfile and install `therubyracer`. Once you've done that, run the following command to pre-compile all of Triage's assets:

    bundle exec rake assets:precompile

#### Nginx

[Nginx](http://nginx.org) is the recommended web server for Triage. It serves all of the static assets (images, stylesheets, etc.) directly, passing application requests on to Triage (well, technically speaking, to [Unicorn](http://unicorn.bogomips.org/), but we'll delve into that later).

Install nginx using your system's package manager:

    $ sudo apt-get install nginx
