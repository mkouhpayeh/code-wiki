# Cloud Configuration

## Terraform 
- Terraform is a tool for building, changing, and versioning infrastructure safetly and efficiently. Compatible with many clouds and services.
- AWS User groups
  - AmazonRDSFullAccess
  - AmazonEC2FullAccess
  - IAMFullAccess
  - AmazonS3FullAccess
  - AmazonDynamoDBFullAccess
  - AmazonRoute53FullAccess
- AWS Users
  - Create access key ID > Security Credentials
  ```bash
  Install AWS CLI
  aws configure
  
  terraform init
  terraform plan -- Looking at the resulting, can catch the bugs.
  terraform apply [-auto-approve]

  terraform workspace list
  terraform workspace new production
  terraform workspace select production
  terraform destroy
  ```
- Variable: Variables are values you pass into Terraform. Input parameter
```
variable "assets_bucket_name" {
  type        = string
  description = "S3 bucket name for frontend assets."

  validation {
    condition     = length(trimspace(var.assets_bucket_name)) > 0
    error_message = "assets_bucket_name must not be empty."
  }
}

variable "db_password" {
  type      = string
  sensitive = true
}
```
- Ouput: Outputs are values Terraform shows after apply, or values a module exposes to another module. return values
- Module: Modules are the containers for multiple resources that are used together. They are the main way to package and reuse resource configurations with Terraform.
  - Root Module: Default module containing all .tf files in main working directory
  - Child Module: A seprate external module referred to from a .tf file
- 
## Provisioners 
Perform action on local or remote machine
- Ansible
- Terraform + Chef
- Puppet

## Additional Tools
- AWS Consul: A terraform module for how to run Consul on AWS using Terraform and Packer
- Terragrunt: Provides utility to make certain terraform use cases easier. Minimize code repetition. Enables multi-account separation
- Cloud Nuke: Easy cleanup of cloud resources
- Makefiles: Prevent human error

## CI/CD
- [Github Actions](https://github.com/hashicorp/setup-terraform)
- [CircleCI](https://circleci.com/developer/orbs/orb/circleci/terraform)
- [GitLab](https://docs.gitlab.com/ee/user/infrastructure/)
- [Atlantis](https://runatlantis.io)

## Terraform Testing
- Built-in
```
terraform fmt [-check]
terraform validate
terraform plan
custom validation rules
```
- External
  - tflint
  - checkov, tfsec, terrascan, terraform-compliance, snyk
  - Terraform Sential (enterprise only)
