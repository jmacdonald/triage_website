### Installation

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

If you get errors after running this command, it's likely that you're missing development libraries for one or more of Triage's dependencies. Read the error messages, they'll give you insight into which ones you're missing. After installing any missing libraries, you can re-run the aforementioned `bundle install` command.

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

#### LDAP

If you're planning on integrating Triage with an LDAP server for authentication, you'll need to configure that as well. To enable LDAP support, you'll need to set the `ldap_enabled` flag in `config/triage.yml`:

    defaults: &defaults
      ldap_enabled: true

Once that's complete, you'll need to setup `config/ldap.yml` with information relating to your LDAP server. You only need to modify the `production` section, unless you plan on contributing to Triage. Leave the `ssl` value as-is, but make sure to fill out the rest. If you're not sure what values to use, talk to your LDAP server administrator.

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

Once installed, you'll need to configure nginx by modifying `/etc/nginx/nginx.conf`. Here's a good starting configuration:

    worker_processes 1;
    user nobody nogroup;

    pid /tmp/nginx.pid;
    error_log /tmp/nginx.error.log;

    events {
      worker_connections 1024; # increase if you have lots of clients
      # Set this to on if you have more than 1 working processes
      # This will allow only one child to watch the pollset and accept
      # a connection to a socket
      accept_mutex off; # "on" if nginx worker_processes > 1
    }

    http {
      include mime.types;
      default_type application/octet-stream;
      access_log /tmp/nginx.access.log combined;

      # This tells Nginx to ignore the contents of a file it is sending
      # and uses the kernel sendfile instead
      sendfile on;

      # Set this to on if you have sendfile on
      # It will prepend the HTTP response headers before
      # calling sendfile()
      tcp_nopush on;

      # This disables the "Nagle buffering algorithm" (Nginx Docs)
      # Good for websites that send a lot of small requests that
      # don't need a response
      tcp_nodelay off;

      gzip on;
      gzip_http_version 1.0;
      gzip_proxied any;
      gzip_min_length 500;
      gzip_disable "MSIE [1-6]\.";
      gzip_types text/plain text/html text/xml text/css
                 text/comma-separated-values
                 text/javascript application/x-javascript
                 application/atom+xml;

      upstream unicorn_server {
       server "unix:/var/www/triage/tmp/unicorn.sock" fail_timeout=0;
      }

      server {
        listen 80;
        client_max_body_size 4G;
        server_name _;

        keepalive_timeout 5;

        # Location of our static files
        root "/var/www/triage/public";

        location / {
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header Host $http_host;
          proxy_redirect off;

          # If you don't find the filename in the static files
          # Then request it from the unicorn server
          if (!-f $request_filename) {
            proxy_pass http://unicorn_server;
            break;
          }
        }

        error_page 500 502 503 504 /500.html;
        location = /500.html {
          root "/var/www/triage/public";
        }
      }
    }

Change all of the references to `/var/www/triage/` if you've installed to a different path. Once this configuration is in place, (re)start nginx. Now Triage's static assets are being served and application requests will be forward to Unicorn, which we'll setup next.

#### Unicorn

Believe it or not, you've actually already installed Unicorn! It's specified as one of Triage's dependencies, so it's installed as part of the `bundle install` process. The only thing you need to do is edit `config/unicorn.rb` and change the `triage_directory` variable, if you've installed to a different path (note the lack of a trailing slash). Once that's complete, run the following command from the triage directory to start unicorn using the aforementioned configuration file:

    $ bundle exec unicorn_rails -c config/unicorn.rb

That's it! Triage is now installed and ready for [initial setup](/docs/initial_setup/).
