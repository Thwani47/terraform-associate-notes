# Read, Generate, and Modify Configuration

## Input Variables

- Input variables enable us to customize aspects of Terraform modules without altering the module's source code.
- Input variables make moduels reusable and composable
- Variables declared in child modules need to be provided by calling modules
- We can also pass variable values via the CLI or environment variables
- We can compare modules to functions in traditional programming languages
  - input variables are akin to function arguments
- A Terraform is declared via a `variable` block

```tf
variable "image_id" {
    type = string
}
```

- The label after the `variable` keyword (the variable name) has to be unique in the module
- The variable name cannot be one of (`source`, `version`, `providers`, `count`, `for_each`, `lifecycle`, `depends_on`, `locals`) as these are reserved Terraform keywords
- A variable declaration has these arguments
  - `default` - the default value to be assigned to the variable.
    - If the default value is specified, the variable becomes optional
  - `type` - The data type for the variable. The type is optional
    - A Terraform variable can be one of these types
      - `string`
      - `number`
      - `bool`
      - `list(<TYPE>)`
      - `map(<TYPE>)`
      - `set(<TYPE>)`
      - `object(<TYPE>)`
      - `tuple([<TYPE>, ...])`
    - `description` - A text describing the variable
    - `validation` - A block to define validation rules
      - Values that do not meet these validation rules are not accepted
    - `ephemeral` - This marks the variable as ephemeral and it is not written to the state file or plan file(s)
      - An expression that references ephemeral variables is implicitly ephemeral
    - `sensitive` - Thi smarks the variable as sensitive and it is not outputed to the standard output
      - Sensitive variables are written in plain-text in the state file
    - `nullable` - This sets whether the variable can be null or not

```tf
variable "username" {
    type        = string
    default     = "user123"
    description = "The username to be used to log in to the VM"
    validation  = {
        condition     = length(var.username) > 6
        error_message = "The username has to be at least 6 characters long"
    }
    ephemeral   = false
    sensitive   = false
    nullabe     = false
}
```

```bash
$> terraform apply -var="username=testuser123"
$> terraform apply -var-file="testing.tfvars"
$> export TF_VAR_username="testuser123"
$> terraform apply
```

## Output Values

- Output make information about your infrastructure available on the command line
- We can also use output values to expose information to other Terraform modules
- Thinking of modules as functions, we can think of output values as return values in traditional programming languages
- Output values are declared using the `output` block

```tf
output "instance_ip_addr" {
    value = aws_instance.sever.private_ip
}
```

- The output name has to be unique in the module it is defined in
- The output has a value argument that takes an expression whose result is returned to the user
- Parent modules accessing output values from child modules can use the syntax `<MODULE-NAME>.<OUTPUT-NAME>`, for example, a child module named `web_server` that exposes an output named `instance_ip_addr` can be accessed by `module.web_server.instance_ip_addr`
- We can add a `precondition` block in our output declaration to run custom checks to ensure our output meets certain conditions
- We can optionally add `description`, `sensitive`, `ephemeral` and `depends_on` arguments on the output declaration

## Local Values

- A local value assigns a name to an expression so it can be used multiple times within the module
- A local value is similar to a local variable within a function
- Local variables are declared in a single `locals` block

```tf
locals {
    service_name = "forum"
    owner        = "Community Team"
    common_tags  = {
        Service = local.service_name
        Owner   = local.owner
    }
}
```

- Local values are implicitly ephemeral if they reference an ephemeral variable or output

```tf
variable "service_name" {
    type       = "string"
    default   = "forum"
    ephemeral = true
}

locals {
    service_tag = "${var.service_name}"
}
```

`local.service_tag` is ephemeral

- Local values can be used within the module using `local.<LOCAL-VALUE-NAME>`

## Vault Provider

- The Vault provider allows Terraform to read from and write to HashiCorp Vault
- We can use the Vault to store secrets used in our configuration

## Type Constraints

- Terraform has these primitive types
  - string
  - number
  - bool
- Terraform will convert number and bool to string when needed, and vice-versa
- Complex types group multiple values into a single value. There are two categories of complex types
  - Collection types - used to group similar values (e.g `list(string)`, `map(string)`, `set(string)`)
    - All elements in a collection must be of the same types
  - Structural types - used to group potentially dissimilar values (e.g `object(...)`, `tuple(...)`)

## Data Sources

- Data sources allow Terraform to use information defined outside of Terraform, defined by another separate Terraform configuration, or modified by functions
- A data source is declared using a `data` block

```tf
data "aws_ami" "example" {
    most_recent = true

    owners      = ["self"]
    tags        = {
        Name   = "app-server"
        Tested = "true"
    }
}
```

- Data sources cause Terraform to only read objects

## Resource Address Reference

- A resource address is a string that identifies zero or more resource instances in your configuration
- `module.module_name[module index]`
- `resource_type.resource_name[instance index]`
  - The index is a 0-based integer N for instances defined by the `count` meta-argument
  - The index is an alphanumerical key to index a resource with multiple instances defined by the `for_each` meta-argument

```tf
resource "aws_instance" "web" {
    count = 4
}
```

```bash
$> aws_instance.web[2] # get the 3rd instance
$> aws_instance.web # get all the "web" aws_instances 
```

```tf
resource "aws_instance" "web" {
    for_each = tomap({
        "terraform" : "value1",
        "resource" : "value2",
        "indexing": "value3",
        "example" : "value4"
    })
}
```

```bash
$> aws_instance.web["terraform"]
```

## References to Named Values

- Terraform has these named values avaialable
  - Resources
    - `<RESOURCE TYPE>.<NAME>` represents a managed type of the given type and name
  - Input variables
    - `var.<NAME>`
  - Local values
    - `local.<NAME>`
  - Child module outputs
    - `module.<MODULE NAME>.<OUTPUT NAME>`
  - Data sources
    - `data.<DATA TYPE>.<NAME>`
  - Filesystem and workspace info
  - Block-local values

## Dependency Graph

- The following node types can exist within the graph
  - The resource node
  - The provider configuration node
  - The resource meta-node
- A standard DFS traversal is done to walk the graph
- A node is walked as soon as all its dependencies are walked
