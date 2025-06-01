---
date: 2025-06-01
tags:
  - course
hubs:
  - "[[docker]]"
  - "[[devops-concept]]"
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

> [!Note]Docker CLI doesn't necessarliy needs to be on the same host. We can
> have the cli remotely and manage the docker daemon on a remote machine.

i.e. we can run all the commands remotely by specifying the `-H` flag as shown
below.

```bash
docker -H=10.123.2.1:2375 run nginx
```

## How does a application gets containerized?

Docker uses namespaces to isolate workspace, PIDs, Unix Timesharing, Network,
Interprocess, Mounts. i.e. Every Linux system starts with a PID of 1 and so on
for other processes' that it runs. The container could have 1 process running
with a PID of 1 in it's own namespace, and it will be mapped to hosts unique
PID.

```bash
docker exec <container-name> ps -eaf
```

Docker uses _cgroups_ know as control groups to control how much resources each
container could use.

    - This can be done by providing `--cpus=0.5` flag to ensure that container could only use 50% of the host cpu resource.
    - For memory we could use the `memory=100m` flag to limit the memory to 100 Megabytes.

## Where and How does docker stores data?

Docker stores all it's data by default inside `/var/lib/docker`.

### Layered Architecture:

When docker builds images, it builds them in a layered way. Each line of
instructions in the dockerfile creates a new layer in the docker image with just
the changes from previous layer.  
This caching mechanism helps with saving space and time when you update your
application code or build a new container that uses the same base image/steps
which is cached.
