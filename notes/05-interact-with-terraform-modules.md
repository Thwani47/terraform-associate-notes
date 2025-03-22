# Interact with Terraform Modules

## Variables and Outputs

- Input variables serve as parameters for a Terraform module, so users can customize behavior without editing the source
- Output variables are like return values for a Terraform module
- Local variables are a convenience feature for assigning a short name to an expression

### Input Variables

- Input variables enable us to customize aspects of Terraform modules without altering the module's source code
- We can share modules across different Terraform configurations, making the modules composable and reusable
- It can be useful to compare Terraform modules to function definitions in traditional programmnig languages
  - Input variables are like function arguments
  - Output variables are like function return values
  - Local values are like a function's temporary local variable
- Each input variable accepted by a module must be declared using a `variable` block

```tf
    variable "image_id" {
        type = string
    }

    variable "availability_zones_names" {
        type    = list(string)
        default = ["us-west-1a"]
    }

    variable "docker_ports" {
        type = list(object({
            internal = number
            external = number
            protocol = string
        }))

        default = [
            {
                internal = 8300
                external = 8300
                protocol = "tcp"
            }
        ]
    }
```

- The label after the `variable` keyword is the name of the variable and it must be unique in the module
- The name of the variable can be anything except `source`, `version`, `providers`, `count`, `for_each`, `lifecycle`, `depends_on`, and `locals`
- A variable declaration also has these optional arguments:
  - `default` - a default value to be used which makes the variable optional
    - This expects a literal value and cannot reference other objects in the configuration
  - `type` - this specifies the value types that are accepted for the variable
    - Supported type keywords are
      - `string`
      - `number`
      - `bool`
      - `list(<type>)` (list(string), list(number), etc)
      - `set(<type>)` (set(string), set(number), etc)
      - `map(<type>)`
      - `object({<ATTR_NAME> = <TYPE>, ...})`
      - `tuple([<TYPE>, ...])`
    - The keyword `any` may also be used to indicate that any value of any type may be used
    - If both the `type` and `default` arguments are specified, the given default value must satisfy the type constraints
  - `description` - this specifies the input variable's documentation
  - `ephemeral` - This specifies that the input variable is only available during runtime and it is not written to the state file
    - These can only exist in certain contexts:
      - In a write-only argument
      - Another ephemeral variable
      - In local values
      - In ephemeral resources
      - In ephemeral outputs
      - When configuring providers in the provider block
      - In provisioner and connection blocks
    - If another expression references an ephemeral variable, that expression implicitly becomes ephemeral
  - `sensitive` - The marks the variable as sensitive and limits Terraform UI output
    - These are hidden from standard output but will be stored as clear text in the state and plan files
    - A provider might disclose a sensitive value if the value is part of an error message
  - `nullable` - This specifies if the variable can be null or not within the module
    - The default for this field is true
    - If nullable is set to false and the variable declaration has a default value provided, Terraform will use the default value
- We can add custom validation rules for a particular variable by adding a `validation` block within the corresponding `variable` block

```tf
variable "image_id" {
    type        = string
    description = "The id of the machiine image (AMI) to use for the server"

    validation {
        condition     = length(var.image_id) > 4 && substr(var.image_id, 0, 4) == "ami-"
        error_message = "The image_id value must be a valid AMI id, starting with 'ami-'. "
    }
}
```

- An input variable value can be accessing from within expressions as `var.<NAME>` where `<NAME>` matches the label given the declaration block

```tf
variable "image_id" {
    type = string
}

resource "aws_instance" "web" {
    instance_type = "t2.micro"
    ami           = var.image_id
}
```

