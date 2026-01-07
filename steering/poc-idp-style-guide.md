# poc-idp Repository Style Guide

This guide documents the exact coding style, patterns, and conventions used in the poc-idp repository. **All new code must follow these patterns exactly.**

> **Source Repository:** `https://github.com/santyar0/poc-idp.git`  
> **Verified Against:** Actual repository code in `poc-idp/` folder

---

## File Organization

### Required File Structure

```
├── main.tf                    # Random string generator and main setup
├── main.auto.tfvars           # Auto-loaded variable values
├── variables.tf               # ALL input variables with validation
├── locals.tf                  # Computed values and resource naming (740 lines)
├── outputs.tf                 # Output values (no sensitive data)
├── data.tf                    # Data source definitions
├── backend.tf                 # Terraform backend configuration
├── providers.tf               # Provider requirements and configuration
├── alarms.tf                  # CloudWatch alarms for all services
├── acm.tf                     # ACM certificate management
├── alb.tf                     # Application Load Balancer
├── aws_backup.tf              # AWS Backup configuration
├── bastion.tf                 # Bastion host resources
├── cloudfront.tf              # CloudFront distribution
├── cloudmap.tf                # AWS Cloud Map service discovery
├── docdb.tf                   # DocumentDB cluster
├── ecr.tf                     # ECR repositories
├── ecs.tf                     # ECS cluster and services
├── efs.tf                     # EFS file system
├── elasticache.tf             # ElastiCache Redis
├── rds.tf                     # RDS Aurora cluster
├── route53.tf                 # Route53 DNS records
├── s3.tf                      # S3 buckets
├── vpc.tf                     # VPC and networking
├── vpn.tf                     # Client VPN endpoint
├── files/                     # Static files and templates
│   ├── cloudfront/spa/        # CloudFront functions
│   └── frontend_bucket_example/
├── tfvars-sample/             # Template variable files
│   ├── example-small-setup.tfvars
│   ├── example-medium-setup.tfvars
│   ├── example-large-setup.tfvars
│   └── pipeline.tfvars
├── tfvars/                    # Generated configuration files
├── tfbackends/                # Backend configuration files
└── modules/                   # Reusable modules
    ├── vpc/
    ├── db/
    ├── alb/
    ├── efs/
    ├── elasticache/
    └── vpn_sg/
```

### File Naming Rules

| File Type | Pattern | Examples |
|-----------|---------|----------|
| Service files | `{service}.tf` | `ecs.tf`, `rds.tf`, `vpc.tf`, `alb.tf`, `docdb.tf` |
| Template configs | `example-{size}-setup.tfvars` | `example-small-setup.tfvars`, `example-medium-setup.tfvars` |
| Generated configs | `{app}-{account}-{region}.tfvars` | `myapp-123456789012-us-east-1.tfvars` |
| Backend configs | `{app}-{account}-{region}.tfbackend` | `myapp-123456789012-us-east-1.tfbackend` |
| Auto-loaded vars | `*.auto.tfvars` | `main.auto.tfvars` |

---

## Naming Conventions

### Resource Naming Pattern

**ALL resource names MUST be defined in `locals.tf` using this exact pattern:**

```hcl
# Pattern: {service}_{descriptor}_{suffix}
resource_name = "${substr(var.application, 0, X)}-${substr(var.environment, 0, Y)}-{descriptor}-${random_string.setup.id}"
```

### Character Limits by Service

**CRITICAL: These limits are AWS service-specific and MUST be followed:**

| Service Type | Max Length | App Substr | Env Substr | Pattern Example |
|--------------|------------|------------|------------|-----------------|
| **Short names** (21 chars) | 21 | 21 | 4 | VPC, ALB, ECS cluster, CloudFront |
| **Medium names** (38 chars) | 38 | 38 | 5 | S3, RDS, ElastiCache, EFS, DocumentDB, Bastion, VPN |
| **Subnet groups** (9 chars) | varies | 9 | 5 | DocumentDB subnet group |
| **Policy names** (72 chars) | 72 | 21-38 | 4-5 | IAM policies with descriptive names |

### Verified Examples from poc-idp

```hcl
# locals.tf - EXACT patterns from actual repository code

# VPC (21 char limit for name)
vpc_name = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-${random_string.setup.id}"

# ALB (21 char limit)
alb_name = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-ecsalb-${random_string.setup.id}"

# ECS Cluster (21 char limit)
ecs_name = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-${random_string.setup.id}"

# ECS Task Definition Family (21 char limit)
ecs_task_definition_names = { 
  for svc_name, _ in local.ecs_services : 
  svc_name => "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-${svc_name}-family-${random_string.setup.id}" 
}

# ECS Service Names (21 char limit)
ecs_services_names = { 
  for svc_name, _ in local.ecs_services : 
  svc_name => "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-${svc_name}-service-${random_string.setup.id}" 
}

# S3 Buckets (38 char limit)
s3_user_data_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-user-data-${random_string.setup.id}"
s3_frontend_name  = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-frontend-${random_string.setup.id}"

# RDS (38 char limit)
db_name                 = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-database-${random_string.setup.id}"
db_subnet_group_name    = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-subnetgroup-${random_string.setup.id}"
db_parameter_group_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-parametergroup-${random_string.setup.id}"
db_sg_name              = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-dbsecurity-${random_string.setup.id}"
db_secret_name          = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-dbsecret-${random_string.setup.id}"
db_monitoring_role_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-dbmonitoring-${random_string.setup.id}"

# ElastiCache (38 char limit for most, 21 for replication group ID)
elasticache_replication_group_id = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 5)}-cache-${random_string.setup.id}"
elasticache_secret_name          = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-cachesecret-${random_string.setup.id}"
elasticache_sg_name              = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-cachesecurity-${random_string.setup.id}"
elasticache_subnet_group_name    = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-cachesubnetgroup-${random_string.setup.id}"

# EFS (38 char limit)
efs_name                     = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-storage-${random_string.setup.id}"
efs_sg_name                  = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-efssecurity-${random_string.setup.id}"
efs_access_point_policy_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-efs-access-point-${random_string.setup.id}"

# DocumentDB (38 char limit, 9 for subnet group)
docdb_name                 = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-docdb-${random_string.setup.id}"
docdb_subnet_group_name    = "${substr(var.application, 0, 9)}-${substr(var.environment, 0, 5)}-docdbsubnetgroup-${random_string.setup.id}"
docdb_sg_name              = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-docdbsecurity-${random_string.setup.id}"
docdb_secret_name          = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-docdbsecret-${random_string.setup.id}"
docdb_parameter_group_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-docdb-cluster-parameter-group-docdbsecret-${random_string.setup.id}"

# CloudFront (21 char limit)
cloudfront_name         = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-cdn-${random_string.setup.id}"
cloudfront_oac_name     = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-cdn-oac-${random_string.setup.id}"
cloudfront_vpc_origin_name = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-cdn-vpc-origin-${random_string.setup.id}"

# AWS Backup (38 char limit)
backup_vault_name     = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-backup-vault-${random_string.setup.id}"
backup_plan_name      = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-backup-plan-${random_string.setup.id}"
backup_plan_rule_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-backup-plan-rule-${random_string.setup.id}"
backup_iam_role_name  = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-backup-iam-role-${random_string.setup.id}"

# ECR (38 char limit)
ecr_names = { 
  for svc_name in keys(local.ecs_filtered_need_repo) : 
  svc_name => "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-${svc_name}-ecr-${random_string.setup.id}" 
}

# Bastion (38 char limit)
bastion_name              = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-bastion-${random_string.setup.id}"
bastion_sg_name           = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-bastionsecurity-${random_string.setup.id}"
bastion_iam_resource_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-bastionsecurity-${random_string.setup.id}"
bastion_secret_name       = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-bastionsecret-${random_string.setup.id}"
bastion_ssh_key_name      = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-bastionsshkey-${random_string.setup.id}"

# VPN (38 char limit)
vpn_name        = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-vpn-${random_string.setup.id}"
vpn_sg_name     = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-vpnsecurity-${random_string.setup.id}"
vpn_secret_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-vpnsecret-${random_string.setup.id}"

# VPC Endpoints (38 char limit)
vpc_endpoint_name_prefix = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-${random_string.setup.id}"
vpc_endpoints_sg_name    = "${local.vpc_endpoint_name_prefix}-vpc-endpoints"

# SNS (38 char limit)
sns_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-monitoring-alarms-${random_string.setup.id}"
```

### Security Group Naming

