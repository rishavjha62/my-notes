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

To list all the images available locally:  
`docker images` or `docker image ls`
