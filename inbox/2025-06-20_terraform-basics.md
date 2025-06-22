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
  of the type `block name`. following it is Resource Type
