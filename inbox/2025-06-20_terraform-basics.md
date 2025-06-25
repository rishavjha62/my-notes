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

- **resource Type**: Following resource we've the declaration of the resource
  Type that we want to create. This is a fixed value and depends on the provider
  where we want to create the resource. In this example we've the local resource
  with the type being file `[local = provider, file = resource]`.

- **resource Name**: This is logical name that can be used to identify the
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
    machine_type = "n1-standard-2"
    network_interface {
        network = "default"
    }
    boot_disk {
        initialize_params {
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

Terraform will first delete the resource and create a new resource. This can be
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

```json
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

### resource Attributes

They are used to link two resources together and to bound the ouput of one
resource with the other.

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

### resource Dependencies

In the above example, terraform first creates `random_pet` resource and then
`local_file` resource. When these resource are deleted, they are deleted in
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

### Output Variables

The output variables could be used to store the value of an expression in
terraform. For example, the `random_pet` resource will generate a random_pet
name using the attribute called `id` when we apply the configuration. To save
this output in a variable called `pet-name`, we could create an output block as
shown below:

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

output "pet-name" {
    value = random_pet.my-pet.id
    description = "Record the value of pet ID generated by the random_pet resource."
}
```

Once the block has been created and we've run the configuration file by
`terraform apply`, we can see the output on the screen. We can also use the
`terraform output` command as shown below.

```bash

# to print all output variables defined in all the files in the current configuration directory
terraform output

# to print a particular variable
terraform output <variable_name>
```

- Use of output variables:

1. When to quickly display details about the provisioned resources on the
   screen.
2. Feed the output variables to other IAC tools such as Ansible or a ad-hoc
   script.

## Terraform State

### Local State

When `terraform plan` is ran, it checks the state file present inside the
working directory. The file named `terraform.tfstate` is present which is a JSON
data structure that maps the real world infrastructure resources to the resource
definition in the configuration files.  
It is the single source of truth when running commands such as `terraform plan`
& `terraform apply`.

If a change is made to the configuration file and `terraform plan` or
`terraform apply` command is run, terraform by default refreshes the state again
compares it against the configuration files.

- Purpose of state

1. **_Mapping Configuration to Real World_**: State files are blueprints of all
   the resources that terraform manages. It records the identity of the resource
   it creates in this state file. Each resource will have a unique id.
2. **_Tracking Metadata_**: It also tracks the metadata details such as resource
   dependencies. When deleting a resource with dependicies, terraform will rely
   on the state file.
3. **_Performance_**: When dealing with 100s and 1000s of resources, distributed
   across multiple providers, it is not feasible for terraform to reconcile
   state for every terraform operation as it would take a lot of time to fetch
   details from all the providers. In such cases terraform state could be used
   as a record of truth without having to reconcile. We can bypass having to
   refresh state every time by `terraform plan --refresh=false`.
4. **_Collaboration_**: It is recommended to store the terraform state file in a
   remote data store for collaboration and security purposes.

- Considerations for state file:

1. State file contains sensitive information, thus need to be stored in remote
   data storage rather than than on version control system.
2. The state file shouldn't be updated manually. Changes however can be made
   using the `state` commands.

### Remote State:

- Remote State  
  Storing the terraform state file in a remote stroage bucket like google cloud
  stroage bucket, or amazon s3.

- State Locking  
  If two people update the same resource at the same time, it could lead to
  unintended consequence such as corruption of the state file. Terraform can
  protect itself when concurrent operation are run against same configuration by
  locking the state file. This feature is called **_State Locking_**.

- Configuring remote backend with s3

We need 2 things to configure remote backend for terraform state file.

- storage bucket (either GCS bucket or aws s3 bucket, etc) to store state file.
- A Database (either Cloud SQL, Firestore or Amazons DynamoDB) for state locking
  & consistency checks.

Example with GCS: developer.hashicorp.com/terraform/language/backend/gcs

```terraform
terraform {
  backend "gcs" {
    bucket  = "terraform-state-bucket-unique"
    prefix  = "terraform/state"  # Folder structure within bucket
  }
}
```

Example with AWS s3: developer.hashicorp.com/terraform/language/backend/s3

```terraform
terraform {
backend "s3" {
bucket= "my_bucket_name"
key = "path/to/my/key"
region = "us-west-1"
use_lockfile = true
}
```

> [!Note]  
> State locking has to be enabled in s3 by setting the arguemnt
> `use_lockfile = true`.

### Terraform state commands

Note that the terraform state file should not be updated manually. In case a
change is required, it can be done using `terraform state` command.

Syntax for terraform state commands:

```bash
terraform state <sub-command> [options] [args]
```

The sub commands could be:  
| table |  
| --------- |  
| list |  
| mv |  
| pull |  
| rm |  
| show |

- `list` : The list command could be used to display all the resources recorded
  within the terraform state file. Can be used with arguments which can be
  resource name.

- `show`: The show command will provide detailed information about the resource
  and can also be passed with arugment as the resource name.

- `mv`: Used to move items in a terraform state file. Resource can be moved
  within the source file which means renaming the resource or from once state
  file to another state file.

```bash
terraform state mv [options] SOURCE DESTINATION
```

- `pull`: To downlaod and view the resources in a remote state file. This output
  can passed to tools like like `jq` to filter the data.

- `rm`: To delete items from terraform state file and can be used with argument

## Terraform Commands

```bash
# To check if the syntax is correct
terraform validate

# To format the terraform configuration files
terraform fmt

# Prints the current state of the infrastructure as seen by terraform
terraform show
# to view the json format output
terraform show -json

# To see list all providers used in configuration files
terraform providers
# To copy provider plugins to another directory
terraform providers mirror </your/new/path>

# To view all output variables in the configuration directory
terraform output
# for a specific variable
terraform output <variable_name>

# To sync with the real world infrastructure
terraform apply -refresh-only

# To create a visual representation of the dependicies in the configuration
terraform graph
# this output can be created in graph using graphviz (external tool)
terraform graph | dot -Tsvg > graph.svg
```

### Mutable vs Immutable Infrastructure

Mutable infrastructure allows for updates to be made to existing servers, while
immutable infrastructure requires creating a new instance for any update,
ensuring consistency & reducing risks associated with configuration drift.

### Lifecycle Rules

You may want to first create a new resource and then delete the old resource or
not want the resource to be deleted at all even if there is a change made in its
local configuration. This can be achieved in terraform by using **Lifecycle
Rules**.

```terraform
resource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = "We love pets!"
    file_permission = "0700"

    lifecycle {
        create_before_destroy = true
    }
}
```

- `create_before_destroy = true` ensures that a new resource is created before
  deleting old one.
- `prevent_destroy = true` rejects any changes that will result in resource
  getting destroyed and will throw an error.

> ![Note]  
> `terraform destroy` will still delete these resources.

- `ignore_changes = all` will prevent a resource from being updated based on a
  list of attributes that we defined within the lifecycle block. The
  `ignore_changes` block will accept a list of attributes which needs to be
  ignored for any changes as shown below.

```terraform
resource "aws_instance" "webserver" {
    ami = "ami-030493849775"
    instance_type = "t2.micro"
    tags = {
        Name = "ProjectA-Webserver"
    }
    lifecycle {
        ignore_changes = [
        tags, ami
        ]
    }
}
```

## Datasources

Since infrastructure can be provisioned using other tools such as Chef, Puppet,
Cloud Formation, Salt Stack, Ansible, Ad-hoc script, manually, etc.  
Datasources allow terraform to read attibutes from resources which are
provisioned outside its control. We can define a data block within the
configuration file to read data as shown below.

```terraform
resource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = data.local_file.dog.content
}

data "local_file" "dog" {
    filename = "/root/dog.txt"
}
```

## Meta Arguments

Meta Arguments can be used with every resource type to modify the behaviour of
resource. These can help manage dependencies, create multiple instances, and
control resouce lifecycle actions.

- depends_on -> For defining explicit dependencies
- lifecycle -> Defines how the resources should be created, updated & destroyed
- count
- for_each
- provider

1. count: To create multiple instances for a resource. Value should be greater
   than 1.

variables.tf

```terraform
variable "filename" {
    default = [
    "roots/pets.txt"
    "roots/dogs.txt"
    "roots/cat.txt"
    ]
}
```

main.tf

```terraform
resource "local_file" "pet" {
    fileaname = var.filename[count.index]
    count =  length(var.filename)  #3
}
```
