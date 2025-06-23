---
date: 2025-06-20
tags:
  - course
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

```terraform
resource "<block>" "<parameters>" {
    key1 = "value1"
    key2 = "value2"
}
```

```terraform
resource "local_file" "pet" {
    filename = "~/pets.txt"
    content = "we love pets"
}
```

- **Block Name**: In above example the `resource` is the first element which is
  of the type `block name`.

- **Resource Type**: Following resource we've the declaration of the Resource
  Type that we want to create. This is a fixed value and depends on the provider
  where we want to create the resource. In this example we've the local resource
  with the type being file `[local = provider, file = resource]`.

- **Resource Name**: This is logical name that can be used to identify the
  resource. It can be named anything. Here we've called it `pet`.

- **Arguments**: Within the curly braces, we can provide the arguments in a key
  value formats. These arguments are specific to the type of resources we are
  creating.

Example resource :
https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance

```terraform
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
   initialize the working directory containing the `.tf` files. It will look for
   all the providers used based on the resource type declared and will download
   the plugin to be able to work on these resources.

3. Review the execution plan with `terraform plan` command. This will show all
   the actions carried out by terraform to create the resources.

4. Apply the changes with `terraform apply` command.

> `terraform show` can be used to check all the resources that has been created.
> This information is inspected from the state file.

### Update & Destroy Infrastructure

In case we've made some changes to our .tf file. We can run the `terraform plan`
command before publishing the resource. This will show all the changes in a
`diff` format.

Terraform will first delete the resouce and create a new resouce. This can be
seen in the terminal with `-+` symbol. This type of Infrastructure is called
**Immutable Infrastructure**.

To completely remove the infrastructure run `terraform destroy` command. This
will show the execution plan as well which can be seen with `-` symbol.

### Providers

On running `terraform init`, terraform downloads & installs plugins for the
providers used within the configuration. These plugins can be for the cloud
providers such GCP, AWS, Azure or local provider. It uses plugins based
architecture to work with hundreds of such infrastructure platforms.  
These plugins are available at registry.terraform.io

There are 3 kinds of providers:

1. **Official providers**: These are owned and maintained by Hashicorp.
2. **Partner providers**: These are owned and maintained by 3rd Party technology
   comapany that has gone through a partner provided process from Hashicorp.
   Some examples are heroku, digital ocean, etc.
3. **Community providers**: These are published and maintained by individual
   contributors.

These plugin version are shown when you run `terraform init` command. It is a
safe command to run as many times as required without affecting the
infrastructure.

> The plugin is installed in a hidden directory called `.terraform` folder
> within the working directory.
>
> The plugin name is shown in the format `hashicorp.local` which is also known
> the source address. This is an identifier used by terraform to locate and
> download the plugin from the registry.
>
> 1. The first part of the name is the **organizational namespace**.
> 2. This is followed by the type of the provider (such as local, google, azure,
>    aws)
> 3. The plugin could also have the hostname in front which is where the plugin
>    is located. If omitted, it defaults to `registry.terraform.io`.

### Configuration Directory

We can have multiple `.tf` files in a directory. Also, there could be multiple
configuration blocks created within a single configuration file with multiple
providers. A common naming conventions for such a configuration file is to call
it `main.tf`. There are other configuration files that can be created within the
directory such as:

| File Name    | Purpose                                               |
| ------------ | ----------------------------------------------------- |
| main.tf      | Main configuration file containing resource defintion |
| variables.tf | Contains variable declaration                         |
| outputs.tf   | Contains output from resources                        |
| provider.tf  | Contains Provider definition                          |

- Using Multiple Providers:

```terraform
resource "local_file" "pet" {
    filename = "~/pets.txt"
    content = "we love pets"
}

resource "random_pet" "my-pet" {
    prefix = "Mrs"
    separator = "."
    length = "1"
}
```

We need to re-run `terraform init` if a new provider is added before we can make
use of it.

### Using Input Variables

- Defining Variables inside the `variables.tf` file.

```terraform
variable "filename" {
    default = "/root/pets.txt"
}
variable "content" {
    default = "We love pets!"
}
variable "prefix" {
    default = "Mrs"
}
variable "separator" {
    default = "."
}
variable "length" {
    default = "1"
}
```

- Using variables inside the `main.tf` file

```terraform
resource "local_file" "pet" {
    filename = var.filename
    content = var.content
}

