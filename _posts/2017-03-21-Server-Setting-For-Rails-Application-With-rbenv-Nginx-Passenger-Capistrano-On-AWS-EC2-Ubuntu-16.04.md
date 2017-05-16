---
layout:   post
title:    "Server Setting For Rails Application With rbenv, Nginx, Passenger, Capistrano On AWS EC2 Ubuntu 16.04"
date:     2017-03-21 23:26:18 +0900
tags:     aws rails
author:   junwoo
comments: true
---

## 1. Install ruby using rbenv

### 1.1. Install rbenv

Install some dependencies for Ruby.

```console
$ sudo apt update
$ sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev nodejs
```

Install rbenv and ruby-build plugins.

```console
$ cd
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
$ exec $SHELL

$ git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
$ echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
$ exec $SHELL
```

### 1.2. Install ruby & bundler gem

Install ruby.

```console
$ rbenv install -v [RUBY VERSION] # ex) rbenv install -v 2.3.3
$ rbenv global [RUBY VERSION]
$ ruby -v # Check Ruby version
```

Install bundler gem.

```console
$ echo "gem: --no-document" > ~/.gemrc
$ gem install bundler
```

## 2. Configure capistrano & Run deploy __(Client side)__
### 2.1. Install the Capistrano gem

Add Capistrano to your Gemfile:

```ruby
# Gemfile
group :development do
  gem 'capistrano'
  gem 'capistrano-bundler'
  gem 'capistrano-rails'
  gem 'capistrano-passenger'
  gem 'capistrano-rbenv'
  gem 'capistrano-linked-files'
end
```

Then run Bundler to ensure Capistrano is downloaded and installed:

```console
$ bundle && bundle exec cap install
```

### 2.2. Configure Capfile
Find the following lines, and uncomment them:
```ruby
require "capistrano/rbenv"
set :rbenv_ruby, '2.3.3' # Add this line, with your ruby version
require "capistrano/bundler"
require "capistrano/passenger"
require "capistrano/linked_files"
```

### 2.3. Configure config/deploy.rb
Find the following lines, and uncomment them or add lines:
```ruby
set :application, "my_app_name" # Your app name
set :repo_url, "git@example.com:me/my_repo.git" # Your app's git repo url

set :deploy_to, "/home/ubuntu/my_app_name"

append :linked_files, "config/application.yml", "config/database.yml", "config/secrets.yml"
append :linked_dirs, "log", "tmp/pids", "tmp/cache", "tmp/sockets", "vendor/bundle", "public/system", "public/uploads", "public/assets"

set :keep_releases, 5
set :passenger_restart_with_touch, true
set :rails_env, :production

namespace :deploy do
  task :reset_db do
    on roles(:app), in: :groups, limit: 3, wait: 10 do
      within release_path do
        with rails_env: fetch(:rails_env) do
          execute :rake, "db:drop"
          execute :rake, "db:create"
          execute :rake, "db:migrate"
          execute :rake, "db:seed"
          # You can add additional rake task or some commands here
        end
      end
    execute :touch, release_path.join('tmp/restart.txt')
    end
  end
  task :assets do
    on roles(:app), in: :groups, limit: 3, wait: 10 do
      within release_path do
        with rails_env: fetch(:rails_env) do
          execute :rake, "assets:precompile"
        end
      end
      execute :touch, release_path.join('tmp/restart.txt')
    end
  end
end
```

### 2.4. Configure deploy/production.rb
Find the following lines, and uncomment them:
```ruby
# deploy/production.rb
server 'example.com', # or Public IP
  user: 'ubuntu',
  roles: %w{app},
  ssh_options: {
    keys: %w(~/.ssh/example.pem), # Pem key file path on your local
    forward_agent: true, # See 0.4.1. below this codes
    auth_methods: %w(publickey)
  }
```

#### 2.4.1. SSH Agent Forwarding
SSH Agent Forwarding is a great way to keep SSH keys manageable as it allows the deployment server to use your own local private key to authenticate to the git repository, instead of having to give your deployment server access to your git repository.

* Only for macOS using keychain

```console
$ ssh-add -K
```

* All OS

Open up the file at ~/.ssh/config. If this file doesn't exist, you can create it by entering touch ~/.ssh/config in the terminal.
Enter the following text into the file, replacing example.com with your server's domain name or IP:

```console
 Host example.com 
  ForwardAgent yes 
```


### 2.5. Upload linked_files using capistrano-linked-files

```console
$ bundle exec cap production linked_files:upload_files
```

## 3. Passenger & nginx

### 3.1. Install passenger & nginx

Install some dependencies.

```console
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
$ sudo apt install -y apt-transport-https ca-certificates
```

Add Passenger APT repository.

```console
$ sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger xenial main > /etc/apt/sources.list.d/passenger.list'
$ sudo apt update
```

Install Passenger & Nginx

```console
$ sudo apt install -y nginx-extras passenger
$ sudo service nginx start
```

### 3.2. Configure passenger

```console
$ sudo nano /etc/nginx/passenger.conf
```

Change the passenger_ruby line to point to your ruby executable

```config
# /etc/nginx/passenger.conf
passenger_ruby /home/ubuntu/.rbenv/shims/ruby;
```

### 3.3. Configure nginx

```console
$ sudo nano /etc/nginx/nginx.conf
```

Find the following lines, and uncomment them:

```config
# /etc/nginx/nginx.conf
##
# Phusion Passenger
##
# Uncomment it if you installed ruby-passenger or ruby-passenger-enterprise
##

include /etc/nginx/passenger.conf;
```

And add the nginx host

```console
$ sudo nano /etc/nginx/sites-enabled/default
```
Replace the file's contents with the following

```config
# /etc/nginx/sites-enabled/default
server {
        listen 80;
        listen [::]:80 ipv6only=on;

        server_name         example.com;
        passenger_enabled   on;
        rails_env           production;
        root                /home/ubuntu/[my_app_name]/current/public;

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```
Restart nginx.
```console
$ sudo service nginx restart
```


## 4. Install Mysql, Configure database.yml

Install Mysql.

```console
$ sudo apt-get install mysql-server mysql-client libmysqlclient-dev
```

Add gem to Gemfile:

```ruby
# Gemfile
gem 'mysql2'
```

Configure database.yml

```yml
# /home/ubuntu/my_app_name/shared/config/database.yml
production:
  adapter: mysql2
  host: 127.0.0.1
  database: [my_app_name]_production
  username: root
  password: [my_mysql_password]
  pool: 5
  timeout: 5000
```


## 5. Configure application.yml & secrets.yml using figaro

Add gem to Gemfile:

```ruby
gem 'figaro'
```

Generate 'SECRET KEY'

```console
$ rake secret RAILS_ENV=production
```

Append the 'SECRET KEY' to application.yml

```yml
SECRET_KEY_BASE: [SECRET KEY]
```


## 6. Run deploy __(Client side)__

All deploy method includes passenger restart

```console
$ bundle exec cap production deploy
$ bundle exec cap production deploy:reset_db
$ bundle exec cap production deploy:assets
```

Manually touch restart.txt

```console
$ bundle exec cap production passenger:restart
```