```hcl
# Pattern: {service}_sg_name (38 char limit for most)
alb_accessible_ecs_service_sg_name = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-alb-ecs-sg-${random_string.setup.id}"
internal_ecs_sg_name               = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-internal-ecs-sg-${random_string.setup.id}"
db_sg_name                         = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-dbsecurity-${random_string.setup.id}"
elasticache_sg_name                = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-cachesecurity-${random_string.setup.id}"
efs_sg_name                        = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-efssecurity-${random_string.setup.id}"
docdb_sg_name                      = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-docdbsecurity-${random_string.setup.id}"
bastion_sg_name                    = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-bastionsecurity-${random_string.setup.id}"
vpn_sg_name                        = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-vpnsecurity-${random_string.setup.id}"
vpc_endpoints_sg_name              = "${local.vpc_endpoint_name_prefix}-vpc-endpoints"
```

### IAM Policy Naming

```hcl
# Pattern: {service}_policy_name (21-72 char limit depending on descriptiveness)
ecs_services_db_policy_name      = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-shared-db-task-execution-policy-${random_string.setup.id}"
ecs_services_docdb_policy_name   = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-shared-docdb-task-execution-policy-${random_string.setup.id}"
ecs_services_redis_policy_name   = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-shared-redis-task-execution-policy-${random_string.setup.id}"
ecs_services_efs_policy_name     = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-shared-efs-task-execution-policy-${random_string.setup.id}"
ecs_services_s3_policy_name      = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-shared-s3-task-execution-policy-${random_string.setup.id}"
ecs_services_command_exec_policy = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-ecs-command-exec-policy-${random_string.setup.id}"
bastion_s3_policy_name           = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-bastion-s3-access-${random_string.setup.id}"
```

### IAM Role Naming

```hcl
# Pattern: {service}_role_name (21-38 char limit)
ecs_services_role_names = { 
  for svc_name, _ in local.ecs_services : 
  svc_name => "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-${svc_name}-task-execution-role-${random_string.setup.id}" 
}

ecs_services_task_role_names = { 
  for svc_name, _ in local.ecs_services : 
  svc_name => "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-${svc_name}-task-role-${random_string.setup.id}" 
}
```

---

## Variable Structure

### Variable Declaration Pattern

**ALL variables MUST follow this exact structure:**

```hcl
variable "variable_name" {
  description = "Clear, concise description of what this variable controls"
  type        = string|number|bool|list(type)|map(type)|object({...})
  default     = appropriate_default_value
  
  validation {
    condition     = validation_logic
    error_message = "Clear error message explaining the validation requirement"
  }
}
```

### Variable Naming Convention

**Pattern:** `{service}_{property}_{descriptor}`

```hcl
# Examples from poc-idp
variable "region" { }
variable "account_id" { }
variable "application" { }
variable "environment" { }
variable "ecs_services" { }
variable "ecs_cloudwatch_log_group_retention_in_days" { }
variable "rds_create_cluster" { }
variable "rds_instance_class" { }
variable "rds_engine" { }
variable "rds_engine_version" { }
variable "elasticache_create_instance" { }
variable "elasticache_node_type" { }
variable "vpc_cidr_block" { }
variable "vpc_enable_high_available_nat_gateway" { }
variable "route53_dns_name" { }
variable "route53_backend_subdomain" { }
variable "route53_frontend_subdomain" { }
```

### Mandatory Variables

**These variables MUST exist in every deployment:**

```hcl
variable "region" {
  description = "Region in AWS cloud where all resources should be configured"
  type        = string
  default     = "us-east-1"
}

variable "account_id" {
  description = "AWS account ID where all resources should be configured"
  type        = string
}

variable "environment" {
  description = "Environment name in which resources should be deployed"
  type        = string
  default     = "dev"
}

variable "application" {
  description = "Name of the application"
  type        = string
  default     = ""
}

variable "assume_role_name" {
  description = "If not empty provider will assume this role name to get access to target AWS account"
  type        = string
  default     = ""
}
```

### Boolean Variables for Conditional Resources

**Pattern:** `{service}_create` or `{service}_create_{resource}`

```hcl
# Service creation flags
variable "rds_create_cluster" {
  description = "Whether to create a new RDS instances."
  type        = bool
  default     = true
}

variable "elasticache_create_instance" {
  description = "Whether to create a new RDS instances or not"
  type        = bool
  default     = true
}

variable "efs_create" {
  description = "Whether to create a new EFS."
  type        = bool
  default     = true
}

variable "docdb_create" {
  description = "Specifies whether to create DocumenDb resources."
  type        = bool
  default     = false
}

variable "bastion_create" {
  description = "Whether to create a bastion host?"
  type        = bool
  default     = true
}

variable "vpn_create" {
  description = "Whether to create a ClientVPN?"
  type        = bool
  default     = false
}

variable "vpc_endpoints_create" {
  description = "Specifies whether to create VPC endpoint resources."
  type        = bool
  default     = true
}

variable "s3_create_user_data_bucket" {
  description = "Whether to create the S3 user data bucket"
  type        = bool
  default     = true
}

variable "s3_create_frontend_bucket" {
  description = "Whether to create the S3 frontend bucket"
  type        = bool
  default     = true
}

variable "kms_create_shared_key" {
  description = "Whether to create a new shared KMS key. Set to false if you want to use an existing key."
  type        = bool
  default     = true
}

variable "route53_create_zone" {
  description = "Whether to create a new Route53 hosted zone"
  type        = bool
  default     = false
}
```

### Complex Object Variables

**ECS Services Pattern (from poc-idp):**

```hcl
variable "ecs_services" {
  description = "Map of ECS services to be created"
  type = map(object({
    desired_count = optional(number, 1)

    is_alb_target = bool
    need_ecr_repo = bool
    alb_path      = optional(string)

    task_role_arn = optional(string)

    enable_execute_command = optional(bool, false)

    execution_role_arn = optional(string, null)

    permissions = optional(object({
      elasticache = optional(bool, false)
      rds         = optional(bool, false)
      efs         = optional(bool, false)
      docdb       = optional(bool, false)
      s3          = optional(bool, false)
      }), {
      elasticache = false
      rds         = false
      efs         = false
      docdb       = false
      s3          = false
    })
    
    extra_secrets = optional(list(object({
      name      = string
      valueFrom = string
    })), [])

    container_definitions = list(object({
      name  = string
      image = string

      registry_access_secret_arn = optional(string)

      cpu    = number
      memory = number

      health_check_path = optional(string, "/")

      essential = optional(bool, true)

      command = optional(list(string))

      open_ports = list(number)

      environment = optional(any, {})
    }))
    
    autoscaling = optional(object({
      min_capacity        = number
      max_capacity        = number
      cpu_target_value    = optional(number)
      memory_target_value = optional(number)
      scale_in_cooldown   = optional(number, 300)
      scale_out_cooldown  = optional(number, 60)
    }))
  }))
  default = {}

  validation {
    condition = alltrue([
      for s in var.ecs_services : alltrue([
        for c in s.container_definitions : (
          (c.cpu == 256 && contains([512, 1024], c.memory)) ||
          (c.cpu == 512 && contains([1024, 2048], c.memory)) ||
          (c.cpu == 1024 && contains([2048, 3072, 4096], c.memory)) ||
          (c.cpu == 2048 && contains([4096, 5120, 6144, 7168, 8192], c.memory)) ||
          (c.cpu == 4096 && c.memory >= 8192 && c.memory <= 30720 && c.memory % 1024 == 0)
        )
      ])
    ])
    error_message = "Each container's CPU and memory must match ECS Fargate allowed combinations. https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-tasks-services.html#fargate-tasks-size"
  }

  validation {
    condition = length([
      for s in values(var.ecs_services) : s.alb_path
      if contains(keys(s), "alb_path") && s.alb_path != null
      ]) == length(
      toset([
        for s in values(var.ecs_services) : s.alb_path
        if contains(keys(s), "alb_path") && s.alb_path != null
      ])
    )

    error_message = "All defined alb_path values must be unique across ecs_services."
  }
}
```

### Validation Patterns

**Certificate ARN Validation:**

```hcl
variable "route53_existing_certificate_arn" {
  description = "The ARN of the existing ACM certificate to use for the ALB"
  type        = string
  default     = ""

  validation {
    error_message = "Certificate should be located in the same region as the all resources"
    condition     = var.route53_existing_certificate_arn == "" || strcontains(var.route53_existing_certificate_arn, var.region)
  }
}

variable "route53_existing_cloudfront_certificate_arn" {
  description = "The ARN of the existing ACM certificate to use for the CloudFront. Must be located in us-east-1"
  type        = string
  default     = ""
  
  validation {
    error_message = "Certificate should be located in us-east-1 region"
    condition     = var.route53_existing_cloudfront_certificate_arn == "" || strcontains(var.route53_existing_cloudfront_certificate_arn, "us-east-1")
  }
}
```

