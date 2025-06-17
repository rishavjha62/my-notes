---
date: 2025-06-15
tags:
  - course
hubs:
  - "[[kubernetes]]"
urls:
  - udemy.com/course/certified-kubernetes-administrator-with-practice-tests
---

# kubernetes administrator

## Scheduling

- [Manual Scheduling](#manual-scheduling-how-scheduling-works)
- [Labels & Selectors](#labels--selectors)
- [Taints & Tolerations](#taints--tolerations)
- [Node selector](#node-selector)
- [Node Affinity](#node-affinity)
- [Resource Limits](#resource-requirements--limitations)
- [Daemon Sets](#daemon-sets)
- [Static Pods](#static-pods)
- [Priority Classes](#priority-classes)
- [Multiple Schedulers](#multiple-schedulers)
- [Scheduler Events](#scheduler-events)
- [Configure Kubernetes Scheduler](#configure-kubernetes-scheduler)

### Manual Scheduling: How scheduling works?

Every pod has a field called `nodeName` that is not set by default. The
scheduler goes through all the pods that don't have this property set. The
scheudler then identifies the right node for the pod by running the scheduling
algorithm & sets the `nodeName` property on those pods by creating a binding
object.  
In case there is no scheduler to monitor or schedule nodes, the pods continue to
be in pending state. You can manually schedule the pods by setting the
`nodeName` manually on these pods.

> The `nodeName` can only be assigned at creation time.

> [!NOTE]  
> The way to assign a existing pod to another node is by creating a binding
> object & send a POST request to the pod's binding API thus mimicing what the
> actual scheduler does.  
> You must convert the below yml file to json format as shown below.
>
> ```yml
> apiVersion: v1
> kind: Binding
> metadata:
>   name: nginx
> target:
>   apiVersion: v1
>   kind: Node
>   name: node02
> ```
>
> ```bash
> curl --header "Content-Type: application/json" --request POST --data '{"apiVersion":"v1", "kind": "Binding"...} \
> http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/
> ```

### Labels & Selectors

Labels : They are properties attached to items. They are defined under metadata
under the labels section and can be added as key value format.

Selectors: They help with filtering those items based on the labels.

Services, Deployments, ReplicaSet use labels to create pods.

Annotations: They're used to record other details like buildVersion, contact
details, phone number, etc.

### Taints & Tolerations:

Taints and Toleration are used to set restrictions on what pods can be scheudled
on a node.

For example: If we've 3 nodes (node 1, 2, 3 & 4) and 4 pods (A, B, C, D), the
scheduler tries to place these pods on available worker nodes. As of now there
are no restrictions. Let's assume we've dedicated resources on node 1 for a
particular use case so we would like only those pods to be put on node 1 that
belong to the applications.  
First, we prevent all nodes from being placed on node 1 by placing a blue taint
on the node. None of the pods have any tolerations, therefore none of the pods
will be placed on this node. Now, if the pod D belongs to our applications, we
add a toleration to our D pod.

Taints are applied on pods. There are 3 taint effect:

- NoSchedule -> The pods will not be scheduled on the node.
- PreferNoSchedule -> The system will try to avoid placing a node on the pod but
  it is not guaranteed.
- NoExecute -> New pods will not be scheudled on the node and existing pods will
  be evicted if they don't tolerate the taint.

```bash
kubectl taint nodes <node-name> key=value:taint-effect

```

Tolerations are added to pods and can be added in the pod definition file under
the spec section.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod

spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

> [!NOTE]  
> Taints and Tolerations are only meant to restrict nodes from accepting certain
> pods. It doesn't guarantee that a pod will always be placed on a certain node.

### Node Selector:

To set limitations on the pods so that they can only run on certain nodes. We
can do this by specifying the `NodeSelector` field in the pod definition file
under the spec section & labeling the node as done in the pod definition file.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx
      image: nginx
  NodeSelector:
    size: Large
```

```bash
kubectl label ndoes node01 size=Large
```

### Node Affinity:

The primary feature of Node Affinity is to ensure that pods are hosted on
particular nodes. It provides advanced expressions to limit pod placement on
specific nodes. It can be done by updating the pod definition file under spec
section by adding `affinity` as shown below.

```yml
apiVersion: v1
kind: pod
metadata:
  name: myapp-pod
spec:
  contianers:
    - name: nginx
      image: nginx
  affinity:
    nodeAAffinity:
      requiredDuringSchedulingIgnoreDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: size
              operator: In
              value:
                - Large
                - Medium
```

```yml
affinity:
  nodeAAffinity:
    requiredDuringSchedulingIgnoreDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: size
            operator: NotIn
            value:
              - Small
```

The Node Affinity types defines the behaivour of the scheduler with respect to
node affinity and the stages in the lifecycle of the pod. There are 2 types of
node affinity types:

- requiredDuringSchedulingIgnoreDuringExecution
- preferredDuringSchedulingIgnoreDuringExecution
  > In Planning: requiredDuringSchedulingRequiredDuringExecution

|       | DuringScheduling | DuringExecution |
| ----- | ---------------- | --------------- |
| Type1 | Required         | Ignored         |
| Type2 | Preferred        | Ignored         |
| Type3 | Required         | Required        |

### Resource Requirements & Limitations

The kube scheduler decides which node the pod goes on to. The scheduler
considers the amount of resource required by the pod and those available on the
nodes and identies the best nodes to place the pod on.  
If sufficient resources are not avialable on any of the nodes, the scheduler
then holds back scheduling the pod and the pod will be in pending state.  
To define the resources used by the pod, we create a section named `resources`
under the `spec>containers`.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - contianerPort: 8080
      resources:
        requests:
          memory: "4Gi"
          cpu: 2
        limits:
          memory: "2Gi"
          cpu: 2
```

We can also set a limit on the resouces the pod can access under the `resources`
section in `spec>containers` as shown above.

> [!NOTE]  
> A pod can only use the cpu defined in the limits however it can use more
> Memory than what is specified. The system thorttles the cpu so that it can go
> beyond the defined limit.  
> The pod will be terminated in case it constantly tries to use more memory than
> what is defined and will give OOM(Out of Memory) error in the logs.

> [!NOTE]  
> By default kubernetes doesn't have a cpu or memory request limit set.

Limit Ranges:  
We can ensure all pods have some default resource set by using **limit ranges**.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-resource-constraint
spec:
  limits:
    - default:
        cpu: 500m
      defaultRequest:
        cpu: 500m
      max:
        cpu: "1"
      min:
        cpu: 100m
      type: Container
```

Resource Quotas:  
To restrict the total amount of resources that can be consumed by applications
deployed in the cluster, we use quotas at a namespace level using the definition
file.

```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limit.cpu: 10
    limit.memory: 10Gi
```

### Daemon Sets:

Daemon Sets are like ReplicaSet as it helps to deploy multiple instances of
pods. It runs one copy of pod on each node in the cluster. Whenever a new node
is added, a replica of the pod is deployed on the node.  
Use cases of daemon sets:

- Monitoring agent
- logs viewer
- kube-proxy can be deployed as a daemon set
- networking solutions like weavenet reuquires an agent on each node.

It can deployed with a yaml definition file and looks similar to the ReplicaSet.
The kind is of the type DaemonSet `kind: DaemonSet`.

> It uses default scheduler & node affinity to deploy a pod on each node.

### Static Pods:

The kubelet can manage the node independently in absence of kubeapi server,
scheduler, controllers, & etcd. We've kubelet & Docker installed on host. We can
configure the kubelet to read the pod definition files from a directory
`/etc/kubernetes/manifests` on the host designated to store information about
the pods. It can also ensure that pod stays alive. The pods created this way are
called **Static pods**.

> [!NOTE]

> This directory could be any directory on the host. The location for this
> directory could be set by passing the directory to the kubelet as on option
> while running the service.
>
> ```kubelet.service
> ExecStart=/usr/local/bin/kubelet \\
>   --pod-manifest-path=/etc/Kubernetes/manifests \\
> ```
>
> The other way to do this is by defining `--config=kubeconfig.yaml` in the
> `kubelet.serive`. Clusters setup by kubeadmin tool use this method.  
> kubeconfig.yaml  
> staticPodPath: /etc/kubernetes/manifests

The kubelet can create both kind of pods, the static pods and the ones from the
API server at the same time. The API server is still aware of the static pods
created by the kubelet. When the kubelet creates a static pod that is part of a
cluster, it also creates a mirror object in the kubeapi server as a read only
mirror of the pod.

Use Case:  
These static pods can be used to deploy the control plane comomponents as pods
on the nodes.

1. Start by installing kubelet on all the master nodes.
2. Create pod definition file that uses docker images of the various control
   comomponents.
3. Place the pod definition files in the designated manifests folder.

### Priority Classes

They help us define priorities for different workload. They are non namespace
objects. They are available for any pod on any namespace. They can be defined
with a range of numbers (2,000,000,000 - 1,000,000,000 - 2,147,483,648).

The kubernetes control plane objects gets the highest priority up to
2,000,000,000.

```bash
# To list the priority classes run
kubectl get priorityclass
```

Use pod defintion files to create a new priority class. And once a priority
class is created it can be used within a pod defintion file creating a pod as
shown below:

```yml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
```

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  contianers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
  priorityClassName: high-priority
```

If no priority class is defined in the definition file, it is assumed to have a
priority of 0. The default value can however be changed to a different default
value by adding `globalDefault: True` in the definition file created for
priority class.

The `preemptionPolicy` decides the behaivour wether to evict a lower priority
pod. By default it is set to `preemptionPolicy: PreemptLowerPriority` which
kills the lower priority pods. If it is set to `preemptionPolicy: never` it will
wait for the reources to be available before a new pod with lower priority is
placed on nodes.

### Multiple Scheduers

We can create our custom scheduler and multiple of those can be used by certain
application. These schedulers can be created using pod defintion file.

We use these custom scheduler in case we want to perform additional checks.

When creating a pod or deployment we can instruct kubernetes to have the pod
scheduled by a specific scheduler.

```yml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

We can point the new scheduler service to this new scheduler:

```my-scheduler.service
ExecStart=/usr/local/bin/kube-scheduler \\
    --config=/etc/kubernetes/config/my-scheduler-config.yml
```
