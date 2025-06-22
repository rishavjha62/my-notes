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

## HCL basics:

### Basics of creating a resource

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

Example resource :
https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance

```tf
resource "google_compute_instance" "my-first-gce" {
    name = "my-first-gce-with-terraform"
    zone = "us-central1-a"
    tags = ["foo", "bar"]
    mahcine_type = "n1-standard-2"
    network_interface {
        network = "default"
    }
    boot_disk {
        intitialize_params {
            image = "debian-10-buster-v20210196"
            size = 20
        }
    }
    labels = {
        "env" = "test"
    }
}
```

### Terraform Workflow:

A simple terraform workflow consists of 4 steps:

1. Write the config file.

2. Run the `terraform init` command. This will check the config file and
   initialize the working directory containing the .tf files. It will look for
   all the providers used based on the resource type declared and will download
   the plugin to be able to work on these resources.

3. Review the execution plan with `terraform plan` command. This will show all
   the actions carried out by terraform to create the resouces.

4. Apply the changes with `terraform apply` command.

> `terraform show` can be used to check all the resouces that has been created.
> This information is inspected from the state file.

### Update & Destroy Infrastructure

In case we've made some changes to our .tf file. We can run the `terraform plan`
command before publishing the resource. This will show all the changes in a
`diff` format.

Terraform will first delete the resouce and create a new resouce. This can be
seen in the terminal with `-+` symbol. This type of Infrastructure is called
**Immutable Infrastructure**.