**Enum Validation:**

```hcl
variable "cloudfront_price_class" {
  description = "The price class for the CloudFront distribution"
  type        = string
  default     = "PriceClass_200"
  
  validation {
    condition     = contains(["PriceClass_99", "PriceClass_200", "PriceClass_All"], var.cloudfront_price_class)
    error_message = "Invalid value for cloudfront_price_class. Must be one of: PriceClass_99, PriceClass_200, PriceClass_All"
  }
}

variable "ecr_image_mutability" {
  description = "Provide image mutability"
  type        = string
  default     = "IMMUTABLE"

  validation {
    condition     = contains(["IMMUTABLE", "MUTABLE"], var.ecr_image_mutability)
    error_message = "Variable ecr_image_mutability must be one of: IMMUTABLE, MUTABLE"
  }
}
```

**Path Validation:**

```hcl
variable "uri_frontend_path" {
  description = "Cloudfront path to the frontend S3 bucket starting with the '/' ."
  type        = string
  default     = "/"
  
  validation {
    condition     = substr(var.uri_frontend_path, 0, 1) == "/"
    error_message = "frontend_path must start with '/'"
  }
}

variable "uri_backend_path" {
  description = "Cloudfront path to the backend ALB starting with '/' (only used if backend_subdomain == frontend_subdomain). If in use cant be the same as frontend_path"
  type        = string
  default     = "/api"
  
  validation {
    condition     = var.route53_backend_subdomain == var.route53_frontend_subdomain ? var.uri_backend_path != var.uri_frontend_path : true
    error_message = "backend_path and frontend_path cannot be the same if backend_subdomain is the same as frontend_subdomain"
  }
  
  validation {
    condition     = substr(var.uri_backend_path, 0, 1) == "/"
    error_message = "backend_path must start with '/'"
  }
}
```

**Conditional Validation:**

```hcl
variable "sns_monitoring_recepients_email_list" {
  description = "List of the email addresses to which notification will be send"
  type        = list(string)
  default     = []

  validation {
    condition     = length(var.sns_monitoring_recepients_email_list) > 0 && (length(var.rds_alarms) > 0 || length(var.elasticache_alarms) > 0 || length(var.ecs_alarms) > 0 || length(var.alb_alarms) > 0 || length(var.cloudfront_alarms) > 0 || length(var.route53_health_check_endpoint) > 0)
    error_message = "Variable 'sns_monitoring_recepients_email_list' should be defined when monitoring alarms are defined"
  }
}
```

---

## Module Usage Pattern

### Module Source Comments

**ALWAYS include version comments in module source:**

```hcl
module "module_name" {
  source = "./modules/module_path" // vX.Y.Z
  # or
  source = "terraform-aws-modules/service/aws" // vX.Y.Z
}
```

### Security Group Module Pattern

**EXACT pattern from poc-idp:**

```hcl
module "{service}_sg" {
  source = "./modules/vpn_sg" // v5.3.0

  count = var.{service}_create ? 1 : 0

  name            = local.{service}_sg_name
  use_name_prefix = false
  description     = "Security group for {service}"
  vpc_id          = module.vpc.vpc_id

  ingress_with_source_security_group_id = concat(var.vpn_create ?
    [{
      rule                     = "{protocol}-tcp"
      source_security_group_id = module.vpn_sg[0].security_group_id
    }] : [], var.bastion_create ?
    [{
      rule                     = "{protocol}-tcp"
      source_security_group_id = module.bastion_sg[0].security_group_id
    }] : []
  )

  ingress_cidr_blocks = module.vpc.private_subnets_cidr_blocks
  ingress_rules       = ["{protocol}-tcp"]
}

# Example: RDS Security Group
module "db_sg" {
  source = "./modules/vpn_sg" // v5.3.0

  count = var.rds_create_cluster ? 1 : 0

  name            = local.db_sg_name
  use_name_prefix = false
  description     = "Security group for RDS"
  vpc_id          = module.vpc.vpc_id

  ingress_with_source_security_group_id = concat(var.vpn_create ?
    [{
      rule                     = var.rds_engine == "aurora-postgresql" ? "postgresql-tcp" : "mysql-tcp"
      source_security_group_id = module.vpn_sg[0].security_group_id
    }] : [], var.bastion_create ?
    [{
      rule                     = var.rds_engine == "aurora-postgresql" ? "postgresql-tcp" : "mysql-tcp"
      source_security_group_id = module.bastion_sg[0].security_group_id
    }] : []
  )

  ingress_cidr_blocks = module.vpc.private_subnets_cidr_blocks
  ingress_rules       = var.rds_engine == "aurora-postgresql" ? ["postgresql-tcp"] : ["mysql-tcp"]
}
```

### VPC Module Pattern

```hcl
module "vpc" {
  source = "./modules/vpc/modules/vpc-create-or-adopt" // 0.3.0

  # Use already existing VPC...
  existing_vpc_id = var.vpc_id

  # ... or create a new one
  vpc_name       = local.vpc_name
  vpc_cidr_block = local.base_cidr_block
  azs            = local.azs

  enable_nat_gateway                    = true
  vpc_enable_high_available_nat_gateway = var.vpc_enable_high_available_nat_gateway

  vpc_flow_log_enable         = var.vpc_flow_log_enable
  flow_log_destination_type   = "cloud-watch-logs"
  flow_log_destination_arn    = var.vpc_flow_log_destination_arn
  vpc_flow_log_retention_days = var.vpc_flow_log_retention_days

  private_subnet_names  = [for i in range(3) : "${local.vpc_name}-private"]
  public_subnet_names   = [for i in range(3) : "${local.vpc_name}-public"]
  database_subnet_names = [for i in range(3) : "${local.vpc_name}-db"]

  default_route_table_name = local.vpc_name

  create_database_subnet_group = false

  create_database_subnet_route_table = true

  public_route_table_tags = {
    Name = "${local.vpc_name}-public-rt"
  }
  private_route_table_tags = {
    Name = "${local.vpc_name}-private-rt"
  }
  database_route_table_tags = {
    Name = "${local.vpc_name}-db-rt"
  }
}
```

### RDS Module Pattern

```hcl
module "db" {
  source = "./modules/db" // v9.15.0

  count = var.rds_create_cluster ? 1 : 0

  name               = local.db_name
  engine             = var.rds_engine
  engine_version     = var.rds_engine_version != "" ? var.rds_engine_version : data.aws_rds_engine_version.db[0].latest
  storage_type       = var.rds_storage_type
  kms_key_id         = aws_kms_alias.shared.target_key_arn
  availability_zones = local.azs

  instances = {
    for i in range(1, var.rds_instance_count + 1) : i => {
      instance_class = var.rds_instance_class
    }
  }

  # DB parameter group
  create_db_cluster_parameter_group          = true
  create_db_parameter_group                  = true
  db_cluster_parameter_group_name            = local.db_parameter_group_name
  db_cluster_parameter_group_use_name_prefix = false
  db_cluster_parameter_group_family          = var.rds_family != "" ? var.rds_family : data.aws_rds_engine_version.db[0].parameter_group_family
  db_parameter_group_family                  = var.rds_family != "" ? var.rds_family : data.aws_rds_engine_version.db[0].parameter_group_family
  db_cluster_parameter_group_parameters      = var.rds_parameters

  # DB subnet group
  create_db_subnet_group = true
  db_subnet_group_name   = local.db_subnet_group_name
  subnets                = module.vpc.database_subnets

  database_name                       = "maindb"
  master_username                     = "dbadmin"
  master_password                     = random_password.db_password[0].result
  manage_master_user_password         = false
  iam_database_authentication_enabled = var.rds_iam_database_authentication_enabled

  # DB Security Group
  create_security_group  = false
  vpc_security_group_ids = [module.db_sg[0].security_group_id]

  allow_major_version_upgrade  = var.rds_allow_major_version_upgrade
  apply_immediately            = var.rds_apply_immediately
  backup_retention_period      = 7
  preferred_maintenance_window = "Mon:06:00-Mon:09:00"
  preferred_backup_window      = "05:25-05:55"
  skip_final_snapshot          = true

  cluster_performance_insights_enabled          = var.rds_performance_insights_enabled
  cluster_performance_insights_retention_period = var.rds_performance_insights_enabled ? 7 : 0
  cluster_performance_insights_kms_key_id       = var.rds_performance_insights_enabled ? aws_kms_alias.shared.target_key_arn : null

  # DB Monitoring
  create_monitoring_role      = var.rds_create_monitoring_role
  cluster_monitoring_interval = var.rds_monitoring_interval
  monitoring_role_arn         = var.rds_monitoring_role_arn
  iam_role_name               = local.db_monitoring_role_name
  iam_role_description        = "IAM Role for RDS monitoring"
  database_insights_mode      = var.rds_database_insights_mode

  # DB Logs
  enabled_cloudwatch_logs_exports        = var.rds_enabled_cloudwatch_logs_exports
  create_cloudwatch_log_group            = var.rds_create_cloudwatch_log_group
  cloudwatch_log_group_retention_in_days = var.rds_cloudwatch_log_group_retention_in_days
  cloudwatch_log_group_kms_key_id        = aws_kms_alias.shared.target_key_arn
  cloudwatch_log_group_skip_destroy      = false
  cloudwatch_log_group_class             = "STANDARD"
  cloudwatch_log_group_tags              = merge(local.tags, var.additional_tags)

  # Database Deletion Protection
  deletion_protection      = var.rds_deletion_protection
  delete_automated_backups = true

  cluster_timeouts = {
    create = "40m"
    delete = "80m"
    update = "60m"
  }
  
  tags = {
    Name = local.db_name
  }
}
```

