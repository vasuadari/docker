# Getting started with docker

## Contents

1. [Introduction](#user-content-introduction)
2. [Installation](#user-content-installation)
3. [Basics of docker](#user-content-basics-of-docker)
    - [docker-machine](#user-content-docker-machine)
    - [docker](#user-content-docker)
    - [Image](#user-content-image)
    - [Container](#user-content-container)
    - [Dockerfile](#user-content-dockerfile)
    - [Docker hub](#user-content-docker-hub)
4. [Dockerize Applications](#user-content-dockerize-applications)
    - [Rails on docker](#user-content-rails-on-docker)
    - [Rails application](#user-content-rails-application)
        - [Docker compose](#user-content-docker-compose)
        - [Creating Dockerfile](#user-content-creating-dockerfile)
        - [Creating docker-compose file](#user-content-creating-docker-compose-file)
        - [Start your application on docker](#user-content-running-your-application-on-docker)
    - Using Nginx to host your rails application (TODO)

## Introduction

Using docker you can wrap your application with all its dependencies into a [container](#user-content-container) by having your host machine(Think of a machine with freshly installed Ubuntu) clean.

## Installation

Follow below links to setup docker on your machine.

[Ubuntu](https://docs.docker.com/engine/installation/linux/ubuntulinux/)

[Mac OS X](https://docs.docker.com/engine/installation/mac/)

## Basics of docker

Before getting started with docker, you need to understand below terminologies.

### docker-machine

Docker follows client-server architecture. `docker-machine` is a command to create Virtual machine with a minimal linux os running on it. All the images that you build will be hosted on this server.

### docker

`docker` is a command to interact with your `docker-machine`. Think of it as a client which talks to a server(`docker-machine`) to get the work done.

### Image

It is a pre-built image like the ones you download for OSes which can be ran on the container.

### Container

Light weight virtual machine which runs your image.

### Dockerfile

It is a manifest file which is used for building your own images. This manifest file contains set of instructions which guides docker to run list of commands to setup your application. Docker rebuilds the image only if it finds any new instruction after the first build.

Below is the sample Dockerfile

```
# Filename: Dockerfile
# Pull image from the docker hub
FROM ruby

RUN which ruby

```

On mac, you may need to start `docker Quickstart Terminal` before trying out docker commands.

Now, run `docker build -t myruby .` within the same directory. Your output will look like the below.
Use `-t` option to name your image.

```
Sending build context to Docker daemon 197.7 MB
Step 1 : FROM ruby
 ---> c6ebc3270d1c
Step 2 : RUN which ruby
 ---> Running in 425293ec8f98
/usr/local/bin/ruby
 ---> bc3c6f463349
Removing intermediate container 425293ec8f98
Successfully built bc3c6f463349
```

If you try to rerun the same command, you will find that docker says that it is `Using cache`.

```
Sending build context to Docker daemon 197.7 MB
Step 1 : FROM ruby
 ---> c6ebc3270d1c
Step 2 : RUN which ruby
 ---> Using cache
 ---> bc3c6f463349
Successfully built bc3c6f463349
```

Run `docker images` to find your images.

To start a new container, run `docker run -t myruby` which will start the irb console.
To enter into the container with new TTY, run `docker run -it myruby /bin/bash`.

### Docker hub

It is a public repository which hosts all the docker images.

## Dockerize Applications

Step by step guide to move things to docker.

### Rails on docker

**Requirements**:

- ruby
- build-essential
- nokogiri dependencies
- nodejs

**Dockerfile**:

```
# Pull image from the docker hub
FROM ruby

RUN apt-get update -qq && apt-get install -y build-essential

# Nokogiri dependencies
RUN apt-get install -y libxml2-dev libxslt1-dev

# JS runtime dependencies
RUN apt-get install -y nodejs

RUN gem install rails
```

Run `docker build -t rails .` to create a new rails image.

### Rails application

Lets take a sample rails application which uses MySQL database with redis and a sidekiq for running background jobs.

#### Docker compose

  It's a tool provided by docker(_You may need to install this separately from [here](https://docs.docker.com/compose/install/)_) which takes a YAML file with all the components, links and the ports to be exposed. And it will take care of linking, exposing ports, mounting volumes etc. This command will look for docker-compose.yml file or you can also pass the custom YAML file to it.

  Sample docker-compose.yml may look like this:

  ```
  redis:
    image: redis
    ports:
      - '6379'
  ```

  Run `docker-compose build` inside the directory which contains docker-compose file to build all the images and then do `docker-compose up` to start the services. When you run build, it will fetch latest redis image from the docker hub since we are using `image` option. When do `docker-compose up`, redis container will be started in its default port which we are exposing it to our host machine by setting it in the `ports` options.

### Creating Dockerfile

Lets create a Dockerfile with ruby and install bundler.

```
# Pulls latest ruby version from the docker hub
FROM ruby

RUN apt-get update -qq && apt-get install -y build-essential

# Nokogiri dependencies
RUN apt-get install -y libxml2-dev libxslt1-dev

# JS runtime dependencies
RUN apt-get install -y nodejs

RUN apt-get install -y mysql-client

RUN gem install bundler

ENV BUNDLE_PATH /bundle
```

### Creating docker-compose file

Create a docker-compose file within the same directory where you have created Dockerfile. Typically you will have these files inside the root of your rails application.

**MySQL**:

For MySQL server you would need a persistent storage meaning you wouldn't want to have data inside the same container. For these reasons, docker provides
you with data volume containers or you could use data volume. Lets create a data volume container for MySQL. Data volume container is a container which will save your data but use a light weight and existing image for running these containers.

Now, lets create a docker-compose file with mysql data volume.

Filename: docker-compose.yml
```
mysql_data_volume:
  image: redis
  volumes:
    - /var/lib/mysql
  command: echo 'Data container for mysql'
```

We have named the image as `mysql_data_volume`. Here we are using `redis` as a light weight image and `/var/lib/mysql` is a path that is created in this container. Whenever we start the services, docker runs a startup command. For data containers, you don't want your containers to be running all the time.
So, we just run a command which will create a container and exit. But, it will not be removed.

Lets add MySQL server to it.

```
mysql_data_volume:
  image: redis
  volumes:
    - /var/lib/mysql
  command: echo 'Data container for mysql'

mysql:
  image: mysql:5.5
  volumes_from:
    - mysql_data_volume
  environment:
    - MYSQL_ROOT_PASSWORD=$DOCKER_MYSQL_ROOT_PASSWORD
```

Here, we are setting mysql version as `5.5` which will be downloaded from the docker hub. `mysql` will be the image name. Set the env. variable `DOCKER_MYSQL_ROOT_PASSWORD` in your host machine. When you start the services, docker will pick up the env. variables.

**Redis**:

```
redis:
  image: redis
```

I'm not creating data container for redis. But, you can try creating a data container for redis like we have done for mysql.

**Sidekiq**:

```
sidekiq:
  build: .
  working_dir: /opt/myapp/current
  environment:
    - BUNDLE_GEMFILE=/opt/myapp/current/Gemfile
  command: bundle exec sidekiq --environment staging -C /opt/myapp/shared/config/sidekiq.yml
  volumes:
    - '$PWD:/opt/myapp/current'
  links:
    - mysql
    - redis
```

`sidekiq` will be our image name. Setting `build` to . will tell docker-compose to look for Dockerfile within this directory. You need to set working directory for running rails commands. Use `environment` to set environment variables for the sidekiq container. Running sidekiq as startup command. You can mount host volume on the docker container using `volumes` option. Syntax goes like this `[HOST_FILE_PATH]:[DOCKER_CONTAINER_FILE_PATH]`. Since, sidekiq needs mysql and redis to run, we are linking both our images to it using `links` option.

**Application**

```
rails_app:
  build: .
  working_dir: /opt/myapp/current
  environment:
    - BUNDLE_GEMFILE=/opt/myapp/current/Gemfile
  command: bundle exec thin start
  ports:
    - '3000:3000'
  volumes:
    - '$PWD:/opt/myapp/current'
  links:
    - mysql
    - redis
```

We are using same Dockerfile for both sidekiq and the application. In `ports` options, we are exposing default port of the thin server on the host machine.

### Running your application on docker

First, run `docker-compose build .` to build all the images. Once it completes, we need to run `bundle install`. You do not want to run it all the time. Lets cache the gems by creating a data volume container for bundler.

```
bundle_data:
  image: redis
  volumes:
    - /bundle
  command: echo 'Data container for bundler'
```

Now, you need to use mount these volume inside `sidekiq` and `rails_app` images.

Your final docker-compose will look the following,

```
bundle_data:
  image: redis
  volumes:
    - /bundle
  command: echo 'Data container for bundler'

mysql_data_volume:
  image: redis
  volumes:
    - /var/lib/mysql
  command: echo 'Data container for mysql'

mysql:
  image: mysql:5.5
  volumes_from:
    - mysql_data_volume
  environment:
    - MYSQL_ROOT_PASSWORD=$DOCKER_MYSQL_ROOT_PASSWORD

redis:
  image: redis

sidekiq:
  build: .
  working_dir: /opt/myapp/current
  environment:
    - BUNDLE_GEMFILE=/opt/myapp/current/Gemfile
  command: bundle exec sidekiq --environment staging -C /opt/myapp/shared/config/sidekiq.yml
  volumes:
    - '$PWD:/opt/myapp/current'
  volumes_from:
    - bundle_data
  links:
    - mysql
    - redis

rails_app:
  build: .
  working_dir: /opt/myapp/current
  environment:
    - BUNDLE_GEMFILE=/opt/myapp/current/Gemfile
  command: bundle exec thin start
  ports:
    - '3000:3000'
  volumes:
    - '$PWD:/opt/myapp/current'
  volumes_from:
    - bundle_data
  links:
    - mysql
    - redis
```

To run `bundle install` using docker-compose, do `docker-compose run rails_app bundle install`.

Finally, run `docker-compose up` to start all the services.
