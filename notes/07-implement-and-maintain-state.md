# Implement and Maintain State

## Backends

- Terraform allows us to use HCP Terraform to store data or define a `backend` block to store state in a remote object
- We do not need to define a `backend` block when we store the state in HCP Terraform or Terraform Enterprise
- We cannot have both the `cloud` block and `backend` block in the same configuration
- We add the `backend` block within a nested `terraform` block

```tf
terraform {
    backend "remote" {
        organization = "my-org"

        workspaces = {
            name = "prod"
        }
    }
}
```

- A configuration can only have ONE `backend` block
- A backend block cannot refer to input variables, locals, or data source attributes
- By default, Terraform uses a `local` backend
- Terraform comes with some built-in backends
- We cannot load additional backends
- If we make any changes to a backend's configuration, we need to run `terraform init` to validate and (re)configure the backend
- The backend configuration is stored in the `.terraform` folder. We should not check this folder to version control
- We can use the `-backend-config` option to specify a file containing the backend configuration values during init
- We can also use the `-backend-config="KEY-VALUE"` option to specify the backend configuration values via the command line
- We can also use backend configuration files that follow the naming pattern `*.backendname.tfbackend`. For example, `config.azurerm.backend`

## Local Backend

- The local backend stores state on the local filesystem.
- This is the default backend

```tf
terraform {
    backend "local" {
        path = "relative/path/to/terraform.tfstate"
    }
}
```

## Authenticating

- We can integrate the Terraform CLI with HCP Terraform and Terraform Enterprise and
  - Use the CLI as a front-end developer for CLI-driven runs in HCP Terraform
  - Use HCP Terraform or Terraform Enterprise as a state backend and private module registry
- Running the `terraform login` command generates an API token we can use to authenticate to HCP Terraform and Terraform Enterprise
- The `terraform logout` command terminates our current session
- `terraform login` opens a new browser window
  - We can provide crendetials via the CLI if we're running Terraform in an automated environment
- We need to specify the hostname when running the login command, `terraform login [hostname]`.
  - If we do not specify the hostname, it defaults to `app.terraform.io`
- The API key will be stored in plain text in a file called `credentials.tfrc.json`
- The `terraform login` command works with any login server that supports the login protocol from HCP
- The syntax for the logout command is similar to the login command, `terraform logout [hostname]`
- The logout command **only removes the API token from the local storage. It is not destroyed on the remote server. It will remain active until manually deleted**

## Sensitive Data in State

- When using local state, state is stored in plain-text JSON files

## Run modes and options

- The default run mode is to plan and then apply the plan
- If `auto-apply` is enabled, then a successful plan is applied immediately. If it is not enabled, the user will be prompted to confirm before applying
- The `destroy` mode instructs Terraform to create a plan which destroys all objects
  - This can be created on the CLI via the `terraform plan -destroy` or `terraform destroy`
  - Or via the API using the `is-destroy` option
- We can use `terraform plan out <file>` to save a plan and then use `terraform plan <file>` to apply a saved plan
- The `refresh-only` mode instructs Terraform to create a plan that updates the Terraform state to match changes made to remote objects outside of Terraform
  - `terraform plan -refresh-only` or `terraform apply -refresh-only`
