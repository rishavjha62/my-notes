---
date: 2025-05-29
tags:
  - cheat-sheet
hubs:
  - "[[docker]]"
  - "[[devops-concept]]"
urls:
  - https://docs.docker.com/reference/cli/docker/
---

# docker cheat sheet

## Collection of Docker commands

To install Docker:

- Requirements:

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

- Installation:

```bash
# get the docker file
curl -fsSL https://get.docker.com -o get-docker.sh
chmod +x get-docker
sudo usermod -aG docker $USER  # to add current user to docker group
```

To restart or enable docker service:

```bash
sudo systemctl restart docker.service
sudo systemctl enable docker.service
```

To list all running containers:  
`docker ps`  
or  
`docker container ls`

To list all containers running and stopped:  
`docker ps -a`  
or  
`docker container ls -a`

To list all the images available locally:  
`docker images`  
or  
`docker image ls`

To remove image:  
`docker rmi <image-name>`

To run a container:

> [!NOTE] Docker will run the container if the image already exists locally,
> otherwise it will first download it from dockerhub

`docker run <image-name>`  
i.e.

```bash
docker run ubuntu
docker run -d ubuntu # runs it in detached mode
docker run -it ubuntu # runs it in interactive mode with terminal access hence - i & t

# some practical examples

# runs postgres container named postgres-test, -e for env variables, -p for port mapping
# here 5431:5432 -> 5431 is the host machine port & 5432 is container port
docker run --name fastapi-postgres -e POSTGRES_PASSWORD=password -d -p 5431:5432 postgres:alpine

# runs jenkins container -v for mapping volume
docker run -p 5000:8080 -v jenkins:/var/jenkins_home jenkins/jenkins

# combined all above
docker run --name fastapi-postgres -e POSTGRES_PASSWORD=password -d -p 5432:5432 -v fastapi-app-data:/var/lib/postgresql/data  postgres:alpine
```

To gracefully stop a container:  
`docker stop <container-name>`

To kill a container  
`docker kill <container-name>`

To restart a container  
`docker restart <container-name>`

To suspend & resume the container:

```bash
docker pause <container-name>
#or
docker unpause <container-name>
```

To destroy the container:

```bash
docker rm <image-name>
or
docker rm <image-hash> # don't need to enter the complete hash just enough to seperate it from others
```

To inspect the container:

```bash
docker logs -f <container-name or container-hash>
docker top <container-name or container hash>
docker inspect <container-name or container hash>

# to check image history
docker history <image-name or hash>
```

To interact with the container:

```bash
docker attach <container-name or container hash>
docker exec <container-name or container hash> args ...
```

Image management :

```bash
# To pull an image:
docker pull repo[:tag]

# To push an image:
docker push repo[:tag]

# Search for an image on dockerhub
docker search <text>

# login to a registry
docker login

# logout from a regsitry
docker logout


```

| command            | description                                           |
| ------------------ | ----------------------------------------------------- |
| FROM image/scratch | base image for the build                              |
| MAINTAINER email   | name of the maintainer (metadata)                     |
| COPY path dst      | copy from host to location in container               |
| ADD src dst        | same as COPY but untar archives and accepts http urls |
| RUN args...        | run an arbitrary command inside the container         |
| USER name          | set the default username                              |
| WORKDIR path       | set the default working directory                     |
| CMD args...        | set the default command                               |
| ENV name value     | set the environment variable                          |

i.e. an example from practice that creates a fastapi app container.

```Dockerfile
FROM ubuntu:latest

RUN apt-get update -y && \
    apt-get install -y python3 python3-pip python3-venv

# Create and activate a virtual environment
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY ./assessment/requirements.txt /tmp/


# Upgrade pip inside venv and install packages

RUN /opt/venv/bin/pip install --upgrade pip && \
    /opt/venv/bin/pip install -r /tmp/requirements.txt


COPY ./assessment /opt/assessment
WORKDIR /opt/assessment

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```

To build a docker image locally:

```bash
docker build [options] -t <container-name> # -t to name the container
docker build . -t <container-name> # from the Dockerfile in current path
docker build -f Dockerfile -t test/my-app:latest --no-cache #build with no cache
```

Storage:

To view storage used by docker: `docker system df`

To remove all build cache: `docker builder prune`

To remove all unused storage volume: `docker volume prune`
