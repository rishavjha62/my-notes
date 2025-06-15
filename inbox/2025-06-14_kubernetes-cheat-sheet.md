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

There are 2 ways to deploy to kubernetes objects: imperatively with many kubectl
commands, or declaratively, by writing manifests and using `kubectl apply`.

In the imperative approach you give step by step instructions to reach your
goal, however in the declarative approach you define the final state.

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
