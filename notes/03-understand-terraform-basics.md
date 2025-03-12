# Understand Terraform Basics

## Files and Directories

- Terraform code is stored in plain text files with the `.tf` file extension.
- There's also a JSON variant of the languag where the files are stored with the `.tf.json` file extension.
- Files containing Terraform code are often referred to as **configuration files**
- Terraform configuration filses must always use UTF-8 encoding and use Unix-style line-endings (LF)
- A Terraform **module** is a collection of `tf` and/or `.tf.json` files kept together in a directory
- A Terraform module only consists of the top-level configuration files in a directory. Nested directories are treated as seperate modules, and are not automatically included in the configuration.
- Terraform evaluates all the configuration files in a module, effectively treating the configuration as a single file.
- Terraform always runs ni the context of a single **root** module
- In Terraform CLI, the root module is the working directory where Terraform is invoked. In Terraform Cloud and Terraform Enterprise, the root module defaults to the top-level configuration directory.

- Terraform expects all configuration files to define a distinct set of configuration objects. If any two files define the same object, Terraform returns an error.
- For the rare cases where we might want to override some configuration, Terraform allows us to use configuration files with names ending in `_override.tf` or `_override.tf.json`.
- Terraform initially skips these override files when loading configuration, and then processes each override file in lexicographical order.

For example, if we have a file `example.tf` defining some configuration:

```tf
resource "aws_instance" "web" {
    instance_type = "t2.micro"
    ami           = "ami-408c7f28"
}
```

