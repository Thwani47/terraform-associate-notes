# Understand HCP Terraform Capabilities

## What is HCP Terraform

- HCP Terraform is an application that helps teams use Terraform to provision infrastructure
- HCP Terraform manages Terraform runs in a consistent and reliable environment
- HCP Terraform enable easy access to shared state and secret data.
- It enables access controls for approving changes to infrastructure, and a private registry for sharing Terraform modules
- HCP Terraform is available as a hosted service at [https://app.terraform.io](https://app.terraform.io)
- Terraform Enterprise is a self-hosted distribution of HCP Terraform
  - It offers organizations a private instance of HCP Terraform that inclused advanced features available in HCP Terraform

## HCP Terraform Plans and Features

- Many of HCP Terraform features are free for small teams, including
  - Remote state storage
  - Remote runs
  - VCS connections
  - Private module registry
  - SSO
  - Policy enforcement
- HCP Terraform manages plans and billing at the **organization** level
- An HCP Terraform user can belong to multiple organizations
- The set of features available to a user depend on which organization they are currently working under
- Free organizations are limited to 500 managed resources
- HCP Terraform runs Terraform on disposable VMs in its own cloud infrastructure
- HCP Terraform organizes infrastructure into projects and workspaces
- In HCP Terraform, remote state is tied to a workspace
- Each workspace can be linked to a VCS repository
  - We can optionally specify a branch or subdirectory
  - HCP Terraform watches for repo changes
- HCP Terraform can send notifications to other systems like Slack, and any other system that accepts webhooks
  - Notifications are configured on a workspace level
- HCP Terraform Free inlcudes one policy set with up to 5 policies
  - Policies can enable us to set VM sizes, and so much more
- Before making changes in the major cloud providers, HCP Terraform will display an estimate of its total cost
  - We can also use Sentinel to set up policies to alert us when changes go above certain cost estimates

## Teams Overview

- Teams are a group of users within an organization
- Team management is available in HCP Terraform Standard
- Teams can only have permissions on workspaces within their organization
- Each team can have an API token not associated with a specific user
- The owner's team cannot be empty. It must have atleast one user
  - In the Free tier, only 5 users can be in the owner's team

## HCP Terraform Workspaces

- A workspace is a group of infrastructure resources managed by Terraform
- A workspace contains everything Terraform needs to manage a given collection of infrastructure
- HCP Terraform workspaces and local directories serve the same purpose but store their data differently
  - With HCP Terraform, the configuration is stored in a VCS repo or uploaded via the API, as compared to being stored on disk with local Terraform
  - With HCP Terraform, variables are stored in a workspace
- Each workspaces backups its previous state files
- HCP Terraform also tracks the Terraform runs for a given workspace
- HCP Terraform workspaces are different from the Terraform CLI workspaces
  - HCP Terraform workspaces are required. We cannot manage resources in HCP Terraform without creating at least one workspace
  - Terraform CLI workspaces are associated with a working directory and are used to isolate multiple state files in the same working directory
- Projects enable us to organize workspaces into groups
- HCP Terraform workspace health assessments are avaiable in HCP Terraform Plus
  - Health assessments perform drift detection and determines whether the real-world infrastructure matches the configuration
  - Health assessments also perform continous validation to determine whether custom validation rules are still met