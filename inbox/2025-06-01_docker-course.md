---
date: 2025-06-01
tags:
  - course
hubs:
  - "[[docker]]"
  - "[[devops-concept]]"
urls:
  - udemy.com/course/learn-docker
  - youtube.com/watch?v=RqTEHSBrYFw
  - udemy.com/course/containers-under-the-hood
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

Linux building blocks that make up the core of a container:

1. **Namespaces**:

Docker uses _namespaces_ to isolate workspace PIDs, Unix Timesharing, Network,
Interprocess, Mounts, User, UTS, IPC, Cgroup. i.e. Every Linux system starts
with a PID of 1 and so on for other processes' that it runs. The container could
have 1 process running with a PID of 1 in it's own namespace, and it will be
mapped to hosts unique PID.

A namespace wraps a global system resource in an abstraction that makes it
appear to the processes within the namespace that they have their own isolated
instance of the global resource.  
Changes to the global resource are visible to other processes that are members
of the namespace, but are invisible to other processes.

- PID namespace :  
  To create a new PID namespace.

```bash
# to create a new namespce
sudo unshare -p --fork --mount-proc

# to check all namespaces
lsns  # -t pid -> to view PIDs

# to run commands from parent namespace
sudo nsenter -t <pid> -pr <command>
```

- Mount namespaces :  
  A directory of drive could be mounted(superimposed) on another directory.

```bash
# to mount a directory on another
sudo mount --bind <dir_to_mount> <mount_dir>

# to show all mounted directories
df -a

# to unmount
sudo umount -l <mount_dir>

#to create a new namespace for mount & PID
sudo unshare -pfm --mount-proc

#to make a certain directory as the root directory but ensure it's mounted
pivot_root <new_location> <backup_for_old_root>
```

```bash
# shows all the processes running inside the container
docker exec <container-name> ps -eaf
```

2. **Control Groups (cgroups)**

Docker uses _cgroups_ know as control groups to control how much resources each
container could use.

It's a Linux kernel feature which allow processes to be organized into
hierarchical group whose usage of various types of resource can then be limited
and monitored.

```bash
# to see all available cgroups
cat /proc/cgrous
```

    - This can be done by providing `--cpus=0.5` flag to ensure that container could only use 50% of the host cpu resource.
    - For memory we could use the `memory=100m` flag to limit the memory to 100 Megabytes.

3. **Union Mount Filesystems (overlayfs)**

Allows files and directories of separate file systems, knows as branches, to be
transparently overlaid, forming a single coherent file system.  
Contents of directories which have the same path within the merged braches will
be seen together in a single merged directory, within the new, virtual
filesystem.

Docker stores all it's data by default inside `/var/lib/docker`.

## Layered Architecture:

When docker builds images, it builds them in a layered way. Each line of
instructions in the dockerfile creates a new layer in the docker image with just
the changes from previous layer.  
This caching mechanism helps with saving space and time when you update your
application code or build a new container that uses the same base image/steps
which is cached.

Once you build the image from the dockerfile, they are read only and can't be
changed unless you create a new build. when a container is created using the
image, docker creates a new writable layer on top of the image layers. This
writable layer is used to store data created by the container such as log files,
temp files or files modified by the user. The life of this layer ends whenever
the contianer is destroyed.

![Layers-representation](../Excalidraw/docker-layers.excalidraw.md)

When you modify your app, the changes take place in a copy present in the read-
write layer. This is called _Copy on Write_ mechanism.  
To persist this data, we need to mount a persistent volume on the container. We
can use the `docker volume create <volume-name>` command to create a volume and
then mount it when running the docker container using the flag
`-v /path/on/host:/path/on/container` command. This is called _Bind Mounting_.  
Similary we can use the _Volume mount_ to mount the volume we created above.

> [!NOTE] Using the -v flag is the old way and is replaced with --mount.

> [!NOTE] Docker suggests to use _Volume mount_ as it is efficient due to read
> write operations.

example:

```bash
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

### Who is responsible for doing all the storage operations?

The _Storage Drivers_ are responsible for tasks such as maintaing the layered
Architecture, creating the writable layer, moving files across layers to enable
copy on write, etc.  
Some of the common storage drivers are:

1. AUFS
2. ZFS
3. BTRFS
4. Device Mapper
5. Overlay
6. Overlay2

> [!NOTE] The storage drivers are choosen based on the underlying OS. i.e.  
> AUFS is used for Ubuntu.
