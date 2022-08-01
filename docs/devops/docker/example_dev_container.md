# Example Container

## Clone a repo todo_app in your local filesystem

`git clone github:some_repo ./todo_app`

## Create a container

We are creating a new container based on ruby:2.7.2 image with volume mounted to our filesystem's `todo_app` folder in current working directory.

```sh
docker run -i -t --volume="$pwd":/todo_app -w /todo_app ruby:2.7.2 bin/bash
```

This will setup todo_app as working directory which will be prompted to us during start of the container.

If you exit this container and use the `RUN` command again, the changes will be lost, as it will create a completely new container.

## Continue using the same container with exec

Use `docker exec` command to use the previous container you have built.

```sh
docker start $container_name
docker exec -i -t $container_name -w /todo_app $container_name bin/bash
```

## Docker Compose

Typing all the commands and their options every time we work with container is cumbersome. Thus, Docker Compose comes into play where we create a configuration files that will ease things up when working with one or several containers.

### Basic Docker Compose YML

```yml
# Version of the file
version: "1.0"
services:
  app: 
    image: "ruby:2.7.2"
    volumes:
      type: "bind"
      source: ..
      target: "/todo_app"
    working_dir: "/todo_app"
    # Let's container know to always run until interrupted.
    command: sleep infinity
```

```sh
docker compose up
```

```sh
docker compose exec app bin/bash
```

### Build your own image with Dockerfile and set that up with Compose

If you would like to define the instructions and build your own custom image, you can create a Dockerfile and set that up in Docker Compose config.

```docker
<!-- Dockerfile -->
FROM ruby:2.7.2
RUN apt-get update
RUN apt-get install -y yarn
```

> Prefer less RUN statements and try to join multiple commands with `\` to reduce the [Docker Layers](https://vsupalov.com/docker-image-layers/), which will lead to more slimmer images.

```yml
version: "1.0"
services:
  app: 
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      type: "bind"
      source: ..
      target: "/todo_app"
    working_dir: "/todo_app"
    # Let's container know to always run until interrupted.
    command: sleep infinity
```

### Port Mapping

```YML
version: "3.2"
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
  volumes:
    - type: bind
    source: ..
    target: /workspace
  working_dir: /workspace
  command: sleep infinity
  ports:
    # Don't forget to bind the puma server to 0.0.0.0 instead of loopback 127.0.0.1
    - "3000:3000"
  ```