### ALB Module Pattern

```hcl
# trivy:ignore:AVD-AWS-0053    Ignore this issue, as the ALB should be publicly accessible
# trivy:ignore:AVD-AWS-0054
module "alb" {
  source = "./modules/alb" // v9.15.0
  
  name               = local.alb_name
  load_balancer_type = "application"

  internal = local.route53_frontend_backend_domain_is_the_same ? true : false

  vpc_id  = module.vpc.vpc_id
  subnets = local.route53_frontend_backend_domain_is_the_same ? module.vpc.private_subnets : module.vpc.public_subnets

  enable_deletion_protection = var.ecs_enable_alb_deletion_protection

  listeners = merge({ 
    http = {
      port     = 80
      protocol = "HTTP"
      name     = "HTTP Listener"

      redirect = {
        port        = "443"
        protocol    = "HTTPS"
        status_code = "HTTP_301"
      }
    } 
  }, var.route53_enforce_acm_certificate_validation ? {
    https = {
      port     = 443
      protocol = "HTTPS"
      name     = "HTTPS Listener"

      certificate_arn = var.route53_existing_certificate_arn != "" ? var.route53_existing_certificate_arn : aws_acm_certificate_validation.cert_validation[0].certificate_arn

      rules = {
        for ind, obj in [for k, v in local.alb_target_services : { key = k, value = v }] : obj.key => {
          priority = 1000 - length(obj.value.alb_path) * 10 + ind
          actions = [
            {
              type             = "forward"
              target_group_key = obj.key
            }
          ]
          conditions = [
            {
              path_pattern = {
                values = [var.uri_backend_path != "/" ?
                "${var.uri_backend_path}${obj.value.alb_path}*" : "${obj.value.alb_path}*"]
              }
            }
          ]
        }
      }

      fixed_response = {
        content_type = "text/plain"
        message_body = "Not Found"
        status_code  = "404"
      }
    }
  } : {})

  target_groups = var.route53_enforce_acm_certificate_validation ? {
    for key, value in local.alb_target_services : key => {
      backend_port     = value.container_definitions[0].portMappings[0].hostPort
      backend_protocol = "HTTP"
      target_type      = "ip"

      create_attachment = false

      health_check = {
        enabled             = true
        interval            = 30
        path                = value.container_definitions[0].health_check_path
        port                = "traffic-port"
        healthy_threshold   = 3
        unhealthy_threshold = 3
        timeout             = 6
        protocol            = "HTTP"
        matcher             = "200-399"
      }
    }
  } : {}

  security_group_ingress_rules = {
    all_http = {
      from_port   = 80
      to_port     = 80
      ip_protocol = "tcp"
      description = "HTTP web traffic"
      cidr_ipv4   = "0.0.0.0/0"
    }
    all_https = {
      from_port   = 443
      to_port     = 443
      ip_protocol = "tcp"
      description = "HTTPS web traffic"
      cidr_ipv4   = "0.0.0.0/0"
    }
  }
  
  security_group_egress_rules = {
    all = {
      ip_protocol                  = "-1"
      referenced_security_group_id = module.alb_accessible_ecs_service_sg.security_group_id
      description                  = "Allow all outbound traffic ONLY to ECS services"
    }
  }

  depends_on = [aws_acm_certificate_validation.cert_validation]

  tags = {
    Name = local.alb_name
  }
}
```

### VPC Endpoints Module Pattern

```hcl
module "vpc_endpoints" {
  source = "./modules/vpc_endpoints/modules/vpc-endpoints" // v5.14.0

  create = var.vpc_endpoints_create

  vpc_id                     = module.vpc.vpc_id
  create_security_group      = true
  security_group_name        = local.vpc_endpoints_sg_name
  security_group_description = "VPC endpoints security group"

  security_group_ids = module.vpn_sg[*].security_group_id
  security_group_rules = {
    ingress_https = {
      description = "HTTPS from VPC"
      cidr_blocks = [module.vpc.vpc_cidr_block]
    }
  }

  endpoints = merge({
    s3 = {
      service      = "s3"
      service_type = "Gateway"
      route_table_ids = distinct(concat(
        module.vpc.private_route_table_ids,
        module.vpc.database_route_table_ids,
        module.vpc.public_route_table_ids
      ))
      tags = {
        Name = "${local.vpc_endpoint_name_prefix}-s3-vpc-endpoint"
      }
    }
    }, {
    for svc in var.vpc_endpoint_services : replace(svc, ".", "_") => {
      service             = svc
      service_type        = "Interface"
      private_dns_enabled = true
      subnet_ids          = module.vpc.database_subnets
      tags = {
        Name = substr("${local.vpc_endpoint_name_prefix}-${replace(svc, ".", "_")}-vpc-endpoint", 0, 64)
      }
    }
  })
}
```

---

## Resource Patterns

### Random String for Unique IDs

**ALWAYS use in main.tf:**

```hcl
resource "random_string" "setup" {
  length  = 6
  special = false
  upper   = false
}
```

### Random Password Pattern

```hcl
resource "random_password" "{service}_password" {
  count = var.{service}_create ? 1 : 0

  length           = 24
  min_lower        = 1
  min_numeric      = 1
  min_special      = 1
  min_upper        = 1
  override_special = "!#$^-"
}

# Example: RDS Password
resource "random_password" "db_password" {
  count = var.rds_create_cluster ? 1 : 0

  length           = 24
  min_lower        = 1
  min_numeric      = 1
  min_special      = 1
  min_upper        = 1
  override_special = "!#$^-"
}
```

### Secrets Manager Pattern

```hcl
resource "aws_secretsmanager_secret" "{service}_secrets" {
  count = var.{service}_create ? 1 : 0

  name        = local.{service}_secret_name
  kms_key_id  = aws_kms_alias.shared.target_key_arn
  description = "Stores {service} secrets of the deployed environment"

  tags = {
    Name = local.{service}_secret_name
  }
}

resource "aws_secretsmanager_secret_version" "{service}_secrets_version" {
  count = var.{service}_create ? 1 : 0

  secret_id     = aws_secretsmanager_secret.{service}_secrets[0].id
  secret_string = jsonencode(local.{service}_secret_map)
}

# Example: RDS Secrets
resource "aws_secretsmanager_secret" "db_secrets" {
  count = var.rds_create_cluster ? 1 : 0

  name        = local.db_secret_name
  kms_key_id  = aws_kms_alias.shared.target_key_arn
  description = "Stores RDS secrets of the deployed environment"

  tags = {
    Name = local.db_secret_name
  }
}

resource "aws_secretsmanager_secret_version" "db_secrets_version" {
  count = var.rds_create_cluster ? 1 : 0

  secret_id     = aws_secretsmanager_secret.db_secrets[0].id
  secret_string = jsonencode(local.db_secret_map)
}
```

### IAM Role Pattern

```hcl
resource "aws_iam_role" "{service}_role" {
  count = var.{service}_create ? 1 : 0

  name = local.{service}_role_name

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "{service}.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })

  tags = {
    Name = local.{service}_role_name
  }
}
```

### IAM Policy Pattern

