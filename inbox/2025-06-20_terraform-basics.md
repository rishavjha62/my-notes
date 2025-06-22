---
date: 2025-06-20
tags:
  -  #course
hubs:
  - "[[terraform]]"
urls:
  - udemy.com/course/terraform-beginner-to-advanced
  - udemy.com/course/terraform-for-the-absolute-beginners
  - udemy.com/course/terraform-certified
---

# terraform basics

HCL basics:

```tf
resource "<block>" "<parameters>" {
    key1 = "value1"
    key2 = "value2"
}
```

```tf
resource "local_file" "pet" {
    filename = "~/pets.txt"
    content = "we love pets"
}
```

- **Block Name**: In above example the `resource` is the first element which is
  of the type `block name`.

- **Resource Type**: Following resource we've the declration of the Resource
  Type that we want to create. This is a fixed value and depends on the provider
  where we want to create the resource. In this example we've the local resouce
  with the type being file `[local = provider, file = resource]`.

- **Resource Name**: This is logical name that can be used to identify the
  resouce. It can be named anything. Here we've called it `pet`.

- **Arguments**: Within the curly braces, we can provide the arguments in a key
  value formats. These arguments are specific to the type of resources we are
  creating.
