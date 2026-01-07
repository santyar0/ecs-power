# Component Extension Guide

Extend poc-idp infrastructure with new AWS components while maintaining consistency, security, and best practices.

> **For exact coding patterns and style rules**, see `#poc-idp-style-guide`  
> **For complex customizations**, see `#ai-dlc-infrastructure-workflow`

## When to Extend

**Good Candidates:**
- Adding Lambda, API Gateway, SQS, SNS
- Adding ElastiSearch, Redis clusters
- Adding EFS, FSx storage
- Adding monitoring tools
- Adding CI/CD components

**Consider Alternatives:**
- Core networking changes → Use existing VPC patterns
- Security model changes → Extend existing IAM patterns
- Fundamental architecture changes → Separate deployment

---

## Extension Philosophy

1. **Extend, Don't Replace** - Build upon existing patterns
2. **Consistency First** - Follow established conventions
3. **Security by Design** - Implement controls from the start
4. **Operational Excellence** - Include monitoring from day one

---

## Repository Architecture

```
├── main.tf           # Provider configuration
├── variables.tf      # Input variables with validation
├── locals.tf         # Computed values and naming
├── outputs.tf        # Output values
├── data.tf           # Data sources
├── backend.tf        # Backend configuration
├── providers.tf      # Provider requirements
├── {service}.tf      # Service-specific resources
├── alarms.tf         # CloudWatch alarms
└── modules/          # Reusable modules
    └── {service}/
```

---

## Naming Conventions

**Character Limits:**
| Type | Limit | Examples |
|------|-------|----------|
| Short | 21 chars | VPC, ALB, ECS cluster |
| Medium | 38 chars | S3, RDS, ElastiCache, EFS |
| Policy | 72 chars | IAM policies |

**Pattern:**
```hcl
# locals.tf
resource_name = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-{service}-${random_string.setup.id}"
```

---

## 8 Rules for Adding Components

### Rule 1: File Placement

Create dedicated `{service}.tf` file for each AWS service:
```
# Adding Lambda
Create: lambda.tf
Update: variables.tf, locals.tf, outputs.tf, alarms.tf
```

### Rule 2: Variable Structure

```hcl
variable "service_property_name" {
  description = "Clear description"
  type        = string
  default     = "default_value"
  
  validation {
    condition     = validation_logic
    error_message = "Clear error message"
  }
}
```

**Naming:** `{service}_{property}_{descriptor}`

### Rule 3: Resource Naming in Locals

```hcl
# locals.tf
lambda_function_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-lambda-${random_string.setup.id}"
lambda_role_name     = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-lambda-role-${random_string.setup.id}"
```

### Rule 4: Security Group Pattern

```hcl
module "service_sg" {
  source = "./modules/vpn_sg"

  name        = local.service_sg_name
  vpc_id      = module.vpc.vpc_id
  description = "Security Group for [service]"

  ingress_with_source_security_group_id = [
    {
      rule                     = "all-all"
      source_security_group_id = module.other_sg.security_group_id
    }
  ]

  egress_rules = ["all-all"]

  tags = { Name = local.service_sg_name }
}
```

### Rule 5: IAM Policy Pattern

```hcl
resource "aws_iam_role" "service_role" {
  name = local.service_role_name

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Principal = { Service = "service.amazonaws.com" }
      Action    = "sts:AssumeRole"
    }]
  })

  tags = { Name = local.service_role_name }
}

resource "aws_iam_policy" "service_policy" {
  name = local.service_policy_name

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Sid      = "DescriptiveStatementId"
      Effect   = "Allow"
      Action   = ["service:Action1", "service:Action2"]
      Resource = "arn:aws:service:${var.region}:${var.account_id}:resource/*"
    }]
  })
}
```

### Rule 6: Tagging

```hcl
tags = { Name = local.resource_name }

# Extended tags
tags = merge(local.tags, {
  Service     = "service-name"
  Environment = var.environment
  Component   = "component-name"
})
```

### Rule 7: Conditional Creation

```hcl
# Using count
resource "aws_service_resource" "main" {
  count = var.service_create ? 1 : 0
  name  = local.service_name
}

# Using for_each
resource "aws_service_resource" "multiple" {
  for_each = local.service_configs
  name     = local.service_names[each.key]
}
```

### Rule 8: Module Integration

```hcl
module "service_name" {
  source = "./modules/service_name"

  # Required parameters first
  name   = local.service_name
  vpc_id = module.vpc.vpc_id

  # Optional parameters
  instance_class = var.service_instance_class
  
  # Tags last
  tags = { Name = local.service_name }
}
```

---

## Extension Workflow

### Phase 1: Planning

1. **Component Analysis** - Research AWS service best practices
2. **Architecture Design** - Design integration with existing infrastructure
3. **Variable Planning** - Plan variable structure following poc-idp patterns

### Phase 2: Implementation

1. **Variables** - Add to `variables.tf` with validation
2. **Locals** - Add computed names to `locals.tf`
3. **Resources** - Create dedicated `{service}.tf` file
4. **Security** - Add IAM roles, security groups
5. **Monitoring** - Add CloudWatch alarms

### Phase 3: Integration

1. **Configuration** - Update tfvars templates
2. **Validation** - Run `terraform fmt`, `validate`, `plan`
3. **Security Check** - Review IAM policies, security groups

