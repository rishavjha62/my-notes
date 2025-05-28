---
date: 2025-05-28
tags:
  - course
hubs:
  - "[[gcp]]"
urls:
  - https://www.udemy.com/course/gcp-google-kubernetes-engine-gke-with-devops/
---

# master gcp gke

## GKE Cluster modes and Types.

There are two type of cluster modes:

1. GKE Standard
2. GKE Autopilot

These clusters can be created with different GKE Cluster types:

1. GKE Zonal Cluster
2. GKE Regional Cluster
3. GKE Public Cluster
4. GKE Private Cluster
5. GKE Alpha Cluster
6. GKE Cluster using Windows Node

## GKE standard vs Autopilot cluster.

![GKE Standard vs Autopilot cluster Architecture.](../images/gke-standard-vs-autopilot-architecture.png)

In the standard architecture, only the control plane is managed by Google.
However in Autopilot mode, both data & control plane is managed by Google.

### Create a Standard GKE Cluster

- via Console:
  1. In the consoleGo to Kubernetes Engine > Clusters > Create
  2. Select GKE Standard > Configure
  3. Fill in the Cluster Basics details.
  4. Fill the node pool details. i.e. no. of nodes, machine type, boot size,
     spot VMs.
  5.