resource "random_pet" "my-pet" {
    prefix = var.prefix
    separator = var.separator
    length = var.length
}
```

- Understanding the variables block

> The variable block can use three parameters:
>
> 1. default: To specify the default value of a parameter
> 2. type: They are optional as well but when used it enforces the type of
>    varable being used.
> 3. description: They are optional but are a good practice to describe what the
>    varilable is used for.

| Type   | Example                |
| ------ | ---------------------- |
| string | "/root/pets.txt"       |
| number | 1                      |
| bool   | true/false             |
| any    | Default Value          |
| list   | ["foo", "bar"]         |
| map    | pet1 = cat             |
|        | pet2 = dog             |
| object | Complex Data Structure |
| tuple  | Comples Data Structure |

- Examples of list, map, object & tuple

```terraform
# List
variable "prefix" {
    default = ["Mr", "Mrs", "Sir", "Madam"]
    type = list(string)
}

# Map
variable file-content {
    type = map(string)
    default = {
        "statement1" = "We love pets!"
        "statement2" = "We love animals"
    }
}
variable "pet_count" {
    default = {
        "dogs" = 3
        "cats" = 1
        "cow" = 2
    }
    type = map(number)
}

# Sets :Similar to list but can't have duplicates
variable "prefix" {
    default = ["Mr", "Mrs", "Sir"] # Incorrect ["Mr", "Mrs", "Sir", "Sir"]
    type = set(string)
}

# Objects can be used to create complex data structure combining all above
variable "bella" {
    type = object ({
        name = string
        color = string
        age = number
        food = list(string)
        favorite_pet = bool
    })
    default = {
        name = "bella"
        color = "brown"
        age = 8
        food = ["Dosa", "Pav Bhaji", "Pani Puri"]
        favorite_pet = true
    }
}

# Tuples : Similar to list but we can use element of variable types
variable kitty {
    type = tuple([string, number, bool])
    default = ["cat", 8, true]
}
```

- Using Variables in Terraform

1. In case no default value is provided inside the variable block, then we'll
   prompted at the time of applying the configuration file in interactive mode.
2. Alternatively, we can use the command line flags while applying as shown
   below.

```bash
terraform apply -var "filename=/root/pets.txt" -var "content=we love pets!" \
-var "prefix=Mrs" -var "separator=." -var "length=2"
```

3. We can also use environment varialbles as shown below with
   TF*VAR*<declared_variable>

```bash
export TF_VAR_filename="/root/pets.txt"
export TF_VAR_content="We love pets!"
export TF_VAR_prefix="Mrs"
export TF_VAR_separator="."
export TF_VAR_length="2"
terraform apply
```

4. We can also use **Variable Definition File** as shown below. These variable
   definition files should always end with `terraform.tfvars` or `*.auto.tfvars`
   `terraform.tfvars.json` or `*.auto.tfvars.json` and will be automatically
   loaded.

```tfvars.json
filename="/root/pets.txt"
content="We love pets!"
prefix="Mrs"
separator="."
length="2"
```

If in case they are named anything else other than the mentioned example above
like `variables.tfvars` then they should be passed with the command line flag
`-var-file` as shown below.

```bash
terraform apply -var-file variables.tfvars
```

> [!NOTE]  
> Terraform uses `Variable Definition Precedence` order to determine which
> variables should be used in case variables are passed with all 4 examples
> above together.
>
> 1. It loads the environemnt variables first.
> 2. Next from `terraform.tfvars`.
> 3. Third from `*.auto.tfvars` in alphabetical orders.
> 4. Finally it considers the command line flag of `-var` or `-var-file` with
>    the **_highest priority_**.

### Resource Attributes

They are used to link two resources together and to bound the ouput of one
resouce with the other.

```terraform
resource "local_file" "pet" {
    filename = var.filename
    content = "My favorite pet is ${random_pet.my-pet.id}"
}

resource "random_pet" "my-pet" {
    prefix = var.prefix
    separator = var.separator
    length = var.length
}
```

### Resource Dependencies

In the above example, terraform first creates `random_pet` resource and then
`local_file` resouce. When these resouce are deleted, they are deleted in
reverse order `local_file` first and then `random_pet`. This type of dependency
is called **_Implicit Dependency_**.  
There is another way to specify the dependencies within the configuration file
using the `depends-on` arguent as shown below. It ensures that `local_file` is
only created after `random_pet`. This is known as **_Explicit Dependency_**.

```terraform
resource "local_file" "pet" {
    filename = var.filename
    content = "My favorite pet is Mr.Cat"
    depends_on = [
        random_pet.my-pet
    ]
}
resource "random_pet" "my-pet" {
    prefix = var.prefix
    separator = var.separator
    length = var.length
}
```
