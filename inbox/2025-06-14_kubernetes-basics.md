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

1. We've some client (using _kubectl_) that run `kubectl apply -f pod.yml`
2. kubectl is sending a POST request to a endpoint in the API server.
3. The _API server_ will persist the file/document in etcd.
4. The API Server will then send the pod to any client(node) that has open watch
   mechanism(via _kubelet_).
5. The node will have a _container runtime_(docker) which will check if the pod
   is already running, if not it will create a new pod on the node.
6. There is a _scheduler_ which orchestrates the pods on the nodes. This
   scheduler has a watch mechanism for both pods and the nodes. It will watch
   for pods that don't have any node name specified and look for the right node
   to put it on as efficiently as possible.
7. Kubernetes bundles together controller process together as the _controller
   manager_. i. The _replicaset controller_ will watch for replicasets and pods.
   It will query for the number of pods that corresponds to the replicasets, and
   it will issue the creation of these pods if not present already. ii. The
   _deployment controller_ will watch for deployments and replicasets. It wil
   query the replicasets and issue creation of replicasets if a new version is
   detected.
