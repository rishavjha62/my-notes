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

> [!NOTE] The way to assign a existing pod to another node.  
> Create a binding object & send a POST request to the pod's binding API thus
> mimicing what the actual scheduler does.  
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