```hcl
resource "aws_iam_policy" "{service}_policy" {
  count = var.{service}_create ? 1 : 0

  name = local.{service}_policy_name

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Sid    = "DescriptiveStatementId"
        Effect = "Allow"
        Action = [
          "service:Action1",
          "service:Action2"
        ]
        Resource = "arn:aws:service:${var.region}:${var.account_id}:resource/*"
      }
    ]
  })
}

# Example: ECS Task Execution Policy for RDS Access
resource "aws_iam_policy" "ecs_exec_db_secrets" {
  count = var.rds_create_cluster ? 1 : 0

  name = local.ecs_services_db_policy_name

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Sid    = "SecretsManagerAccess"
        Effect = "Allow"
        Action = [
          "secretsmanager:GetSecretValue",
          "secretsmanager:DescribeSecret"
        ]
        Resource = aws_secretsmanager_secret.db_secrets[0].arn
      },
      {
        Sid    = "KMSAccess"
        Effect = "Allow"
        Action = [
          "kms:Decrypt",
          "kms:GenerateDataKey*"
        ]
        Resource = aws_kms_alias.shared.target_key_arn
      }
    ]
  })
}
```

### Conditional Resource Creation

```hcl
# Using count for simple conditional
resource "aws_{service}_resource" "main" {
  count = var.{service}_create ? 1 : 0
  
  name = local.{service}_name
  # ... configuration
  
  tags = {
    Name = local.{service}_name
  }
}

# Using for_each for multiple similar resources
resource "aws_{service}_resource" "multiple" {
  for_each = var.{service}_create ? local.{service}_configs : {}

  name = local.{service}_names[each.key]
  # ... configuration
  
  tags = {
    Name = local.{service}_names[each.key]
  }
}

# Example: ECR Repositories
resource "aws_ecr_repository" "ecr_repo" {
  for_each = local.ecs_filtered_need_repo

  name                 = local.ecr_names[each.key]
  image_tag_mutability = var.ecr_image_mutability
  force_delete         = var.ecr_allow_forced_delete

  image_scanning_configuration {
    scan_on_push = true
  }

  encryption_configuration {
    encryption_type = "KMS"
    kms_key         = aws_kms_alias.shared.target_key_arn
  }

  tags = {
    Name = local.ecr_names[each.key]
  }
}
```

### Data Source Pattern

```hcl
# Availability Zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Caller Identity
data "aws_caller_identity" "current" {}

# RDS Engine Version
data "aws_rds_engine_version" "db" {
  count = var.rds_create_cluster ? 1 : 0

  engine             = var.rds_engine
  preferred_versions = var.rds_engine_version != "" ? [var.rds_engine_version] : null
}

# Route53 Zone
data "aws_route53_zone" "zone" {
  count = var.route53_configure_dns && !var.route53_create_zone ? 1 : 0

  name         = var.route53_dns_name
  private_zone = false
}

# External Script Execution
data "external" "deploy_details" {
  program = local.is_linux ? ["bash", "-c", local.bash_script] : ["powershell", "-Command", local.powershell_script]
}
```

---

## Tagging Strategy

### Standard Tags

**ALL resources MUST have these tags (defined in locals.tf):**

```hcl
locals {
  tags = {
    environment      = var.environment
    application      = var.application
    managed_by       = "terraform"
    terraform_module = "ecs-typical-cluster"
    setup_id         = random_string.setup.id
  }
}
```

### Resource-Specific Tags

**Minimum tag for all resources:**

```hcl
tags = {
  Name = local.resource_name
}
```

### Merged Tags Pattern

**When additional tags are needed:**

```hcl
tags = merge(
  {
    Name = local.resource_name
  },
  local.tags,
  var.additional_tags
)

# Example from RDS CloudWatch Log Group
cloudwatch_log_group_tags = merge(local.tags, var.additional_tags)
```

### Tag Variables

```hcl
variable "additional_tags" {
  description = "The list of additional tags"
  type        = map(string)
  default     = {}
}

variable "ignore_tags_keys" {
  description = "Specified tags will be ignored by AWS provider and wouldn't show infrastructure drift"
  type        = list(string)
  default     = []
}

variable "ignore_tags_key_prefixes" {
  description = "Specified tag prefixes will be ignored by AWS provider and wouldn't show infrastructure drift"
  type        = list(string)
  default     = []
}
```

---

## Locals Structure

### Organization in locals.tf (740 lines)

**Sections in order:**

1. **Provider Configuration**
   - `assume_role_arn`
   - `is_linux` detection

2. **Deployment Details Scripts**
   - `bash_script` for Linux/Mac
   - `powershell_script` for Windows

3. **Standard Tags**
   - `tags` map with environment, application, managed_by, terraform_module, setup_id

4. **Module Flags for SSM**
   - `module_flags_for_json`
   - `enabled_modules_for_ssm_json`

5. **Deployment Metadata**
   - `deployment_timestamp`
   - `creation_date`, `creation_author`
   - `last_update_date_previous`, `last_update_author_previous`

6. **VPC Configuration**
   - `vpc_name`
   - `azs` (availability zones)
   - `base_cidr_block`

7. **S3 Configuration**
   - `s3_user_data_name`, `s3_frontend_name`
   - `s3_frontend_example_file_path`, `s3_frontend_example_files`

8. **VPC Endpoints**
   - `vpc_endpoint_name_prefix`
   - `vpc_endpoints_sg_name`

9. **ALB Configuration**
   - `alb_name`

10. **ECS Configuration**
    - `ecs_name`
    - `ecs_services` (complex transformation)
    - `ecs_resolved_images`
    - `ecs_filtered_need_repo`
    - `alb_target_services`
    - `ecs_task_definition_names`
    - `ecs_services_names`
    - `ecs_services_role_names`
    - `ecs_services_*_policy_name`
    - `ecs_services_task_role_names`
    - `alb_accessible_ecs_service_sg_name`
    - `internal_ecs_sg_name`
    - `ecs_services_fqdn`, `ecs_services_path_prefix`, `ecs_services_urls`

11. **RDS Configuration**
    - `db_secret_map`
    - `db_secret_name`, `db_sg_name`, `db_name`
    - `db_subnet_group_name`, `db_parameter_group_name`
    - `db_monitoring_role_name`

12. **ElastiCache Configuration**
    - `elasticache_secret_map`
    - `elasticache_secret_name`, `elasticache_replication_group_id`
    - `elasticache_sg_name`, `elasticache_subnet_group_name`
    - `list_elasticache_cluster_alarms`, `flattened_alarms`

13. **DocumentDB Configuration**
    - `docdb_secret_map`
    - `docdb_name`, `docdb_subnet_group_name`
    - `docdb_sg_name`, `docdb_secret_name`, `docdb_parameter_group_name`

14. **ECS Services Secrets**
    - `ecs_services_secrets_info`
    - `ecs_services_db_secrets`
    - `ecs_services_elasticache_secrets`
    - `ecs_services_docdb_secrets`
    - `ecs_services_extra_secrets`
    - `ecs_services_log_configuration`
    - `ecs_services_execution_role`
    - `ecs_services_execution_role_arns`
    - `ecs_needed_permissions_*`
    - `ecs_extra_secrets`

15. **ECS Autoscaling**
    - `ecs_services_with_autoscaling`
    - `ecs_autoscaling_policies`

16. **CloudMap Configuration**
    - `cloudmap_namespace_name`

17. **EFS Configuration**
    - `efs_name`, `efs_sg_name`, `efs_access_point_policy_name`

18. **AWS Backup Configuration**
    - `backup_override_tag_selector`, `backup_override_tag_selector_value`
    - `backup_enabled`
    - `backup_vault_name`, `backup_plan_name`, `backup_plan_rule_name`
    - `backup_iam_role_name`
    - `backup_policies`
    - `backup_services_arns`

19. **ECR Configuration**
    - `ecr_names`

20. **Route53 Configuration**
    - `route53_frontend_backend_domain_is_the_same`
    - `route53_alb_domain_name`
    - `route53_zone_id`

21. **CloudFront Configuration**
    - `cloudfront_name`, `cloudfront_*_origin_id`
    - `cloudfront_oac_name`, `cloudfront_vpc_origin_name`
    - `cloudfront_spa_routing_enabled`
    - `cloudfront_spa_function_path`, `cloudfront_spa_functions`
    - `cloudfront_spa_uri_backend_path`
    - `frontend_fqdn`, `frontend_url`

22. **RDS Alarms**
    - `replica_alarm_configurations`
    - `list_rds_cluster_alarms`
    - `flattened_replica_alarms`

23. **Bastion Configuration**
    - `bastion_name`, `bastion_sg_name`
    - `bastion_iam_resource_name`, `bastion_secret_name`
    - `bastion_ssh_key_name`, `bastion_s3_policy_name`

24. **VPN Configuration**
    - `vpn_name`, `vpn_sg_name`, `vpn_secret_name`
    - `vpn_config` (multi-line template)

25. **SNS Configuration**
    - `sns_name`

