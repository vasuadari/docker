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
        - Create Dockerfile for rails and its dependencies (TODO)
        - Create docker-compose file to setup the application (TODO)
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