- We can specify input variable values via the command line using the `-var` option when running `terraform plan` and `terraform apply

```bash
> terraform apply -var="image_id=ami-abc123"
> terraform apply -var='image_id_list=["ami-abc123", "ami-def456"]'
> terraform apply -var='image_id_map={"us-east-1": "ami-acb123", "us-east-2": "ami-def456"}'
```

- We can also use variable definition files (`.tfvars`) to specify variable values

```bash
> terraform apply -var-file="devtest.tfvars"
```

- We can also use environment variables to provide input variable values
- These env vars need to have the prefix `TF_VAR_`

```bash
> export TF_VAR_image_id=ami-abc123
```

- Input variable sources have the following order of precedence:
  - environment variables
  - `terraform.tfvars` if present
  - `terraform.tfvars.json` if present
  - Any `*.auto.tvars` or `*.auto.tfvars.json` files, processed in the lexical order of their file names
  - Any `-var` and `-var-file` options on the command line

### Output Values

- Output values make information about your infrastructure available on the command line, and can expose information for other Terraform configurations to use
- A child module can use output values to expose a subset of its resource attributes to a parent module
- A root module can use outputs to print certain values to standard output after running `terraform apply`
- Each output value must be declared using an `output` block

```tf
output "instance_ip_addr" {
    value = aws_instance.web.private_ip
}
```

- We can access outputs defined in a child module using `module.<MODULE_NAME>.<OUTPUT_NAME>`. For example, a child module named `web_server` declares an output named `instance_ip_addr`, we could access that value as `module.web_server.instance_ip_addr`
- We can use `precondition` blocks to run custom conditional checks to specify guarantees about the output

```tf
output "api_base_url" {
    value = "https://${aws_instance_web.private_dns}:8433/"

    precondition {
        condition     = data.aws_ebs_volume.example.encrypted
        error_message = "The server's root volume is not encryped"
    }
}
```

- `output` blocks can optionally include `description`, `sensitive`, `depends_on`, and `ephemeral` meta-arguments
- We cannot set an output value as ephemeral in the root module

### Local Values

- A local value assigns a name to an expression so you can use that expression multile times within a module instead of repeating the expression
- A set of related local values can be declared together in a single `locals` block

```tf
locals {
    service_name = "forum"
    owner`       = "Community Team"
    instance_ids = concat(aws_instance.blue.*id, aws_instance.green.*.id)
    common_tags  = {
        Environment = "production"
        CostCenter  = "Technology"
    }
}
```

- Local values implicitly become ephemeral if they reference an ephemeral value

```tf
variable "service_name" {
    type    = string
    default = "forum"
}

variable "environment" {
    type    = string
    default = "dev"
}

variable "service_token" {
    type      = string
    ephemeral = true
}

locals {
    service_tag    = "${var.service_name}-${var.environment}"
    sessiont_token = "${var.service_name}:${var.service_token}"
}
```

The value `local.session_token` value is implicitly ephemeral because it relies on `sevice_token`, which is ephemeral

- Local values can be referenced using `local.<NAME>`

```tf
resource "aws_instance" "web" {
    # ...
    tags = local.common_tags
}
```

## Modules

- Modules are containers for multiple resources that are used together.
- A module consists of a collection of `.tf` or `.tf.json` files kept together in a directory
- Modules are the main way to package and reuse resource configurations with Terraform
- Every Terraform configuration has at least one module, known as the root module.
- The root module consists of resources defined in the `.tf` files in the main directory
- A Terraform module can call other modules to include their resources into the configuration
- A module that has been called by another module is reffered to as the child module
- We can load modules from the local filesystem or from a public or private registry
- We can also publish modules for others to use
- Modules are called from within other modules using `module` blocks

```tf
module "servers" {
    source  = "./app-cluster"
    servers = 5
}
```

- The `source` argument is mandatory for all modules
  - The source is either the path to a local directory containing the module's configuration files or a remote module source that Terraform should download and use
  - After adding, removing, or modifying module blocks, we must run `terraform init` to allow Terraform to adjust to the changes
  - `terraform init` will not upgrade previously downloaded modules. We need to pass in the `-upgrade` option to `terraform init` to download newer versions of modules
- The `version` argument is mandatory for all modules from a registry

  ```tf
  module "consul" {
      source  = "hashicorp/consul/aws"
      version = "0.0.5"

      servers = 4
  }
  ```

  - if no versions meet the specified version, Terraform will download the latest available version that meets the constraint

- The module blocks also support these meta-arguments:
  - `count` - creates multiple instances of a module from a single `module` block
  - `for_each` - creates multiple instances of a module from a single `module` block
  - `providers` - passes provider configurations to a chile module
  - `depends_on` - creates explicit dependencies between the entire module and the listed targets
- Child modules can declare output values to enable the calling modules access to some resource attributes
- To remove a module from Terraform we can simply remove the module block from the configuration
- If we want to remove the module but not destroy the resources created under that module, we can add the module in a `removed` block and remove the module from the configuration

```tf
removed {
    from  = module.example

    lifecycle {
        destroy = false
    }
}
```

- The Terraform module installer supports installation from a number of different module source types

  - Local paths

    ```tf
    module "consul" {
    source = "./consul"
    }
    ```

    - A local path must begin with either `./` or `../`

  - Terraform Registry

    ```tf
    module "consul" {
        source  = "hashicorp/consul/aws"
        version = "0.1.0"
    }

    module "another-consul" {
        source  = "app.terraform.io/example-corp/k8s-cluster/azurerm"
        version = "1.1.0"
    }
    ```

  - GitHub

    ```tf
    module "consul" {
        source = "github.com/hashicorp/example"
    }

    module "consul" {
        source = "git@github.com:hashicorp/example.git"
    }
    ```

  - BitBucket

    ```tf
    module "consul" {
        source = "bitbucket.org/hashicorp/terraform-consul-aws"
    }
    ```

  - Generic Git, Mercurial repositories
  - HTTP URLs
  - S3 Buckets
  - GCS Buckets
  - Modules in package sub-directories

- If the child module does not declare any configuration aliases, the child module will inherit all default non-alised providers
- The standard module structure is as follows
  - Root module
  - README
  - LICENSE
  - main.tf, variables.tf, outputs.tf
  - Variables and outputs should have descriptions
  - Nested modules should exist under the `/modules` sub-folder
    - Any nested module with a README is considered available for external usage. If the README does not exist, it is considered to be for internal usage only
  - Examples - Usage examples should be in the `./examples` sub-folder. Each example can have a README explaining the goal of the example
