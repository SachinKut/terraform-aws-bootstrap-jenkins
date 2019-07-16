# A Terraform module to generate a Jenkins Master node

This module will create a [Jenkins](https://jenkins.io/) node in the provided VPC and subnet.
Resources created:

* EC2 instance with all required software prerequisites installed
* new Security Group for ssh  and web access
* IAM profile for an instance
* DNS entry in Route53 for Jenkins master
* [not yet] cronjob for sending custom CloudWatch metric reporting cluster health
* [not yet] [optional] Custom AMI for EC2 instance, it gives possibility to restore state of management node

# Notes

Terraform version  `>= 0.12`

## Preparations

The module requires that the AWS policy documents for permissions be created prior to executing.
Please use `github.com/kentrikos/aws-bootstrap` repo to create policies.
Please follow the steps outlined in the README deployment guide.

## Usage

### Basic use

```hcl
module "jenkins" {
  source              = "github.com/kentrikos/terraform-aws-bootstrap-jenkins"

  product_domain_name = "demo"
  environment_type    = "test"

  vpc_id              = "vpc-12345"
  subnet_id           = "subnet-12345"

  http_proxy          = "10.10.10.1"

  ssh_allowed_cidrs   = ["10.10.10.0/24"]
  http_allowed_cidrs  = ["10.10.10.0/24"]
  
  operations_aws_account_number  = "123456789012"
  application_aws_account_number = "210987654321"
}
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| `ami_id` | (Optional) The AMI ID, which provides restoration of pre-created managment node. (default is false). | n/a | n/a |  yes |
| `application_aws_account_number` | AWS application account number (without hyphens) | n/a | n/a |  yes |
| `auto_IAM_mode` | Create IAM Policies in AWS | n/a | n/a |  yes |
| `auto_IAM_path` | IAM path for auto IAM mode uploaded policies | n/a | `"/"` |  no |
| `ec2_instance_type` | Size of EC2 instance. | n/a | `"t3.medium"` |  no |
| `environment_type` | (Required) Type of environment (e.g. test, production) | n/a | n/a |  yes |
| `http_allowed_cidrs` | (Optional) list of cidr ranges to allow HTTP access. | list(string) | n/a |  yes |
| `http_proxy` | (Optional) HTTP proxy to use for access to internet. This is required to install packages on instances deployed in ops AWS accounts. | n/a | n/a |  yes |
| `iam_policy_names` | (Optional) List of IAM policy names to apply to the instance. | list(string) | `["KENTRIKOS_autoscaling_elb_eks","KENTRIKOS_lma","KENTRIKOS_dynamodb","KENTRIKOS_ec2","KENTRIKOS_ecr_route53","KENTRIKOS_iam","KENTRIKOS_s3","KENTRIKOS_ssm","KENTRIKOS_vpc"]` |  no |
| `iam_policy_names_prefix` | (Optional) Prefix for policy names created by portal. | n/a | `"/"` |  no |
| `jenkins_additional_jcasc` | Path to directory containing aditional Jenkins configuration as code files; empty string is for disable | n/a | n/a |  yes |
| `jenkins_admin_password` | Local jenkins Admin username. | n/a | `"Password"` |  no |
| `jenkins_admin_username` | Local jenkins Admin username. | n/a | `"Admin"` |  no |
| `jenkins_config_repo_url` | Git repo url with Product Domain configuration | n/a | n/a |  yes |
| `jenkins_dns_domain_hosted_zone_ID` | R53 Hosted Zone ID for domain that will be used by Jenkins master | n/a | n/a |  yes |
| `jenkins_dns_hostname` | Local part of FQDN for Jenkins master | n/a | `"jenkins"` |  no |
| `jenkins_job_repo_url` | (Optional) Git repo url with Jenkins Jobs | n/a | `"https://github.com/kentrikos/jenkins-bootstrap-pipelines.git"` |  no |
| `jenkins_proxy_http_port` | (Optional) HTTP proxy port to use for access to internet. This is required to install packages on instances deployed in ops AWS accounts. | n/a | `"8080"` |  no |
| `key_name_prefix` | (Optional) The key name of the Key Pair to use for remote management. | n/a | `"jenkins_master"` |  no |
| `name_suffix` | (Optional) Instance name suffix. | n/a | `"jenkins-master-node"` |  no |
| `operations_aws_account_number` | AWS operations account number (without hyphens) | n/a | n/a |  yes |
| `product_domain_name` | (Required) Name of product domain, will be used to create other names | n/a | n/a |  yes |
| `region` | AWS region | n/a | `"eu-central-1"` |  no |
| `ssh_allowed_cidrs` | (Optional) list of cidr ranges to allow SSH access. | list(string) | n/a |  yes |
| `subnet_id` | (Required) The VPC Subnet ID to launch the instance in. | n/a | n/a |  yes |
| `tags` | (Optional) A mapping of tags to assign to the resource. A 'Name' tag will be created by default using the input from the 'name' variable. | map(string) | n/a |  yes |
| `vpc_id` | (Required) The VPC ID to launch the instance in. | n/a | n/a |  yes |

## Outputs

| Name | Description |
|------|-------------|
| `jenkins_dns_name` | FQDN associated with Jenkins master |
| `jenkins_private_ip` | Private IP address assigned to the instance |
| `jenkins_username` | Linux username for the instance. |
| `jenkins_web_login` | Default username for web dashboard |
| `jenkins_web_password` | Default password for web dashboard |
| `jenkins_web_url` | URL for Jenkins web dashboard |
| `ssh_connection` | SSH connection string for remote management. |
| `ssh_private_key` | SSH private key. |

