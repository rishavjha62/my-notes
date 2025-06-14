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

- **Pods**: A container is encapsulated in to a kubernetes object known as pods.
  A pod is single instance of an application. It is smallest object you can
  create in kubernetes.

- **Cluster**: A cluster is a set of nodes grouped together.

- **Master Node**: A master node is another node in the cluster with kubernetes
  installed. It watches over the other nodes in the cluster and is responsible
  for orchestration of the contianers on worker nodes.

## Kubernetes Architecture

When you install kubernetes on a system, the following components are installed:

- API Server
- etcd keystore
- scheduler
- controller
- container runtime
- kubelet
