# Understand Infrastructure as Code (IaC)

## Infrastructure as Code using Terraform

- Infrastructure as Code (IaC) enables us to use a declarative code to create and manage infrastructure.
- IaC allows us to manage our infrastructure using configuration files instead of using command-line tools or a graphical user interface.
- IaC enables us to deploy and manage our infrastructure in a repeatable, consistent, and repeatable way. The same code applied multiple times will always yield the same results.
- We can version these configuration files, which enables us to roll back to previous versions.
- We can use version control tools such as Github, Azure DevOps to manage and version our IaC configuration files.
- We can share and reuse these configuration files, which enables us to deploy different instances of our infrastructure in different environments. For example, we can deploy to a dev, test, and production environment using the same configuration files.
- Terraform is an IaC tool created by Hashicorp.
- We can use Terraform to define infrastructure resources in a declarative human-readable format.
- Terraform can be used to manage infrastructure resources on multiple cloud platforms (Azure, AWS, GCP, etc)
- Terraform uses plugins called **providers** to interact with cloud platforms and other services via their APIs. We can view the publicly available providers on the [Terraform Registry](https://registry.terraform.io/browse/providers?product_intent=terraform)
- Terraform enables us to write our own plugins
- Terraform uses a declarative language, meaning that in the Terraform configuration files we define the desired end-state of our infrastructure, and Terraform will generate the necessary steps it needs to take to get us to our desired end-state, taking into consideration the current state of the infrastructure and any dependencies between our infrastructure resources
- To be able to generate the necessary steps to get our cloud infrastructure to match our desired end-state, Terraform tracks our cloud infrastructure in a state file, and if we make changes to our configuration files, Terraform compares those changes to the state files and decides if needs to add new resources, delete resources, or make changes to any resources, or a combination of these.
- By default, the Terraform state file is stored locally. But it is good practice to store the state file in a remote backend such as GitHub, GitLab, HCP Terraform, etc, to enable collaboration between different users.
