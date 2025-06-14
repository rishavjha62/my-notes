---
date: 2025-06-14
tags:
  - course
hubs:
  - "[[kubernetes]]"
urls:
  - udemy.com/course/learn-kubernetes/
  - youtube.com/live/YXMC9eymqC8?si=8qSPG6Q5NrEFv-J7
---

# kubernetes basics

- **Pods**: A pod is single instance of an application. It is smallest object
  you can create in kubernetes. A container is encapsulated in to a kubernetes
  object known as pods.

> [!NOTE] A pod can have multiple containers.  
> These containers are usually not of the same kind.

- **Cluster**: A cluster is a set of nodes grouped together.

- **Master Node**: A master node is another node in the cluster with kubernetes
  installed. It watches over the other nodes in the cluster and is responsible
  for orchestration of the contianers on worker nodes.

## Kubernetes Architecture

When you install kubernetes on a system, the following components are installed:

- API Server -> REST API
- etcd keystore -> database
- scheduler
- controller
- container runtime
- kubelet

How does a container runs pod?

1. We've some client (using kubectl) that run `kubectl apply -f pod.yml`
2. kubectl is sending a POST request to a endpoint in the API server.
3. The API server will persist the file/document in etcd.
4. The API Server will then send the pod to any client(node) that has open watch
   mechanism(via kubelet).
5. The node will have a container runtime(docker) which will check if the pod is
   already running, if not it will create a new pod on the node.
6. There is a scheduler as which will schedule orchestrate the pods on the
   nodes.