We can create a `override.tf" file with the following contents

```tf
resource "aws_instance" "web" {
    ami = "foo"
}
```

Terraform will merge the override values into the original configuration declaration, and will behave as if the original was defined as follows:

```tf
resource "aws_instance" "web" {
    instance_type = "t2.micro"
    ami           = "foo"
}
```

- The `depends_on` meta-argument may not be used in override blocks. It will cause an error.
- A Terraform configuration may refer to two different kinds of external dependencies that come from outside its codebase:
  - Providers - plugins for Terraform to extend it with support for interacting with various external systems
  - Modules - these allow splitting out group of Terraform configuration constructs into reusable abstractions
- Both of these dependency types can be published and updated intependently from Terraform and from the configuration that depends on them.
- Terraform must determine which versions of these dependencies are compatible with the current configuration.
- Terraform remembers the decisions it made in a **dependency lock file** so that it can, be default, make the same decisions again in the future.
- Currently, Terraform locks provider dependencies.
- The dependency lock file can be found in the same directory containing the root module
- The dependency lock file is always named `.terraform.lock.hcl`
- The lock file is automatically created or updated when we run the `terraform init` command
- The lock file should be tracked via the version control system we're using
- Terraform test files use the `.tftest.hcl` and `tftest.json` file extensions
- Terraform will load all test files within your root configuration directory. We can override the fault testing directory by appending the `-test-directory` flag

## Terraform Syntax

- The Terraform language syntax is built around two key syntax constructs: **arguments** and **blocks**
- An argument assigns a value to a particular name

```tf
image_id = "abc123"
```

The argument before the equals sign is the argument name and the expression after the equals sign is the argument's value

- The context where the argument appears determines what values are valid.
- A block is a container for other content:

```tf
resource "aws_instance" "web" {
    ami = "abc123"
    network_interface = {
        # ...
    }
}
```

- A block has a `type` (`resource` in the above example)
- Each block type defines how many labels must follow the type keyword. The `resource` block expects two labels
- Argument names, block type names, and the names of most Terraform-specific constructs like resources, input variables, etc, are al `identifiers`
- Identifiers can contain letters, digits, underscores, and hyphens.
- The first character of an identifier cannot be a number
- Single-line comments in Terraform begin with the pound (#) sign or double-slashes (//)
- Multi-line comments in Terraform are denoted with the slash-asterik syntax (/\* \*/)
- At the root of a JSON-based Terraform configuration file is a JSON object. The properties of this object correspond to the top-level block types of the Terraform language

```json
{
  "resource": {
    "aws_instance": {
      "web": {
        "ami": "abc123"
      }
    }
  }
}
```

This is equivalent to this native syntax:

```tf
resource "aws_instance" "web" {
    ami = "abc123"
}
```

JSON values interpretes as expressions are mapped as follows

| JSON    | Terraform Language Interpretation                                                                  |
| ------- | -------------------------------------------------------------------------------------------------- |
| Boolean | A literal `bool` value                                                                             |
| Number  | A literal `number` value                                                                           |
| String  | Parsed as a string template and evaluated as descibed below                                        |
| Object  | Each property is mapped per this table, producing an `object(...)` value with suitable types       |
| Array   | Each property is mapped per this table, producing a `tuple(...)` value with suitable element types |
| Null    | A literal `null`                                                                                   |

## Terraform Resources

- Each resource block describes one or more infrastructure objects, such as vnets, compute instances, or higher-level components such as DNS records
- A `resource` block declares a resources of a specific type with a specific local name. Terraform uses the local name to refer to the resource in the same module. The name means nothing outside the module.

```tf
resource "aws_instance" "web" {
    ami           = "ami-abc123"
    instance_type = "t2.micro"
}
```

Here we define an `aws_instance` resource type named `web`. The resource type and name must be unique within a module. i.e, we can't define another `aws_instance` resource type named `web` within the same module

- The resource arguments often depend on the resource type
- A provider is a plugin for Terraform that offers a collection of resource types
- Based on the resource type, Terraform can usually determine which provider to use
- By convention, resource types start with the name of the provider. i.e `aws_instance` is from the `aws` provider
- Terraform resources can be used with these meta-arguments, which can be used with any resource type to change the behavior of the resources:
  - `depends_on` - For specifying hidden dependencies
  - `count` - For creating multiple resource instances according to a count
  - `for_each` - To create multiple instances according to a map, or a set of strings
  - `provider` - To select a non-default provider configuration
  - `lifecycle` - To customize the lifecycle of the resource
  - `provisioner` - To take extra actions after the resource creation
- To remove a resource we can simply remove it from our configuration. On the next run of `terraform apply`, the resource will be deleted from our cloud infrastructure.
- If we want to remove the resource from our configuration, but do not delete it from our cloud infrastrcture, we can remove the `resource` block from our configuration and replace it with a `removed` block

```tf
removed {
    from aws_instance.web

    lifecycle = {
        destroy = false
    }
}
```

The `from` keyword is the address of the resource we want to remove
The `lifecycle` block is required.  The `destroy` argument determines whether the resource will be destroyed or not. A value of `false` means Terraform will remove the resource from the state without destroying it

- We can use `precondition` and `postcondition` blocks to specify assumptions and guarantees about how the resource operates.

```tf
resource "aws_instance" "web" {
    instance_type = "t2.micro"
    ami           = "ami-abc123"

    lifecycle {
        precondition {
            condition     = data.aws_ami.example.architecture == "x86_84"
            error_message = "The selected AMI must be for the x86_64 architecture."
        }
    }
}
```

- Some resource types provide a special `timeouts` nested block argument that allows us to customize how long certain operations are allowed to take before being considered to have failed. 

```tf
resource "aws_db_instance" "db" {
    // ... 
    
    timeouts {
        create = "60m"
        delete= "2h"
    }
}
```

## Terraform Data Sources

[Terraform Data Sources](https://developer.hashicorp.com/terraform/language/v1.1.x/data-sources)

## Terraform Providers

[Terraform Providers](https://developer.hashicorp.com/terraform/language/v1.1.x/providers)

## Terraform Variables and Outputs

[Terraform variables and outputs](https://developer.hashicorp.com/terraform/language/v1.1.x/values)

## Terraform Modules

[Terraformo Modules](https://developer.hashicorp.com/terraform/language/v1.1.x/modules)

## Terraform Expressions

[Terraform Expressions](https://developer.hashicorp.com/terraform/language/v1.1.x/expressions)

## Terraform Functions

[Terraform Functions](https://developer.hashicorp.com/terraform/language/v1.1.x/functions)

## Terraform Settings

[Terraform Settings](https://developer.hashicorp.com/terraform/language/v1.1.x/settings)

## Terraform State

[Terraform State](https://developer.hashicorp.com/terraform/language/v1.1.x/state)
