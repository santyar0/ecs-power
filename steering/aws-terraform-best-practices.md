# AWS Terraform Best Practices

This guide incorporates AWS prescriptive guidance, Kiro best practices, and industry standards for Terraform infrastructure management.

## Core Principles

### 1. Infrastructure as Code (IaC) Fundamentals
- **Declarative Configuration**: Define desired state, not procedural steps
- **Version Control**: All infrastructure code must be in version control
- **Immutable Infrastructure**: Replace rather than modify resources
- **Idempotency**: Operations should be safely repeatable

### 2. Security-First Approach
- **Least Privilege**: Grant minimal necessary permissions
- **Defense in Depth**: Multiple layers of security controls
- **Encryption Everywhere**: At rest, in transit, and in processing
- **Audit Trail**: Comprehensive logging and monitoring

## AWS CLI Best Practices

### Configuration Standards
```bash
# Always use --no-cli-pager for automation
aws configure set cli_pager ""

# Use specific regions to avoid confusion
aws configure set region us-east-1

# Enable CLI auto-prompt for interactive use
aws configure set cli_auto_prompt on-partial
```

### Authentication Best Practices
- **Use IAM roles** instead of access keys when possible
- **Rotate credentials** regularly (90 days maximum)
- **Use AWS SSO** for human access
- **Implement MFA** for all privileged operations

### CLI Integration Patterns
```bash
# Validate credentials before Terraform operations
aws sts get-caller-identity

# Check service quotas before deployment
aws service-quotas list-service-quotas --service-code ec2

# Verify resource limits
aws ec2 describe-account-attributes --attribute-names max-instances
```

## Terraform Project Structure

### Directory Organization
```
project/
├── .terraform/                 # Terraform working directory (gitignored)
├── .terraform.lock.hcl        # Provider version lock file
├── environments/              # Environment-specific configurations
│   ├── dev/
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   └── prod/
├── modules/                   # Reusable modules
│   └── {service}/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── versions.tf
├── main.tf                    # Root module configuration
├── variables.tf               # Input variables
├── outputs.tf                 # Output values
├── locals.tf                  # Local values and computations
├── data.tf                    # Data sources
├── providers.tf               # Provider configurations
└── versions.tf                # Terraform and provider version constraints
```

### File Naming Conventions
- **Service files**: `{service}.tf` (e.g., `ecs.tf`, `rds.tf`)
- **Environment files**: `{env}.tfvars` (e.g., `prod.tfvars`)
- **Backend files**: `{env}.tfbackend` (e.g., `prod.tfbackend`)
- **Module directories**: `{service}-{purpose}` (e.g., `vpc-networking`)

## Version Management

### Terraform Version Constraints
```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### Provider Version Pinning
- **Use version constraints** (`~> 5.0`) for flexibility with patch updates
- **Pin exact versions** in production for stability
- **Test upgrades** in development environments first
- **Document breaking changes** in upgrade notes

## State Management Best Practices

### Remote Backend Configuration
```hcl
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "project/environment/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
    
    # Enable versioning for state recovery
    versioning = true
  }
}
```

### State Security
- **Enable encryption** for state files
- **Use versioning** for state recovery
- **Implement locking** to prevent concurrent modifications
- **Restrict access** using IAM policies
- **Regular backups** with automated testing

### State Management Commands
```bash
# Import existing resources
terraform import aws_instance.example i-1234567890abcdef0

# Remove resources from state without destroying
terraform state rm aws_instance.example

# Move resources in state
terraform state mv aws_instance.old aws_instance.new

# Show current state
terraform state list
terraform state show aws_instance.example
```

## Security Best Practices

### IAM Policy Design
```hcl
# Principle of least privilege
resource "aws_iam_policy" "service_policy" {
  name = local.service_policy_name

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "SpecificServiceAccess"
        Effect = "Allow"
        Action = [
          "service:SpecificAction1",
          "service:SpecificAction2"
        ]
        Resource = [
          "arn:aws:service:${var.region}:${var.account_id}:resource/specific-resource"
        ]
        Condition = {
          StringEquals = {
            "aws:RequestedRegion" = var.region
          }
        }
      }
    ]
  })
}
```

### Encryption Standards
```hcl
# S3 bucket encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "example" {
  bucket = aws_s3_bucket.example.id

  rule {
    apply_server_side_encryption_by_default {
      kms_master_key_id = aws_kms_key.example.arn
      sse_algorithm     = "aws:kms"
    }
    bucket_key_enabled = true
  }
}

