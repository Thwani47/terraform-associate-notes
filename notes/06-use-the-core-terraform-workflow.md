# Use the core Terraform workflow

## The Core Terraform Workflow

- The core Terraform workflow has 3 steps:
  - **Write** - Author infrastructure as code
  - **Plan**  - Preview changes before applying them
  - **Apply** - Provision reproducible infrastructure
- We **write** Terraform configuration just like how we write code, in an editor of our choice.
- It's common practice to store Terraform configuration files in a version-controlled repository
- As we make changes to our infrastructure configuration files, it is good practice to repeatedly run **plans** to flush out any syntax errors and ensure our config works (or will work) as intended
- When we're happy with our configuration, we can then **apply** the configuration and instruct Terraform to provision the resources defined in our configuration on our cloud provider
- HCP Terraform provides a centralized and secure location to store input variables and state

## Terraform init command

- The `terraform init` command initializes a working directory containing Terraform configuration files
- This command is safe to run multiple times.
- This command will never delete the state or update the configuration
- We can run the command with the `-from-module=MODULE-SOURCE` to initialize the repo with a module copied from another source like a version control source. We can also think of this as a shorthand to cloning the repo, and running `terraform init` in the repo
- The `-migrate-state` option can be used to copy existing state to a new backend
- Re-running `terraform init` will install sources for any modules that were added since the last `terraform init` run. Already installed modules will not be updated. We can use the `-upgrade` to install new/latest versions of already installed modules if there are any
- During `terraform init`, Terraform will also attempt to install plugins for the listed providers, both direct and indirect providers
- The `-upgrade` option will also instruct Terraform to ignore the state lock file and consider installing newer versions of the providers

## Terraform get command

- The `terraform get` command downloads and updates the modules declared in the root module
- The modules are downloaded into a `.terraform` subdirectory in the current working directory.
- The `.terraform` subdirectory is not to be commited to the version control respository
  
## Terraform validate command

- The `terraform validate` command validates the configuration files in a directory
- It does not validate remote services, such as remote state or provider APIs
- Running this command requires that the directory be already initialized
- `terraform plan` includes an implied validation check
- `terraform validate -json` produces the output in a human-readable JSON format

## Terraform plan command

- The `terraform plan` command creates an execution plan, which enables us to preview the changes that Terraform plans to make to our infrastructure
- Running `terraform plan -destroy` produces a plan whose goal is to destroy all remote objects, resulting in an empty state. This is the same as running `terraform destroy`
- Running `terraform plan -refresh-only` produces a plan whose goal is to update the Terraform state and any root modules with changes that were made outside of Terraform. i.e, if remote resources were manually updates, running `terraform plan -refresh-only` will produce a plan to describe how the state will be updated to reflect those changes

## Terraform apply command

- The `terraform apply` command executes the proposed actions proposed in a Terraform plan
- The `-auto-approve` options instructs Terraform to apply the changes without prompting for confirmation
- We can also provide a saved plan file to the `terraform apply` command. This may be useful when running Terraform in automation
- We can use `terraform show` to inspect a plan file before applying it

## Terraform destroy command

- The `terraform destroy` command deprovisions all objects managed by a Terraform configuration
- `terraform destroy` is the same as `terraform apply -destroy`
- We can use the `-target` option to destroy a specific resource and its dependencies

## Terraform fmt command

- The `terraform fmt` command formats Terraform configuration file contents so that it matches the canonical format and style
- This command helps ensure consistency and readability
- By default, `terraform fmt` scans and formats the current directory. We can also specify a `target` command to tell Terraofrm to scan
  - a directory
  - a specific file
  - standard input
- We can also specify the `-recursive` option to instruct Terraform to scan the current directory and all its child directories.
