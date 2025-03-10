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

[Terraform Synatx](https://developer.hashicorp.com/terraform/language/v1.1.x/syntax)

## Terraform Resources

[Rerraform Resources](https://developer.hashicorp.com/terraform/language/v1.1.x/resources)

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