# RDS encryption
resource "aws_db_instance" "example" {
  storage_encrypted = true
  kms_key_id       = aws_kms_key.example.arn
  # ... other configuration
}
```

### Network Security
```hcl
# Security group with minimal access
resource "aws_security_group" "app" {
  name_prefix = "${local.app_name}-"
  vpc_id      = var.vpc_id

  # Specific ingress rules only
  ingress {
    description = "HTTPS from ALB"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  # Minimal egress
  egress {
    description = "HTTPS outbound"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${local.app_name}-app-sg"
  }
}
```

## Code Quality Standards

### Variable Validation
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"

  validation {
    condition = contains([
      "t3.micro", "t3.small", "t3.medium",
      "t3.large", "t3.xlarge", "t3.2xlarge"
    ], var.instance_type)
    error_message = "Instance type must be a valid t3 instance type."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string

  validation {
    condition     = can(regex("^(dev|staging|prod)$", var.environment))
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

### Resource Naming Standards
```hcl
locals {
  # Consistent naming pattern
  name_prefix = "${var.project}-${var.environment}"
  
  # Resource-specific names
  vpc_name     = "${local.name_prefix}-vpc"
  subnet_names = {
    for az, subnet in var.subnets : az => "${local.name_prefix}-subnet-${az}"
  }
  
  # Common tags
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
    Owner       = var.owner
    CostCenter  = var.cost_center
  }
}
```

### Documentation Standards
```hcl
/**
 * ECS Cluster Module
 * 
 * Creates an ECS cluster with the following features:
 * - Container Insights enabled
 * - Capacity providers configured
 * - CloudWatch logging
 * 
 * @param cluster_name - Name of the ECS cluster
 * @param enable_container_insights - Enable CloudWatch Container Insights
 * @param capacity_providers - List of capacity providers to use
 */
resource "aws_ecs_cluster" "main" {
  name = var.cluster_name

  setting {
    name  = "containerInsights"
    value = var.enable_container_insights ? "enabled" : "disabled"
  }

  capacity_providers = var.capacity_providers

  tags = merge(local.common_tags, {
    Name = var.cluster_name
  })
}
```

## Testing and Validation

### Pre-deployment Validation
```bash
# Format check
terraform fmt -check -recursive

# Validation check
terraform validate

# Security scanning
tfsec .

# Cost estimation
infracost breakdown --path .

# Plan review
terraform plan -out=tfplan
terraform show -json tfplan | jq
```

### Automated Testing
```hcl
# Use terraform-compliance for policy testing
# Example: security.feature
Feature: Security compliance
  Scenario: Ensure all S3 buckets are encrypted
    Given I have aws_s3_bucket defined
    Then it must have server_side_encryption_configuration
```

### Integration Testing
```bash
# Test infrastructure after deployment
aws ecs describe-clusters --clusters ${cluster_name}
aws rds describe-db-instances --db-instance-identifier ${db_identifier}
curl -f https://${alb_dns_name}/health
```

## Performance Optimization

### Resource Right-sizing
```hcl
# Use data sources for dynamic sizing
data "aws_ec2_instance_types" "available" {
  filter {
    name   = "instance-type"
    values = ["t3.*"]
  }
}

locals {
  # Choose instance type based on environment
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
}
```

### Parallel Execution
```bash
# Increase parallelism for faster deployments
terraform apply -parallelism=20

# Target specific resources for faster updates
terraform apply -target=aws_ecs_service.app
```

### State Optimization
```bash
# Refresh state before operations
terraform refresh

# Clean up unused state
terraform state list | grep -v "aws_" | xargs -I {} terraform state rm {}
```

## Monitoring and Observability

### CloudWatch Integration
```hcl
# Application-specific alarms
resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "${local.name_prefix}-high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ECS"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "This metric monitors ecs cpu utilization"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
    ServiceName = aws_ecs_service.app.name
  }

  tags = local.common_tags
}
```

### Logging Standards
```hcl
# Centralized logging
resource "aws_cloudwatch_log_group" "app" {
  name              = "/aws/ecs/${local.name_prefix}"
  retention_in_days = var.log_retention_days
  kms_key_id        = aws_kms_key.logs.arn

  tags = local.common_tags
}
```

## Cost Optimization

### Resource Tagging for Cost Allocation
```hcl
locals {
  cost_tags = {
    CostCenter  = var.cost_center
    Project     = var.project
    Environment = var.environment
    Owner       = var.owner
    Purpose     = var.purpose
  }
}