26. **ECR Pull Through Cache**
    - `prcr_create`
    - `prcr_suffix`
    - `prcr_extra_iam_policy_name`
    - `prcr_available_upstreams`
    - `prcr_used_upstreams`
    - `prcr_used_private_upstreams`
    - `prcr_used_services`
    - `registry_pull_through_cache_rules`
    - `container_private_registry_secrets`
    - `ecs_external_registry_secret_access_policy`

### Complex Locals Patterns

**ECS Services Transformation:**

```hcl
ecs_services = { 
  for key, value in var.ecs_services : key => merge(
    value,
    {
      cpu    = sum([for container in value.container_definitions : container.cpu])
      memory = sum([for container in value.container_definitions : container.memory])

      container_definitions = tolist([
        for container_index, container in value.container_definitions : merge(
          container,
          {
            image = local.ecs_resolved_images[key][container_index]

            repositoryCredentials = container.registry_access_secret_arn != null ? {
              credentialsParameter = container.registry_access_secret_arn
            } : null

            logConfiguration = local.ecs_services_log_configuration[key]

            portMappings = [
              for port in container.open_ports : {
                protocol      = "tcp"
                containerPort = port
                hostPort      = port
              }
            ]

            mountPoints = var.efs_create && value.permissions.efs ? [
              {
                containerPath = "/data"
                sourceVolume  = "efs-volume"
              }
            ] : []

            secrets = concat(
              local.ecs_services_db_secrets[key],
              local.ecs_services_elasticache_secrets[key],
              local.ecs_services_docdb_secrets[key],
              local.ecs_services_extra_secrets[key]
            )
            
            environment = concat(
              [
                for k, v in container.environment : {
                  name  = k
                  value = v
                }
              ], 
              var.efs_create && value.permissions.efs ? [{
                name  = "STORAGE_PATH"
                value = "/data"
              }] : [],
              var.s3_create_user_data_bucket && value.permissions.s3 ? [{
                name  = "S3_DATA_BUCKET"
                value = module.s3_user_data[0].s3_bucket_id
              }] : [],
              [
                {
                  name  = "SERVICE_NAME"
                  value = container.name
                },
                {
                  name  = "SERVICE_PORT"
                  value = tostring(try(container.open_ports[0], ""))
                },
                {
                  name  = "SERVICE_PATH"
                  value = value.alb_path != null && value.alb_path != "" ? join("", compact([var.uri_backend_path, value.alb_path])) : ""
                },
                {
                  name  = "SERVICE_DOMAIN"
                  value = join(".", compact([var.route53_backend_subdomain, var.route53_dns_name]))
                },
                {
                  name  = "CLOUDMAP_DOMAIN"
                  value = local.cloudmap_namespace_name
                }
              ]
            )
          }
        )
      ])
      
      volumes = var.efs_create && value.permissions.efs ? [
        {
          name = "efs-volume"
          efsVolumeConfiguration = {
            fileSystemId      = module.efs[0].id
            rootDirectory     = "/"
            transitEncryption = "ENABLED"
            authorizationConfig = {
              accessPointId = module.efs[0].access_points["ecs_shared"].id
              iam           = "ENABLED"
            }
          }
        }
      ] : []
    }
  )
}
```

**Secret Maps:**

```hcl
# RDS Secret Map
db_secret_map = var.rds_create_cluster ? (
  var.rds_engine == "aurora-postgresql" ? {
    POSTGRES_MAIN_USER     = "dbadmin"
    POSTGRES_MAIN_PASSWORD = try(random_password.db_password[0].result, null)
    POSTGRES_MAIN_HOST     = module.db[0].cluster_endpoint
    POSTGRES_MAIN_PORT     = 5432
    POSTGRES_MAIN_DBNAME   = "maindb"
  } : {
    MYSQL_MAIN_USER     = "dbadmin"
    MYSQL_MAIN_PASSWORD = try(random_password.db_password[0].result, null)
    MYSQL_MAIN_HOST     = module.db[0].cluster_endpoint
    MYSQL_MAIN_PORT     = 3306
    MYSQL_MAIN_DBNAME   = "maindb"
  }
) : {}

# ElastiCache Secret Map
elasticache_secret_map = var.elasticache_create_instance ? {
  REDIS_URL      = "redis://${module.elasticache[0].replication_group_primary_endpoint_address}:6379?ssl=true&sslprotocols=Tls12"
  REDIS_HOST     = module.elasticache[0].replication_group_primary_endpoint_address
  REDIS_PORT     = 6379
  REDIS_TOKEN    = try(random_password.elasticache_token[0].result, null)
  REDIS_SSL_MODE = "true"
} : {}

# DocumentDB Secret Map
docdb_secret_map = var.docdb_create ? {
  MONGODB_HOST     = module.docdb.docdb_cluster_endpoint
  MONGODB_PORT     = module.docdb.docdb_port
  MONGODB_USER     = module.docdb.docdb_master_username
  MONGODB_PASSWORD = module.docdb.docdb_master_password
} : {}
```

---

## Setup Size Pattern

### Configuration by Size (from tfvars-sample examples)

**Small Setup (Development):**
```hcl
# Minimal resources for development/testing
- RDS: 1 instance, db.t4g.medium
- ElastiCache: disabled
- DocumentDB: disabled
- NAT Gateway: 1 (single AZ)
- Backup: disabled
- VPC Flow Logs: disabled
- VPC Endpoints: none or minimal
- Bastion: public IP enabled
- VPN: disabled
```

**Medium Setup (Staging/Small Production):**
```hcl
# Balanced resources for staging or small production
- RDS: 2 instances (1 writer, 1 reader), db.t4g.medium
- ElastiCache: enabled, 1 node, cache.t4g.medium
- DocumentDB: enabled, 1 instance, db.t4g.medium
- NAT Gateway: 1 (cost-optimized)
- Backup: enabled for all services
- VPC Flow Logs: optional
- VPC Endpoints: minimal (sts)
- Bastion: public IP enabled
- VPN: optional
- Deletion Protection: enabled for RDS
```

**Large Setup (Production):**
```hcl
# High availability for production
- RDS: 2+ instances (1 writer, 1+ readers), db.m8g.large or higher
- ElastiCache: enabled, 3 nodes, cache.m7g.large, Multi-AZ
- DocumentDB: enabled, 3 instances, db.r6g.large
- NAT Gateway: 3 (one per AZ for HA)
- Backup: enabled for all services
- VPC Flow Logs: enabled, 365 days retention
- VPC Endpoints: full set (sts, ecr.api, ecr.dkr, ecs, etc.)
- Bastion: disabled (use VPN instead)
- VPN: enabled with connection logging
- Deletion Protection: enabled for RDS
- CloudWatch Logs: enabled for RDS
- Enhanced Monitoring: enabled
```

### Example tfvars Structure

**From example-small-setup.tfvars:**

```hcl
####
## Generic configuration
####
account_id  = "253650698585" # CHANGE_ME_PLEASE
region      = "us-east-1"    # CHANGE_ME_PLEASE
application = "web"          # CHANGE_ME_PLEASE
environment = "dev"          # CHANGE_ME_PLEASE

ignore_tags_keys = ["AutoCreateDate", "AutoCreatedBy", "AutoRoleName", "IAM User Name"]

####
## ECS configuration
####
ecs_cloudwatch_log_group_retention_in_days = 7
ecs_enable_alb_deletion_protection         = false

ecs_services = {
  "test-nginx" = {
    desired_count = 1
    is_alb_target = true
    alb_path      = "/"
    need_ecr_repo = false
    container_definitions = [
      {
        name               = "nginx"
        image              = "public.ecr.aws/nginx/nginx:latest"
        cpu                = 256
        memory             = 512
        essential          = true
        command            = []
        open_ports         = [80]
        health_check_path  = "/"
        environment        = {}
      }
    ]
  }
}

uri_backend_path = "/"

####
## Service Flags
####
rds_create_cluster          = true
rds_instance_class          = "db.t4g.medium"
elasticache_create_instance = false
efs_create                  = true
docdb_create                = false
bastion_create              = true
bastion_associate_public_ip_address = true

####
## Route53 configuration
####
route53_backend_subdomain  = "api"
route53_dns_name           = "example.com" # CHANGE_ME_PLEASE
route53_frontend_subdomain = "www"

####
## VPC configuration
####
vpc_enable_high_available_nat_gateway = false
vpc_flow_log_enable                   = false
vpc_endpoint_services                 = []

####
## Backup configuration
####
backup_services = {}

####
## Monitoring configuration
####
sns_monitoring_recepients_email_list = [] # CHANGE_ME_PLEASE
```

---

## Output Structure

