---
date: 2025-05-26
tags:
  - notes
hubs:
  - "[[istio]]"
  - "[[devops-notes]]"
urls:
  - https://www.udemy.com/course/istio-hands-on-for-kubernetes
---

# istio notes

## What is Istio?

Istio is a _service mesh_

A Service mesh is a extra layer of software alongside your cluster. It is used
to gather telemtrics for individual network requests, traffic management,
security.

## How does Istio Implements a Mesh?

It cretaes a proxy container (side car, Envoy) which is used to route network
requests from our main container. A collection of proxies is called the _Data
Plane_.

These proxies are controlled by _istiod_ known as _istio daemon_ which lives in
the istio namespace also known as the _Control Plane_. There are other pods in
the control plane for providing UI such as _kiali_ and _jaeger_.