# Apply to all resources
resource "aws_instance" "example" {
  # ... configuration
  
  tags = merge(local.common_tags, local.cost_tags, {
    Name = "${local.name_prefix}-instance"
  })
}
```

### Lifecycle Management
```hcl
# S3 lifecycle policies
resource "aws_s3_bucket_lifecycle_configuration" "example" {
  bucket = aws_s3_bucket.example.id

  rule {
    id     = "transition_to_ia"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    transition {
      days          = 90
      storage_class = "GLACIER"
    }

    expiration {
      days = 365
    }
  }
}
```

## Disaster Recovery

### Multi-Region Setup
```hcl
# Primary region resources
provider "aws" {
  alias  = "primary"
  region = var.primary_region
}

# DR region resources
provider "aws" {
  alias  = "dr"
  region = var.dr_region
}

# Cross-region replication
resource "aws_s3_bucket_replication_configuration" "replication" {
  role   = aws_iam_role.replication.arn
  bucket = aws_s3_bucket.source.id

  rule {
    id     = "replicate_to_dr"
    status = "Enabled"

    destination {
      bucket        = aws_s3_bucket.destination.arn
      storage_class = "STANDARD_IA"
    }
  }
}
```

### Backup Strategies
```hcl
# Automated RDS backups
resource "aws_db_instance" "example" {
  backup_retention_period = var.environment == "prod" ? 30 : 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  # Point-in-time recovery
  enabled_cloudwatch_logs_exports = ["postgresql"]
  
  tags = local.common_tags
}
```

## Compliance and Governance

### Policy as Code
```hcl
# AWS Config rules
resource "aws_config_configuration_recorder" "recorder" {
  name     = "${local.name_prefix}-recorder"
  role_arn = aws_iam_role.config.arn

  recording_group {
    all_supported                 = true
    include_global_resource_types = true
  }
}

# Compliance rules
resource "aws_config_config_rule" "s3_bucket_public_access_prohibited" {
  name = "s3-bucket-public-access-prohibited"

  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_PUBLIC_ACCESS_PROHIBITED"
  }

  depends_on = [aws_config_configuration_recorder.recorder]
}
```

### Audit Trail
```hcl
# CloudTrail for API auditing
resource "aws_cloudtrail" "main" {
  name           = "${local.name_prefix}-trail"
  s3_bucket_name = aws_s3_bucket.cloudtrail.bucket

  event_selector {
    read_write_type                 = "All"
    include_management_events       = true
    exclude_management_event_sources = []

    data_resource {
      type   = "AWS::S3::Object"
      values = ["${aws_s3_bucket.sensitive.arn}/*"]
    }
  }

  tags = local.common_tags
}
```

## Development Workflow

### Git Integration
```bash
# Pre-commit hooks
#!/bin/bash
# .git/hooks/pre-commit
terraform fmt -check -recursive
terraform validate
tfsec --soft-fail
```

### CI/CD Pipeline
```yaml
# .github/workflows/terraform.yml
name: Terraform
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.0
          
      - name: Terraform Format
        run: terraform fmt -check -recursive
        
      - name: Terraform Validate
        run: terraform validate
        
      - name: Terraform Plan
        run: terraform plan -no-color
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## Common Pitfalls and Solutions

### 1. State File Corruption
**Problem**: Concurrent modifications corrupt state
**Solution**: Use state locking with DynamoDB

### 2. Resource Drift
**Problem**: Manual changes outside Terraform
**Solution**: Regular `terraform plan` checks and import commands

### 3. Hardcoded Values
**Problem**: Environment-specific values in code
**Solution**: Use variables and data sources

### 4. Missing Dependencies
**Problem**: Resources created in wrong order
**Solution**: Explicit `depends_on` declarations

### 5. Overprivileged IAM
**Problem**: Broad permissions for convenience
**Solution**: Principle of least privilege with specific actions

## Emergency Procedures

### State Recovery
```bash
# Restore from S3 backup
aws s3 cp s3://terraform-state-bucket/backup/terraform.tfstate ./terraform.tfstate

# Force unlock stuck state
terraform force-unlock LOCK_ID

# Import existing resources
terraform import aws_instance.example i-1234567890abcdef0
```

### Rollback Procedures
```bash
# Rollback to previous state
terraform apply -target=aws_instance.example -var="instance_type=t3.micro"

# Destroy specific resources
terraform destroy -target=aws_instance.problematic

# Plan destruction
terraform plan -destroy
```

This comprehensive guide ensures consistent, secure, and maintainable Terraform infrastructure following AWS and industry best practices.