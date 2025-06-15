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

`--dry-run`: By default as soon as the command is run, the resource will be
created.  
If you simply want to test your command , use the `--dry-run=client` option.
This will not create the resource, instead, tell you whether the resource can be
created and if your command is right.

-o yaml: This will output the resource definition in YAML format on screen.

## Declarative and Imperative commands

There are 2 ways to deploy to kubernetes objects: imperatively with many kubectl
commands, or declaratively, by writing manifests and using `kubectl apply`.

In the imperative approach you give step by step instructions to reach your
goal, however in the declarative approach you define the final state.

- Pods creation & management

```bash
# get the pod details
kubectl get pods
# more details
kubectl get pods -o wide
# get more detailed description
kubectl describe pods <pod-name>

# Declarative approach for pods

# creating pods with yml file
kubectl apply -f <pod-definition.yml>

#Imperative approach for pod

# to create a run a pod
kubectl run --image=<image-name> <pod-name>
# create a deployment
kubectl create deployment --image=<iamge-name> <deployment-name>
# to expose a port on deployment
kubectl expose deployment <deployment-name> --port 80
# edit deployment
kubectl edit deployment <deployment-name>
# scale deployment
kubectl scale deployment <deployment-name> --replicas=5
# updating the image
kubectl set image deployment <deployment-name> <current-image>=<new-image:1.18>
# using object configuration file
kubectl create -f <filename.yml>
kubectl replace -f <filename.yml>
kubectl delete -f <filename.yml
```
