---
date: 2025-06-14
tags:
  - cheat-sheet
hubs:
  - "[[kubernetes]]"
urls:
  - udemy.com/course/learn-kubernetes/
---

# kubernetes cheat sheet

## Declarative and Imperative commands

- Declarative approach for pods

```bash
# creating pods with yml file
kubectl create -f <pod-definition.yml>
# or
kubectl apply -f <pod-definition.yml>

# get the pod details
kubectl get pods
# more details
kubectl get pods -o wide

# get more detailed description
kubectl describe pods <pod-name>

```
