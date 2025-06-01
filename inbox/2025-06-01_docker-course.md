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
Interprocess, Mounts, are created in their own namespaces thereby providing
isolation between containers.  
i.e. Every Linux system starts with a PID of 1 and so on for other processes'
that it runs, however a container inside the system could also have a process
with PID of 1 under it's unique namespace. The container's PID of 1 mapped to
host PID which could be same or different.

Docker uses _cgroups_ know as control groups to control how much resources each
container could use.

    - This can be done by providing `--cpus=0.5` flag to ensure that container could

only use 50% of the host cpu resource.

- For memory we could use the `memory=100m` flag to limit the memory to 100
  Megabytes.
