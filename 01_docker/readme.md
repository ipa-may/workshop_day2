# 01_docker

The goal of workshop `01_docker` is to understand the basics of Docker and containers.
Hands-on: 

## What is Docker?

Docker is a tool that helps us build, package, and run software in a consistent way.
It puts an application together with everything it needs, such as libraries, system tools,
and configuration, so it behaves the same on different computers.

## What is a container?

A container is a lightweight, isolated runtime environment for an application.
You can think of it as a small box that contains the application and its dependencies,
while still sharing the host computer's operating system kernel.

## Docker vs containers

Docker is the platform and tooling.
Containers are the running units created with that tooling.
- Docker helps you create and manage containers.
- A container runs your application in an isolated environment.
- This makes setup easier, improves reproducibility, and reduces "it works on my machine" problems.

## What is an image?

An image is a packaged template used to create containers.
It contains the application code, dependencies, and instructions needed to start the software.
You can think of an image as the blueprint, and a container as the running instance of that blueprint.

## What is docker-compose?

`docker-compose` is a tool for defining and starting multiple containers together.
Instead of typing many Docker commands manually, you describe the setup in a
`compose.yaml` or `docker-compose.yml` file and then start everything with one command.

This is useful when one application needs several services, for example:
- one container for the app
- one container for a database
- one container for another supporting service

## What is a bind mount?

A bind mount connects a folder or file from the host computer into a container.
This means the container can directly read or write files from your machine.

This is useful when:
- you want to edit code on the host and use it immediately in the container
- you want files created by the container to remain available on the host
- you want to share configuration or data between the host and the container


## Useful commands

### docker

`docker ps`
- Shows the running containers.

`docker ps -a`
- Shows all containers, including stopped ones.

`docker images`
- Lists the Docker images available on your computer.

`docker run <image>`
- Starts a new container from an image.

`docker exec -it <container> bash`
- Opens an interactive shell inside a running container.

`docker stop <container>`
- Stops a running container.

`docker rm <container>`
- Removes a stopped container.


### docker compose

`docker compose up`
- Starts the services defined in the compose file.

`docker compose up -d`
- Starts the services in the background.

`docker compose down`
- Stops and removes the containers started by Compose.

## Documentation

For more details, see the official Docker documentation:
- Docker documentation: https://docs.docker.com/
- Docker Compose guide: https://docs.docker.com/guides/docker-compose/
- `docker compose` command reference: https://docs.docker.com/compose/reference/
- Bind mounts: https://docs.docker.com/engine/storage/bind-mounts/
