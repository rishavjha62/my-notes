---
date: 2025-05-29
tags:
  - cheat-sheet
hubs:
  - "[[docker]]"
  - "[[devops-concept]]"
urls:
  -
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

To pull an image:  
`docker pull <image-name>`

`docker pull ubuntu`

To list all the images available locally:  
`docker images` or `docker image ls`

To run a container:

> [!NOTE] Docker will run the container if the image already exists lcoally,  
> otherwiser it will first download it from dockerhub

`docker run <image-name>` i.e.

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

To gracefully stop a contaner:  
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
