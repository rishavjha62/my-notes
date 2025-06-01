---
date: 2025-06-01
tags:
  - course
hubs:
  - "[[docker]]"
  - "[[devops-concept]]"
  - "[[2025-05-29_docker-cheat-sheet]]"
urls:
  -
---

# docker course

## Docker Architecture

When you install docker on a host, it consists of Docker CLI, REST API, & Docker
Daemon.

- Docker Daemon : It the background process that manages the docker objects such
  as the docker images, containers, volumes & network.

- REST API: It is the interface that programs can use to communicate to the
  docker daemon and provide instruction.

- Docker CLI: It is the command line interface that that we can use to perform
  actions such as running a new container, destroying it, etc.
