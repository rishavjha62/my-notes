---
date: 2025-06-14
tags:
  - cheat-sheet
hubs:
  - "[[kubernetes]]"
urls:
  - udemy.com/course/learn-kubernetes/
  - udemy.com/course/certified-kubernetes-administrator-with-practice-tests
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
# using object configuration file
kubectl create -f <filename.yml>
kubectl replace -f <filename.yml>
kubectl delete -f <filename.yml

# Pod
# Create an NGINX Pod
kubectl run nginx --image=nginx

# Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

- Deployment creation & management

```bash

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

# Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

# Generate Deployment with 4 Replicas
kubectl create deployment nginx --image=nginx --replicas=4

# You can also scale a deployment using the kubectl scale command.
kubectl scale deployment nginx --replicas=4

# Another way to do this is to save the YAML definition to a file and modify
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

- Service creation & management

```bash
# Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
# or
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml

# Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
# or
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

- Updates and Roll back of deployments

```bash
# check status of your rollouts
kubectl rollout status deployment/myapp

# check history of your rollouts
kubectl rollout history deployment/myapp-dp

# undo a change
kubectl rollout undo deployment/myapp-dp
```

## kubernetes administrator

### Labels & Selectors

```bash
kubectl get pods --selector app=App1
```

### Taints and Tolerations

```bash
kubectl taint nodes <node-name> key=value:NoSchedule
kubectl taint nodes <node-name> key=vlaue:PreferNoScheudle
kuectl taint nodes <node-name> key=vlaue:NoExecute

# to check the taint on the nodes
kubectl describe nodes <node-name> | grep Taint

# to remove taint from controlplane first run the above command and use the output
kubectl taint nodes <ndoe-name> <output-from-previous-command>-
```

### Node Selector

```bash
#to create a label on the node
kubectl label ndoes <node-name> <label-key>=<label_value>
```

### Priority Class

```bash
# To list the priority classes run
kubectl get priorityclass
```

### To look for scheduled events

```bash
# to know which scheduler was picked by a pod
kubectl get events -o wid

# view logs of the scheduler
kubectl logs my-custom-scheduler --name-space=kube-system
```

## Admission Controller

````bash
kube-apiserver -h | grep enable-admission-plugins

# if running via kubeadm setup
kubectl exec kube-apiserver-controlplane -n kube-system -h | grep enable-admission-plugins
> ```
````

### Monitoring Cluster Components

```bash
# for minikube
minikube addons enable metrics-server

# for others
git clone https://github.com/kubernetes-incubator/metrics-server
kubectl create -f deploy/1.8+/

# to view metrics
kubectl top node
```

Application log

```bash
# To view the application log
kubeclt logs -f <pod-name>
```
