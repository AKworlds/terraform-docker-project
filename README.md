# AWS Cloud Migration Project

A production-style AWS cloud migration project built for a fictional company evaluating a move from an on-premises application to AWS before providing production data.

The project focuses on infrastructure, security, monitoring, containerization, and CI/CD. A simple Flask **Test Page** is used to validate the environment.

## Business Scenario

The company wanted to determine whether AWS could provide a more scalable, maintainable, and potentially cost-effective alternative to its on-premises environment.

The cloud environment needed to:

- Keep application workloads private
- Provide a public application endpoint
- Use repeatable infrastructure deployment
- Containerize the application
- Centralize monitoring and alerting
- Maintain audit logs
- Encrypt audit data
- Automate application deployments
- Avoid long-lived AWS credentials in GitHub

---

## Architecture

```text
Internet
   |
   v
Application Load Balancer
   |
   v
ECS Fargate
Private Subnets
   |
   v
Docker / Flask App


GitHub Push
   |
   v
GitHub Actions
   |
   v
OIDC Authentication
   |
   v
Amazon ECR
   |
   v
ECS Deployment
```

Monitoring and security are provided through:

- CloudWatch
- SNS
- CloudTrail
- KMS
- S3
- IAM

---

## AWS Architecture

The Terraform configuration deploys:

- VPC
- Two public subnets
- Two private subnets
- Internet Gateway
- NAT Gateway
- Route tables
- Security groups
- Application Load Balancer
- ECS Fargate
- ECR
- IAM roles
- CloudWatch
- SNS
- CloudTrail
- S3
- KMS
- GitHub OIDC

ECS tasks run in private subnets with:

```hcl
assign_public_ip = false
```

The ECS security group accepts HTTP traffic only from the ALB security group.

---

## Containerized Application

The application is a lightweight Flask service running with Gunicorn.

Endpoints:

```text
/
```

Displays:

```text
Test Page
```

Health endpoint:

```text
/health
```

The `/health` endpoint is used by the Application Load Balancer to determine whether the ECS task is healthy.

---

## CI/CD

GitHub Actions automatically deploys application changes.

```text
Git Push
   |
   v
GitHub Actions
   |
   v
AWS OIDC
   |
   v
Docker Build
   |
   v
Amazon ECR
   |
   v
New ECS Task Definition
   |
   v
ECS Service Deployment
```

The pipeline:

1. Authenticates to AWS using GitHub OIDC
2. Builds the Docker image
3. Tags the image with the Git commit SHA
4. Pushes the image to ECR
5. Registers a new ECS task-definition revision
6. Updates the ECS service
7. Waits for service stability

Using OIDC eliminates the need to store long-lived AWS access keys in GitHub.

---

## Security and Monitoring

Security controls include:

- Private ECS workloads
- ALB-to-ECS security group restrictions
- IAM roles
- GitHub OIDC federation
- CloudTrail API auditing
- KMS-encrypted audit logs
- S3 Block Public Access
- CloudWatch monitoring
- SNS alerting

CloudWatch alarms monitor:

- ECS CPU utilization
- ALB unhealthy targets

---

## Deployment Validation

The final deployment was validated with:

```text
Running: 1
Pending: 0
Rollout: COMPLETED
```

The ECS task used a Git commit-specific image:

```text
terraform-docker-app:beeab60c9448875458a7973861c0c05ee01498d7
```

The Application Load Balancer target was confirmed:

```text
healthy
```

The application successfully displayed:

```text
Test Page
```

---

## Technologies

**AWS**
- VPC
- ECS Fargate
- ECR
- ALB
- IAM
- CloudWatch
- SNS
- CloudTrail
- KMS
- S3

**DevOps**
- Terraform
- Docker
- GitHub Actions
- GitHub OIDC
- AWS CLI
- Git

**Application**
- Python
- Flask
- Gunicorn

---

## Repository Structure

```text
terraform-docker-project/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── app/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── terraform/
│   ├── alb.tf
│   ├── audit.tf
│   ├── ecr.tf
│   ├── ecs.tf
│   ├── github_oidc.tf
│   ├── kms.tf
│   ├── monitoring.tf
│   ├── network.tf
│   ├── outputs.tf
│   ├── providers.tf
│   └── variables.tf
└── .gitignore
```

Terraform state files, variable files, and local virtual environments are excluded from Git.

---

## What This Project Demonstrates

- AWS cloud architecture
- Infrastructure as Code
- Containerized application deployment
- Private/public network segmentation
- Load balancing
- IAM and OIDC federation
- CI/CD automation
- Centralized monitoring and alerting
- Audit logging and encryption
- AWS troubleshooting and validation

---

## Future Improvements

Possible next steps include:

- HTTPS with AWS Certificate Manager
- Route 53
- AWS WAF
- ECS Auto Scaling
- Production data migration
- Database integration
- Remote Terraform state
- Load testing
- Cost comparison against the existing on-premises environment