### Output Pattern

**NO SENSITIVE DATA in outputs - use Secrets Manager instead**

```hcl
#
# Do not put sensitive data to outputs. Save it in AWS Secret manager
#

output "{service}_{attribute}" {
  description = "Clear description of what this output provides"
  value       = resource_reference
}
```

### Conditional Outputs

```hcl
output "{service}_{attribute}" {
  description = "Description"
  value       = var.{service}_create ? resource[0].attribute : null
}

# Examples
output "db_cluster_endpoint" {
  description = "The connection endpoint"
  value       = var.rds_create_cluster ? module.db[0].cluster_endpoint : null
}

output "elasticache_replication_group_primary_endpoint_address" {
  description = "Address of the endpoint for the primary node in the replication group, if the cluster mode is disabled"
  value       = var.elasticache_create_instance ? module.elasticache[0].replication_group_primary_endpoint_address : null
}

output "efs_id" {
  description = "The ID that identifies the file system (e.g., `fs-ccfc0d65`)"
  value       = var.efs_create ? module.efs[0].id : null
}
```

### Map Outputs

```hcl
output "ecr_repository_url" {
  description = "URL of of the repository(s)"
  value = {
    for key, repo in aws_ecr_repository.ecr_repo :
    key => repo.repository_url
  }
}

output "deployed_ecs_services" {
  description = "Map of ECS services and their URLs"
  value       = local.ecs_services_urls
}
```

### Complex Conditional Outputs

```hcl
output "vpc_endpoints_arns" {
  description = "Map of VPC endpoint ARNs"
  value = can(module.vpc_endpoints) && can(module.vpc_endpoints.endpoints) ? {
    for k, ep in module.vpc_endpoints.endpoints : k => ep.arn
  } : {}
}
```

### Flattened List Outputs

```hcl
output "route53_nameservers" {
  description = "Route53 Nameservers"
  value       = flatten([aws_route53_zone.new_zone[*].name_servers, data.aws_route53_zone.zone[*].name_servers])
}

output "route53_records" {
  description = "Route53 Records"
  value       = flatten([aws_route53_record.ecs_alb[*].fqdn, aws_route53_record.cloudfront[*].fqdn])
}
```

### Try Function for Safe Access

```hcl
output "backup_vault_name" {
  description = "Name of the AWS backup vault"
  value       = try(aws_backup_vault.main[0].name, null)
}

output "vpc_endpoints_sg_id" {
  description = "Security group ID used for VPC endpoints"
  value       = try(module.vpc_endpoints.security_group_id, null)
}
```

### Conditional String Outputs

```hcl
output "bastion_ssh_connection_string" {
  description = "The full SSH command to connect to the bastion host."
  value       = var.bastion_create && var.bastion_associate_public_ip_address ? "ssh -i ~/.ssh/${aws_instance.bastion[0].key_name}.pem ubuntu@${aws_eip.bastion_eip[0].public_ip} -p 22" : null
}

output "bastion_secret_key_arn" {
  description = "Arn of the aws secret manager resource which holds ssh private key"
  value       = var.bastion_create && var.bastion_associate_public_ip_address && length(var.bastion_ssh_key_name) == 0 ? aws_secretsmanager_secret.private_key[0].arn : null
}
```

### One Function for Single Element

```hcl
output "bastion_instance_id" {
  description = "Bastion host ID"
  value       = one(aws_instance.bastion[*].id)
}

output "vpn_secretsmanager_secret_arn" {
  description = "The ARN of the AWS Secrets Manager secret storing the ClientVPN config."
  value       = one(aws_secretsmanager_secret_version.vpn_secret_version[*].arn)
}
```

---

## Comments and Documentation

### File Headers

**Minimal or no headers - let the filename speak**

```hcl
# rds.tf contains RDS cluster resources
# ecs.tf contains ECS cluster and services
```

### Inline Comments for Complex Logic

```hcl
# Calculate the number of NAT gateways based on high availability setting
# HA enabled: 3 NAT gateways (one per AZ)
# HA disabled: 1 NAT gateway (cost optimization)
enable_nat_gateway                    = true
vpc_enable_high_available_nat_gateway = var.vpc_enable_high_available_nat_gateway
```

### Security Scanner Ignore Comments

```hcl
# trivy:ignore:AVD-AWS-0053    Ignore this issue, as the ALB should be publicly accessible
# trivy:ignore:AVD-AWS-0054
module "alb" {
  source = "./modules/alb" // v9.15.0
  # ...
}
```

### Conditional Logic Comments

```hcl
# Use already existing VPC...
existing_vpc_id = var.vpc_id

# ... or create a new one
vpc_name       = local.vpc_name
vpc_cidr_block = local.base_cidr_block
```

### Section Comments in tfvars

```hcl
####
## Generic configuration
####

####
## ECS configuration
####

####
## RDS configuration
####
```

---

## Code Formatting Rules

### Indentation

- **2 spaces** for indentation (NO tabs)
- Consistent alignment of `=` signs in blocks

### Block Formatting

```hcl
resource "aws_service_resource" "name" {
  argument1 = "value1"
  argument2 = "value2"
  
  nested_block {
    nested_arg1 = "value1"
    nested_arg2 = "value2"
  }
  
  tags = {
    Name = local.resource_name
  }
}
```

### Alignment

**Align equals signs in variable blocks:**

```hcl
variable "region" {
  description = "Region in AWS cloud where all resources should be configured"
  type        = string
  default     = "us-east-1"
}
```

**Align equals signs in resource arguments:**

```hcl
name               = local.db_name
engine             = var.rds_engine
engine_version     = var.rds_engine_version != "" ? var.rds_engine_version : data.aws_rds_engine_version.db[0].latest
storage_type       = var.rds_storage_type
kms_key_id         = aws_kms_alias.shared.target_key_arn
availability_zones = local.azs
```

### Line Length

- Maximum 120 characters per line
- Break long lines at logical points (after commas, operators)

### Blank Lines

- One blank line between resources
- One blank line between major sections within a resource
- No trailing whitespace

### String Formatting

**Use double quotes for strings:**

```hcl
name        = "my-resource"
description = "This is a description"
```

**Use heredoc for multi-line strings:**

```hcl
local.bash_script = <<-EOT
  set -e
  repo_dir=$(git rev-parse --show-toplevel || pwd)
  repo_name=$(basename "$repo_dir")
EOT

local.vpn_config = <<-EOT
  client
  dev tun
  proto udp
  remote ${try(aws_ec2_client_vpn_endpoint.this[0].dns_name, "")} 443
EOT
```

---

## Terraform Commands

### Standard Workflow

```bash
# Format code
terraform fmt -recursive

# Validate configuration
terraform validate

# Initialize with backend
terraform init -backend-config=tfbackends/{env}.tfbackend -reconfigure

# Plan with variables
terraform plan -var-file=tfvars/{env}.tfvars

# Apply with variables
terraform apply -var-file=tfvars/{env}.tfvars

# Destroy with variables
terraform destroy -var-file=tfvars/{env}.tfvars
```

### Backend Configuration

**tfbackends/{env}.tfbackend:**

```hcl
bucket         = "terraform-state-bucket"
key            = "path/to/terraform.tfstate"
region         = "us-east-1"
dynamodb_table = "terraform-state-lock"
encrypt        = true
```

---

## Provider Configuration

### providers.tf Pattern

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
    external = {
      source  = "hashicorp/external"
      version = "~> 2.0"
    }
  }
}

