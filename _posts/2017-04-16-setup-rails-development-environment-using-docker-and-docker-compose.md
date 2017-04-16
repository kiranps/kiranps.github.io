---
layout: post
title:  "Setup Rails Development Environment Using Docker and Docker Compose"
description: "Development Environment for Rails in Docker (with PostgreSQL, Redis)"
date:   2017-04-16
categories: Docker Docker-Compose Rails PostgreSQL 
---

Using Docker Compose is basically a three-step process.

1. Define your app’s environment with a `Dockerfile`.

2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.

3. Lastly, run `docker-compose up` and Compose will start and run your entire app.


#### Create a Dockerfile

Let’s start by creating a `Dockerfile` in your Rails app directory which specifies what needs to be included in the container

{% highlight dockerfile %}
FROM ruby:2.4.1
# for postgres client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ jessie-pgdg main' >> /etc/apt/sources.list
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev postgresql-client-9.6
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
ADD Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
ADD . /myapp
{% endhighlight %}

#### Setting up Docker Compose

Now we have created a docker image for our Rails app. We need to add other services used by rails app, for that we will use docker compose. 

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a Compose file to configure your application’s services and link them together. 

Add a `docker-compose.yml` file to your repository and include the following configuration:

{% highlight yaml %}
version: '2'
services:
  db:
    image: postgres
    volumes:
      - postgres:/var/lib/postgresql/data
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
volumes:
  postgres: {}
{% endhighlight %}


In the above configuration we defined two containers. The first is db, which is a pre-made PostgreSQL image. Second is web, which is built from the current directory

Now we need to build our web container for that run

    docker-compose build

You need rebuild the container whenever there is a change in Gemfile or Dockerfile


#### Configure the database

We need to configure rails database config, so that it can connect to the db container defined in docker compose file

Update the contents of config/database.yml with the following:

{% highlight yaml %}
development: &default
  adapter: postgresql
  encoding: unicode
  database: myapp_development
  pool: 5
  username: postgres
  password:
  host: db

test:
  <<: *default
  database: myapp_test
{% endhighlight %}

You need to create the database. In terminal run

    docker-compose run web rails db:create

#### Starting the App

You can simply start the app by running:

    docker-compose up

It wil start your rails app on port 3000 and PostgreSQL Server

#### Other commands

Start rails console by runninng:

    docker-compose run web rails console

If you want to open dbconsole run:

    docker-compose run web rails dbconsole

#### Conclusion

Using a Docker development Environment helps to run you app and services in an isolated environment. Docker compose helps to configure all of the application’s service dependencies (databases, queues, caches, web service APIs, etc). You can start the app and other services in a single command. You can easily share your development enviroment with your team members, which makes very easy to get started on a project.

#### Resources

* Dockerfile : [link][Dockerfile]
* Docker Compose : [link][Compose]


[Dockerfile]: https://docs.docker.com/engine/reference/builder/
[Compose]: https://docs.docker.com/compose/compose-file/
