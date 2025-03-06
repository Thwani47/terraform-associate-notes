# Understand the purpose of Terraform vs other IaC

## Terraform Use Cases

- Terraform enables us to define infrastructure resources using human-readable configuration files.
- We can share these files, we can version them, we can reuse them.
- Terraform also enables us to use a consistent workflow to safely and efficiently manage our infrastructure throughout its lifecycle
- We can use Terraform to deploy our infrastructure across multiple cloud
  - This increases fault-tolerance, and allows for more graceful recovery from cloud provider outages
  - We can deploy infrastructure resoruces in AWS, Azure, GCP, etc,
  - Terraform enables us to use the same workflow to interact with multiple (and different) cloud providers
- We can use Terraform to efficiently deploy, release, scale, and monitor infrastructur for multi-tier applications
- We can integrate Terraform with ticketing solutions like ServiceNow to automatically generate new infrastructure requests
- We can use Terraform to deploy and manage applications running on Kubernetes.
  - We can also manage K8s resources such as Pods, deployments, services, etc
- We can use Terraform to manage multiple parallel environments such as dev, test, and prod environments
  
## Purpose of Terraform State

- Terraform requires state to map real-world resources to the configuration
- Terraform also uses the state to track metadata such as resource dependencies
- Terraform also uses the state for peformance boosting