provider "aws" {
  region = var.region

  assume_role {
    role_arn = local.assume_role_arn != "" ? local.assume_role_arn : null
  }

  ignore_tags {
    keys         = var.ignore_tags_keys
    key_prefixes = var.ignore_tags_key_prefixes
  }

  default_tags {
    tags = local.tags
  }
}
```

### backend.tf Pattern

```hcl
terraform {
  backend "s3" {
    # Configuration provided via -backend-config flag
    # See tfbackends/ directory for environment-specific configs
  }
}
```

---

## Critical Rules

### MUST Follow

1. ✅ **ALL resource names in locals.tf** - Never hardcode names in resources
2. ✅ **Use exact character limits** - AWS services have strict limits (21 for short, 38 for medium, 72 for policies)
3. ✅ **Include validation** - All variables must have validation blocks where applicable
4. ✅ **Use modules from poc-idp** - Don't create new module patterns without verification
5. ✅ **Follow file organization** - One service per file (ecs.tf, rds.tf, vpc.tf, etc.)
6. ✅ **Use random_string.setup.id** - For unique resource naming across all resources
7. ✅ **Tag all resources** - Minimum: Name tag, preferably merge with local.tags
8. ✅ **Conditional creation** - Use count or for_each with boolean variables (var.{service}_create)
9. ✅ **Module version comments** - Always include version in module source comments (// vX.Y.Z)
10. ✅ **Use count for conditionals** - Pattern: `count = var.{service}_create ? 1 : 0`
11. ✅ **Reference with [0]** - When using count, reference with resource[0].attribute
12. ✅ **Secrets in Secrets Manager** - Never put sensitive data in outputs or variables
13. ✅ **Use try() for safe access** - Pattern: `try(resource[0].attribute, null)`
14. ✅ **Follow naming pattern** - `${substr(var.application, 0, X)}-${substr(var.environment, 0, Y)}-{descriptor}-${random_string.setup.id}`

### MUST NOT Do

1. ❌ **Don't hardcode resource names** - Always use locals
2. ❌ **Don't exceed character limits** - Will cause AWS errors (21/38/72 char limits)
3. ❌ **Don't skip validation** - Prevents invalid configurations
4. ❌ **Don't create new patterns** - Follow existing poc-idp patterns exactly
5. ❌ **Don't mix services in files** - Keep separation clear (one service per .tf file)
6. ❌ **Don't use different naming** - Consistency is critical across all resources
7. ❌ **Don't skip tags** - Required for cost tracking and resource management
8. ❌ **Don't use inline conditionals without count** - Use proper count/for_each
9. ❌ **Don't put secrets in outputs** - Use AWS Secrets Manager instead
10. ❌ **Don't forget module versions** - Always comment module versions
11. ❌ **Don't use tabs** - Use 2 spaces for indentation
12. ❌ **Don't skip security group patterns** - Use vpn_sg module pattern
13. ❌ **Don't create resources without locals** - All names must be in locals.tf first
14. ❌ **Don't use different substr values** - Follow the 21/4 or 38/5 pattern exactly

---

## Checklist for New Components

When adding a new component, verify:

### Variables (variables.tf)
- [ ] Boolean `{service}_create` variable added with default value
- [ ] Service-specific variables added with proper naming (`{service}_{property}`)
- [ ] All variables have clear descriptions
- [ ] Validation blocks added where applicable (enums, ARNs, paths, etc.)
- [ ] Default values are sensible for small/dev environments
- [ ] Complex object types use `optional()` for optional fields

### Locals (locals.tf)
- [ ] Resource names added with correct character limits (21/38/72)
- [ ] Naming pattern follows: `${substr(var.application, 0, X)}-${substr(var.environment, 0, Y)}-{descriptor}-${random_string.setup.id}`
- [ ] Security group name added (if needed)
- [ ] IAM role/policy names added (if needed)
- [ ] Secret names added (if needed)
- [ ] All computed values defined in locals, not inline

### Service File ({service}.tf)
- [ ] New `{service}.tf` file created
- [ ] Resources use `count = var.{service}_create ? 1 : 0` pattern
- [ ] All resource names reference locals (local.{service}_name)
- [ ] Security groups follow module pattern (./modules/vpn_sg)
- [ ] IAM roles and policies follow pattern
- [ ] Secrets use Secrets Manager pattern
- [ ] Random passwords use proper pattern (if needed)
- [ ] All resources have Name tag minimum
- [ ] Module sources include version comments (// vX.Y.Z)

### Outputs (outputs.tf)
- [ ] Outputs added for key resource attributes
- [ ] Conditional outputs use ternary: `var.{service}_create ? resource[0].attribute : null`
- [ ] NO sensitive data in outputs (use Secrets Manager)
- [ ] Clear descriptions for all outputs
- [ ] Use `try()` for safe access where appropriate
- [ ] Use `one()` for single-element lists

### Alarms (alarms.tf)
- [ ] CloudWatch alarms added for the service (if applicable)
- [ ] Alarm variables added to variables.tf
- [ ] Alarm naming follows pattern
- [ ] SNS topic integration configured

### Testing
- [ ] Code formatted with `terraform fmt -recursive`
- [ ] Configuration validated with `terraform validate`
- [ ] Plan runs without errors
- [ ] Tested with small/medium/large setup configurations
- [ ] Verified character limits don't exceed AWS limits
- [ ] Tested conditional creation (create=true and create=false)

### Documentation
- [ ] Example added to tfvars files (small/medium/large)
- [ ] Comments added for complex logic
- [ ] Security scanner ignore comments added (if needed)
- [ ] Module usage documented in component-extension-guide

---

## Common Patterns Reference

### Conditional Resource with Count
```hcl
resource "aws_{service}_resource" "main" {
  count = var.{service}_create ? 1 : 0
  
  name = local.{service}_name
  # ... configuration
}
```

### Conditional Output
```hcl
output "{service}_endpoint" {
  description = "Endpoint for {service}"
  value       = var.{service}_create ? aws_{service}_resource.main[0].endpoint : null
}
```

### Security Group Module
```hcl
module "{service}_sg" {
  source = "./modules/vpn_sg" // v5.3.0

  count = var.{service}_create ? 1 : 0

  name            = local.{service}_sg_name
  use_name_prefix = false
  description     = "Security group for {service}"
  vpc_id          = module.vpc.vpc_id

  ingress_cidr_blocks = module.vpc.private_subnets_cidr_blocks
  ingress_rules       = ["{protocol}-tcp"]
}
```

### Secret with Password
```hcl
resource "random_password" "{service}_password" {
  count = var.{service}_create ? 1 : 0

  length           = 24
  min_lower        = 1
  min_numeric      = 1
  min_special      = 1
  min_upper        = 1
  override_special = "!#$^-"
}

resource "aws_secretsmanager_secret" "{service}_secrets" {
  count = var.{service}_create ? 1 : 0

  name        = local.{service}_secret_name
  kms_key_id  = aws_kms_alias.shared.target_key_arn
  description = "Stores {service} secrets"

  tags = {
    Name = local.{service}_secret_name
  }
}

resource "aws_secretsmanager_secret_version" "{service}_secrets_version" {
  count = var.{service}_create ? 1 : 0

  secret_id     = aws_secretsmanager_secret.{service}_secrets[0].id
  secret_string = jsonencode(local.{service}_secret_map)
}
```

### IAM Role and Policy
```hcl
resource "aws_iam_role" "{service}_role" {
  count = var.{service}_create ? 1 : 0

  name = local.{service}_role_name

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "{service}.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })

  tags = {
    Name = local.{service}_role_name
  }
}

resource "aws_iam_policy" "{service}_policy" {
  count = var.{service}_create ? 1 : 0

  name = local.{service}_policy_name

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Sid    = "ServiceAccess"
      Effect = "Allow"
      Action = ["service:Action"]
      Resource = "*"
    }]
  })
}

resource "aws_iam_role_policy_attachment" "{service}_policy_attachment" {
  count = var.{service}_create ? 1 : 0

  role       = aws_iam_role.{service}_role[0].name
  policy_arn = aws_iam_policy.{service}_policy[0].arn
}
```

---

## Related Guides

- **#component-extension-guide** - Step-by-step workflow for adding new components
- **#ecs-deployment-workflow** - Complete deployment process (includes setup size selection and S3 backend setup)
- **#troubleshooting-guide** - Common issues and solutions

---

## Quick Reference

### Character Limits
- **21 chars**: VPC, ALB, ECS cluster, CloudFront
- **38 chars**: S3, RDS, ElastiCache, EFS, DocumentDB, Bastion, VPN, ECR
- **9 chars**: DocumentDB subnet group (special case)
- **72 chars**: IAM policies with descriptive names

### Naming Pattern
```
${substr(var.application, 0, X)}-${substr(var.environment, 0, Y)}-{descriptor}-${random_string.setup.id}
```

### Module Versions (from poc-idp)
- VPC: `./modules/vpc/modules/vpc-create-or-adopt` // 0.3.0
- VPC Endpoints: `./modules/vpc_endpoints/modules/vpc-endpoints` // v5.14.0
- Security Groups: `./modules/vpn_sg` // v5.3.0
- RDS: `./modules/db` // v9.15.0
- ALB: `./modules/alb` // v9.15.0

### Common Variables
- `var.region` - AWS region
- `var.account_id` - AWS account ID
- `var.application` - Application name
- `var.environment` - Environment (dev/staging/prod)
- `var.{service}_create` - Boolean to create service
- `random_string.setup.id` - Unique 6-char ID

### Common Locals
- `local.{service}_name` - Resource name
- `local.{service}_sg_name` - Security group name
- `local.{service}_role_name` - IAM role name
- `local.{service}_policy_name` - IAM policy name
- `local.{service}_secret_name` - Secret name
- `local.tags` - Standard tags map