### Phase 4: Deployment

1. **Staged Deploy** - Dev → Staging → Production
2. **Integration Test** - Verify service communication
3. **Operational Validation** - Confirm monitoring works

---

## Common Patterns

### Adding Lambda

```hcl
# variables.tf
variable "lambda_create" {
  description = "Whether to create Lambda functions"
  type        = bool
  default     = false
}

variable "lambda_functions" {
  description = "Map of Lambda functions to create"
  type = map(object({
    runtime     = string
    handler     = string
    memory_size = optional(number, 128)
    timeout     = optional(number, 30)
  }))
  default = {}
}

# locals.tf
lambda_function_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-lambda-${random_string.setup.id}"
```

### Adding EFS

```hcl
# variables.tf
variable "efs_create" {
  description = "Whether to create EFS file system"
  type        = bool
  default     = false
}

variable "efs_performance_mode" {
  description = "EFS performance mode"
  type        = string
  default     = "generalPurpose"
  
  validation {
    condition     = contains(["generalPurpose", "maxIO"], var.efs_performance_mode)
    error_message = "Must be generalPurpose or maxIO."
  }
}
```

### Adding SQS

```hcl
# variables.tf
variable "queues" {
  description = "Map of SQS queues to create"
  type = map(object({
    visibility_timeout_seconds = optional(number, 30)
    message_retention_seconds  = optional(number, 1209600)
    fifo_queue                = optional(bool, false)
  }))
  default = {}
}
```

---

## Setup Size Configurations

Components should adapt to setup size:

```hcl
locals {
  component_configs = {
    small = {
      instance_size = "small"
      ha_enabled    = false
      backup        = false
    }
    medium = {
      instance_size = "medium"
      ha_enabled    = false
      backup        = true
    }
    large = {
      instance_size = "large"
      ha_enabled    = true
      backup        = true
    }
  }
}
```

---

## AWS Best Practices

### Security
- Least privilege IAM
- Encryption at rest and in transit
- Security groups with minimal access
- Secrets Manager for sensitive data
- CloudTrail logging

### Reliability
- Multi-AZ for production
- Health checks
- Automated backups
- Disaster recovery planning

### Performance
- Right-sizing instances
- Auto scaling where applicable
- Caching with ElastiCache
- CloudFront for content delivery

### Cost
- Resource tagging for cost allocation
- Lifecycle policies
- Reserved instances for predictable workloads
- Cost monitoring

---

## Complete Example: Adding Lambda

### 1. Variables (variables.tf)
```hcl
variable "lambda_create" {
  description = "Whether to create Lambda functions"
  type        = bool
  default     = false
}

variable "lambda_functions" {
  description = "Map of Lambda functions"
  type = map(object({
    runtime     = string
    handler     = string
    memory_size = optional(number, 128)
    timeout     = optional(number, 30)
    environment = optional(map(string), {})
  }))
  default = {}
}
```

### 2. Locals (locals.tf)
```hcl
lambda_functions = {
  for name, config in var.lambda_functions : name => merge(config, {
    function_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-${name}-${random_string.setup.id}"
    role_name     = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-${name}-lambda-role-${random_string.setup.id}"
  })
}
```

### 3. Resources (lambda.tf)
```hcl
resource "aws_iam_role" "lambda_execution_role" {
  for_each = var.lambda_create ? local.lambda_functions : {}
  name     = each.value.role_name

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Principal = { Service = "lambda.amazonaws.com" }
      Action    = "sts:AssumeRole"
    }]
  })

  tags = { Name = each.value.role_name }
}

resource "aws_lambda_function" "functions" {
  for_each      = var.lambda_create ? local.lambda_functions : {}
  function_name = each.value.function_name
  role          = aws_iam_role.lambda_execution_role[each.key].arn
  handler       = each.value.handler
  runtime       = each.value.runtime
  memory_size   = each.value.memory_size
  timeout       = each.value.timeout

  environment { variables = each.value.environment }
  tags = { Name = each.value.function_name }
}
```

### 4. Outputs (outputs.tf)
```hcl
output "lambda_function_arns" {
  description = "ARNs of created Lambda functions"
  value = {
    for name, func in aws_lambda_function.functions : name => func.arn
  }
}
```

---

## Validation Checklist

### Architecture
- [ ] Follows file organization pattern
- [ ] Uses consistent naming conventions
- [ ] Implements proper tagging
- [ ] Includes appropriate outputs

### Security
- [ ] Implements least privilege IAM
- [ ] Enables encryption where applicable
- [ ] Uses security groups appropriately
- [ ] Follows network security best practices

### Operations
- [ ] Includes monitoring and alarms
- [ ] Implements backup strategies
- [ ] Supports high availability (if needed)
- [ ] Includes documentation

### Cost
- [ ] Uses appropriate instance sizes
- [ ] Implements lifecycle policies
- [ ] Includes cost allocation tags

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Resource naming conflicts | Check naming patterns in locals.tf |
| Circular dependencies | Review security group references |
| IAM permission errors | Follow least privilege patterns |
| Network connectivity | Check security groups and routing |

---

## Related Guides

- `#poc-idp-style-guide` - **Exact coding patterns from repository**
- `#ecs-deployment-workflow` - Deployment steps
- `#troubleshooting-guide` - Issue resolution
