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

- Manual Scheduling
- Labels & Selectors
- Taints & Tolerations
- Node selector
- Node Affinity
- Resource Limits
- Daemon Sets
- Multiple Schedulers
- Scheduler Events
- Configure Kubernetes Scheduler

1. Manual Scheduling: How scheduling works?

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
>   apiVerison: v1
>   kind: Node
>   name: node02
> ```
>
> ```bash
> curl --header "Content-Type: application/json" --request POST --data '{"apiVersion":"v1", "kind": "Binding"...} \
> http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/
> ```

2. Labels & Selectors

Labels : They are properties attached to items. They are defined under metadata
under the labels section and can be added as key value format.

Selectors: They help with filtering those items based on the labels.

Services, Deployments, ReplicaSet use labels to create pods.

Annotations: They're used to record other details like buildVersion, contact
details, phone number, etc.

3. Taints & Tolerations:

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

4. Node Selector:

To set limitations on the pods so that they can only run on certain nodes. We
can do this by specifying the `NodeSelector` field in the pod definition file
under the spec section.

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
