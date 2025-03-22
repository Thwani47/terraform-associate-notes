# Use Terraform outside the core workflow

## Importing existing resources

- We can import existing resources to state using Terraform CLI or HCP Terraform
- To import multiple resources we use the Terraform `import` command
- Before we run `terraform import`, we need to define a `resource` block. Terraform will map the imported resource to this block
- Importing via the CLI does not generate configuration. If we want to generate configuration we need to use the `import` block
- We cannot import all Terraform resources

```tf
resource "aws_instance" "web" {
    # configuration
}
```

```bash
> terraform import aws_instance.web i-abc123
```

The Terraform import command locates an AWS instance with ID `i-abc123` and attaches it to the defined resource with the name `web`

- It is also possible to import resources in child modules, using their paths

```bash
> terraform import module.foo.aws_instance.example i-abc123
```

- If an imported resource has other resources it depends on and they are not imported as well and their configuration added to the configuration file(s), Terraform may plan to destroy these resources on the next run of `terraform plan/apply`
- We can import resources to configurations defined with `count` and `for_each`

```bash
 > terraform import 'aws_instance.web[0]' i-abc123
```

```bash
> terraform import 'aws_instance.web["example"]' i-abc123
```

- Using the `import` block is predictable, works with CICD pipelines, and allows us to preview imports before we run them and modify the state
- After importing the resources, we can optionally remove the `import` blocks or leave them there to show where the resources originated from
- We can add an `import` block to any Terraform configuration file. A common pattern is to use a `imports.tf` file, or place each `import` block beside the `resource` block it imports to

```tf
import {
    to = aws_instance.web
    id = "i-abc123"
}

resource "aws_instance" "web" {
    name = "web001"
    # ... other resource config
}
```

- The import ID must be known as plan time for the operation to succeed
- We can use `for_each` to import multiple resources

```tf
locals {
    buckets = {
        "staging" = "bucket1"
        "uat"     = "bucket2"
        "prod"    = "bucket3"
    }
}

import {
    for_each = locals.buckets
    to = aws_s3_bucket.this[each.key]
    id = eack.value
}

resource "aws_s3_bucket" "this" {
    for_each = local.buckets
}
```

- The `import` block is idempotent. Running the same import action will not generate other imports as long as the resource is imported into state.
- We can also instruct Terraform to generate HCL configuration code for the resources we're importing if we don't know how/want to write the code ourselves
- We can do this by running `terraform plan -generate-config-out="generated_resources.tf"`
- If there are any resources targeted by an `import` block but do not have any resource configuration defined, Terraform will generate configuration for these resources in the `generated_resources.tf` file
- The generated config is created during the plan stage. We can review the config and make any formatting, ordering, or any type of change we want before we apply the changes

## Debugging Terraform

- Terraform has detailed logs which can be enabled by setting the `TF_LOG` environment variable to any value. We can set the variable to one of the log levels (`TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`)
- We can also set the value to `JSON` which outputs the logs at the `TRACE` level or higher, and uses JSON encoding as the formatting
- We can also enable a subet of logging by setting the `TF_LOG_CORE` and `TF_LOG_PROVIDER` env vars
- We can choose to persist the logs by setting a `TF_LOG_PATH` env var to have the logs written to a specific file. We need to have `TF_LOG` set if we have `TF_LOG_PATH` set

## Terraform State

- Terraform uses state to map real world resources to our configuration
- It also uses state to keep track of metadata, and to improve performance for large infrastructures
- Terraform uses this state to create plans and make changes to the infrastructure
- The state, by default, is stored in a local file called `terraform.tfstate`. It can also be stored remotely, which is ideal when working in a team
- Modification of the state file is discouraged. Terraform provides a state command that we can use to performance basic state operations on the CLI
- The `terraform_remote_state` data source uses the latest state snapshop from a specified state backend to rtreive the root module output values from some other Terraform configuration
- The `terraform_remote_state` data source is available through the built-in provider `terraform.io/builtin/terraform`
- Backends are responsible for storing state and providing an API for state locking. State locking is optional
- Despite the state being stored remotely, terraform commands such as `terraform console`, `terraform state`, `terraform taint`, and more will continue to operate as if the state was local
- We can manaully retreive the state from the backend using `terraform state pull`. This will load the state and output the state to the standard output. We can choose to save the output to a file or peform other operations on it
- We can also manually write to the backend state using `terraform state push`. This will override the remote state
- Writing to remote state will fail
  - if the two states (local and remote) where created at different times. It's most likely we're updating two different states that could be for different infrastructures (uses the 'lineage' ID to track this)
  - If the remote state has newer changes than the local state (uses the 'serial' number to track this)
- We can force push by using the `-force` flag with `terraform state push`
- Not all backends support state locking
- State locking happens automatically on all operations that could write to the state
- We can use the `force-unlock` to force state unlocking
- Some backends allow for multiple workspaces, which enable multiple states to be associated with a single configuration
- Terraform starts with a single, default workspace named `default` that we cannot delete

```tf
resource "aws_instance" "web" {
    count = terraform.workspace == "default" ? 5 : 1
}
```

- When usingn local state, state is stored in plain-text JSON files
- When using remote state, state is only held in memory when using Terraform. It may be encrypted at rest (this depends on the specific remote backend used)
- Terraform allows us to mark variables and outputs as ephimeral. These won't be written to the state file or the plan files.
- Ephimeral data is useful when dealing with tokens, credentials, and other temporary data we do not want to store to state

## Terraform state command

