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

> [!NOTE]  
> `--dry-run`: By default as soon as the command is run, the resource will be
> created.  
> If you simply want to test your command , use the `--dry-run=client` option.
> This will not create the resource, instead, tell you whether the resource can
> be created and if your command is right.  
> `-o yaml`: This will output the resource definition in YAML format on screen.

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

# Pod
# Create an NGINX Pod
kubectl run nginx --image=nginx

# Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Deployment

#Create a deployment
kubectl create deployment --image=nginx nginx

# Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

# Generate Deployment with 4 Replicas
kubectl create deployment nginx --image=nginx --replicas=4

# You can also scale a deployment using the kubectl scale command.
kubectl scale deployment nginx --replicas=4

# Another way to do this is to save the YAML definition to a file and modify
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml

# service

# Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
# or
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml

# Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
# or
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```
