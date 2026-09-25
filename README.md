# Terraform AWS Infrastructure with GitLab CI/CD

## Overview

This project demonstrates how to provision AWS infrastructure using **Terraform** and automate the Terraform workflow through a **GitLab CI/CD pipeline**.

The project uses **GitLab OIDC (OpenID Connect)** to authenticate securely to AWS without storing permanent AWS access keys in the repository.

Terraform state is stored remotely using **GitLab-managed Terraform state**, allowing the CI/CD pipeline to maintain infrastructure state between jobs.

> This project is intended for DevOps / Cloud Engineering learning and portfolio demonstration.

---

## Technologies Used

- Terraform
- Amazon Web Services (AWS)
- Amazon EC2
- AWS IAM
- AWS Systems Manager Parameter Store
- GitLab CI/CD
- GitLab OIDC
- GitLab-managed Terraform State
- Git
- GitHub

---

## Architecture

```text
Developer
    |
    | git push
    v
GitLab Repository
    |
    v
GitLab CI/CD Pipeline
    |
    +---- Test AWS Connection
    |
    +---- Terraform Validate
    |
    +---- Terraform Plan
    |
    +---- Terraform Apply (Manual)
              |
              | OIDC Authentication
              v
         AWS IAM Role
              |
              v
          Terraform
              |
              v
            AWS EC2
```

Terraform state is stored remotely:

```text
Terraform
    |
    v
GitLab Managed Terraform State
```

---

## CI/CD Pipeline

The GitLab pipeline contains four stages:

```text
test
  |
  v
validate
  |
  v
plan
  |
  v
apply
```

### 1. Test AWS Connection

The pipeline obtains a GitLab OIDC token and uses AWS STS to assume an IAM role.

The connection is verified with:

```bash
aws sts get-caller-identity
```

No permanent AWS access key or secret access key is stored in the repository.

### 2. Terraform Validate

Terraform initializes the GitLab HTTP backend and validates the Terraform configuration.

```bash
terraform init
terraform validate
```

### 3. Terraform Plan

Terraform authenticates to AWS and generates an execution plan.

```bash
terraform plan
```

This allows infrastructure changes to be reviewed before deployment.

### 4. Terraform Apply

The apply stage is configured as a **manual job**.

```bash
terraform plan -out=tfplan-apply
terraform apply -auto-approve tfplan-apply
```

Infrastructure is therefore not automatically created simply because code is pushed to the repository.

---

## Secure AWS Authentication with OIDC

Instead of storing long-lived AWS credentials in GitLab, this project uses:

```text
GitLab CI/CD
     |
     | OIDC Token
     v
AWS Security Token Service (STS)
     |
     | AssumeRoleWithWebIdentity
     v
AWS IAM Role
     |
     v
Temporary AWS Credentials
```

The temporary credentials are then used by Terraform during the pipeline.

This approach reduces the need for long-lived cloud credentials in CI/CD.

---

## Terraform Remote State

Terraform uses GitLab-managed Terraform state through the HTTP backend.

```hcl
terraform {
  backend "http" {
  }
}
```

The backend configuration is supplied by the GitLab CI/CD pipeline during `terraform init`.

This allows Terraform state to persist between pipeline jobs without committing state files to Git.

---

## EC2 Infrastructure

The project retrieves the latest Amazon Linux 2023 AMI using AWS Systems Manager Parameter Store:

```hcl
data "aws_ssm_parameter" "amazon_linux_2023" {
  name = "/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64"
}
```

The EC2 instance configuration is:

```hcl
resource "aws_instance" "my_ec2" {
  ami           = data.aws_ssm_parameter.amazon_linux_2023.value
  instance_type = "t3.micro"

  tags = {
    Name = "terraform-ec2"
  }
}
```

AWS Region:

```text
us-east-1
```

---

## Repository Structure

```text
terraform-aws-project/
|
|-- .gitlab-ci.yml
|-- ec2.tf
|-- provider.tf
|-- README.md
```

### Files

**`.gitlab-ci.yml`**

Defines the GitLab CI/CD pipeline, AWS OIDC authentication, Terraform initialization, validation, planning, and manual apply workflow.

**`provider.tf`**

Defines the AWS Terraform provider and HTTP remote-state backend.

**`ec2.tf`**

Defines the Amazon Linux AMI lookup and EC2 infrastructure.

**`README.md`**

Contains project documentation.

---

## Security Practices

This project demonstrates several infrastructure security practices:

- No AWS access keys committed to Git
- OIDC-based AWS authentication
- Temporary AWS credentials through AWS STS
- IAM role-based access
- Remote Terraform state
- Manual infrastructure deployment
- Infrastructure defined as code
- Separation between planning and deployment

---

## Local Terraform Commands

Initialize Terraform:

```bash
terraform init
```

Validate the configuration:

```bash
terraform validate
```

Review infrastructure changes:

```bash
terraform plan
```

Deploy infrastructure:

```bash
terraform apply
```

Destroy Terraform-managed infrastructure when it is no longer required:

```bash
terraform destroy
```

> Running `terraform apply` can create real AWS resources and may consume AWS credits or incur charges.

---

## Git Workflow

The project is maintained in GitLab and mirrored to GitHub for portfolio visibility.

Push changes to GitLab:

```bash
git push origin main
```

Push the same changes to GitHub:

```bash
git push github main
```

---

## Project Status

The Terraform configuration, GitLab CI/CD pipeline, OIDC authentication, and remote state integration have been configured and tested.

The pipeline can successfully:

- Authenticate GitLab to AWS using OIDC
- Initialize the GitLab Terraform remote backend
- Validate Terraform configuration
- Generate an AWS infrastructure plan

The AWS EC2 deployment currently depends on the AWS account being authorized to launch EC2 resources.

---

## Future Improvements

Planned improvements include:

- Add a manual Terraform destroy pipeline stage
- Add networking resources such as VPC and security groups
- Add Terraform variables and outputs
- Modularize Terraform configuration
- Add additional security scanning
- Add infrastructure monitoring
- Expand the project to support multiple environments

---

## Author

**Komlavi Gidi**

DevOps / Cloud / Platform Engineering

GitHub: `gidikoml`