- A resource address is a string that identifies zero or more resource instances in your overall configuration
- An address is made up of two paths

```bash
[module path][resource spec]
```

- A module path addresses a module within the tree of modules. It takes the form `module.module_name[module index]`
  - `module` indicates a chile (non-root) module. Multile `module` keywords indicate nesting
  - `module_name` is a user-defined name of the module
  - `[module index]` is an optional index to select an instance from a module call that has multiple instances.
- If the module path is omited, then the address applies to the root module
- A resource spec addresses a specific resource instance in the selected module. It has the following syntax `resource_type.resource_name[instance index]`
  - `resource_type` is the type of the resource being addresses
  - `resource_name` is the user-defined name of the resource
  - `[instance_index]` is an optional index to select an instance from a resource that has multiple instances
- The index used in modules and resources can be of two types:
  - `[N]` where N is a 0-based numerical index into a resource with multiple resources specified by the `count` meta-argument. Omitting an index when addresing resources where the count > 1 addresses all the resources
  - `["INDEX"]` where `INDEX` is an alphanumerical key index into a resource with multiple instances specified by the `for_each` meta-argument

### Examples

Given a Terraform config like this

````tf
resource "aws_instance" "web" {
    # ...
    count = 4
}`

An address like this

```tf
aws_instance.web[3]
````

addresses the last instance, and an address like this

```tf
aws_instance.web
```

addresses all instances.

Given configuration like this

```tf
resource "aws_instance" "web" {
    # ...

    for_each = tomap({
        "terraform" = "value1",
        "resource"  = "value2",
        "indexing"  = "value3",
        example     = "value4"
    })
}
```

An addresses like this

```tf
aws_instance.web["example"]
```

refers to only the `example` instance in the config, and resolves to `value4`

- The Terraform state subcommands all work with remote state as if it were local state.
- State backupds are written to disk
- Backup for state modification cannot be disabled. You can manually delete this backup files but you can't disable the backup process itself
- Terraform includes some commands to read and update state without taking any other actions
  - `terraform state list` shows the resource addresses for every resource Terraform knows about in a configuration, optionally filtered by partial resource address
  - `terraform state show` displays detailed state data about one resource
  - `terraform refresh` updates state data to match the real world condition of the managed resources. This is done automatically during plans and applies, but not when interacting with state directly
- The `terraform state show` command shows the attributes of a single resource in the terraform state
- Usage: `terraform state show [options] ADDRESS`
  - The command will show the attributes of a single resource in the state file matching the given address
  - The `terraform state list` command lists resources within a Terraform state
- Usage `terraform state list [options] [address...]`
  - The command will list all resources within the state that match a given address if any. If no addresses are given, all resources will be listed
  - `terraform state list`
  - `terraform state list docker_container.nginx`
- The `terraform state refresh` commadn reads the current settingds from all managed remote objects adn updates the Terraform state to match
  - This command is deprecated. We now use the `-refresh-only` flag to `terraform apply` and `terraform plan` commands
  - This command does not modify the remote objects but only the state
- When we need to replace an object/resource, we can use the following methods

  - Manually replace the resources

    - We can use the `-replace` flag to the `terraform plan` or `terraform apply` commands

    ```bash
    > terraform apply -replace="aws_instance.web"
    ```

    - Replace resources in a `tainted` state

      - The taintained state indicates that the resource exists but may not be in a fully-functional state. Terraform will replace all resources in a tainted state in the next plan or apply operation
      - Sometimes Terraform will itself mark resources as tainted. We can manually do this using the `untaint` command
      - We can also manually mark resources as tainted using the `taint` command, but this approach is deprectaed in favor or the `-replace` option

      ```bash
      > terraform taint docker_container.nginx
      Resource instance docker_container.nginx has been marked as tainted.

      > terraform plan
      docker_container.nginx is tainted, so must be replaced

      > terraform untaint docker_container.nginx
      Resource instance docker_container.nginx has been successfully untainted.
      ```

- The `terraform state mv` command changes a resource address in the configuration. This is used to preserve an object when renaming a resource, or when moving a resource into or out of a child module
  - Usage `terraform state mv SOURCE DESTINATION`
    - Both the source and teh destination must refer to teh same kind of object
  - `terraform state mv docker_container.nginx docker_container.test`
- The `terraform state rm` command tells Terraform to stop managing a resource as part of the current directory or workspace, without destroying the corresponding object
  - The remote object will continue to exist but will not be managed by Terraform
  - Instead of using this command, we can use the `removed` blocks
  - Usage `terraform state rm [options] ADDRESS`
- The `terraform state replace-provider` command transfers existing resources to a new provider without requiring them to be re-created
  - Usage: `terraform state replace-provider [options] FROM_PROVIDER_FQN TO_PROVIDER_FRN`
  - `terraform state replace-provider hashicorp/aws registry.acme.corp/aws`
- Terraform also allows us to recover the state from a backup when a disaster occures
  - The `terraform force-unlock` allows us to unlock the state if a `terraform apply` operation or other process stops unexpectedly before Terraform can release the lock on the state.
    - Usage `terraform force-unlock [options] LOCK_ID`
  - We can use `terraform state pull` to read the state from the configured backend
    - This downloads the state from its remote location, and updates the local copy with the contents of the remote state, and outputs the raw format to standard output
    - We cannot use the command to query Terraform version of the remote state as the output will be formated to a format that's compatible with the local Terraform version before output
  - We can use the `terraform state push` command to write state files to the configured backend
    - This uploads the local state to teh remote backend
    - Usage `terraform state push [options] PATH`
