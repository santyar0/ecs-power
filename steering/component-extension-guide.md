# Component Extension Guide

This guide provides comprehensive rules and patterns for extending the poc-idp infrastructure with new AWS components while maintaining consistency, security, and best practices.

## Repository Architecture Analysis

The poc-idp repository follows a well-structured pattern that must be maintained when adding new components:

### File Organization Pattern
```
├── main.tf                    # Provider configuration and main resources
├── variables.tf               # All input variables with validation
├── locals.tf                  # Computed values and resource naming
├── outputs.tf                 # Output values for other modules
├── data.tf                    # Data source definitions
├── backend.tf                 # Terraform backend configuration
├── providers.tf               # Provider requirements
├── {service}.tf               # Service-specific resources (ecs.tf, rds.tf, etc.)
├── alarms.tf                  # CloudWatch alarms for all services
└── modules/                   # Reusable modules
    └── {service}/
        ├── main.tf
        ├── variables.tf
        ├── outputs.tf
        └── versions.tf
```

### Naming Convention Analysis
The repository uses consistent naming patterns that MUST be followed:

**Resource Names (in locals.tf):**
```hcl
# Pattern: {service}_{descriptor}_{suffix}
vpc_name = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-${random_string.setup.id}"
alb_name = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-ecsalb-${random_string.setup.id}"
```

**Character Limits by Service:**
- Short names (21 chars): VPC, ALB, ECS cluster names
- Medium names (38 chars): S3 buckets, RDS, ElastiCache, EFS
- Policy names (72 chars): IAM policies with descriptive names

## Rules for Adding New Components

### Rule 1: File Placement and Naming

**MUST:**
- Create a dedicated `{service}.tf` file for each new AWS service
- Follow the established file naming pattern
- Place service-specific resources in their dedicated file
- Add variables to `variables.tf` with proper validation
- Add computed names to `locals.tf` following naming patterns
- Add outputs to `outputs.tf` for important resource attributes

**Example: Adding AWS Lambda**
```hcl
# Create: lambda.tf
# Add to: variables.tf, locals.tf, outputs.tf
# Update: alarms.tf (if monitoring needed)
```

### Rule 2: Variable Structure Compliance

**MUST follow the established variable pattern:**
```hcl
variable "service_property_name" {
  description = "Clear, concise description of what this variable controls"
  type        = string|number|bool|list(type)|map(type)|object({...})
  default     = appropriate_default_value
  
  validation {
    condition     = validation_logic
    error_message = "Clear error message explaining the validation requirement"
  }
}
```

**Variable Naming Convention:**
- Pattern: `{service}_{property}_{descriptor}`
- Examples: `lambda_runtime`, `lambda_memory_size`, `lambda_timeout`

### Rule 3: Resource Naming in Locals

**MUST add resource names to locals.tf:**
```hcl
# Lambda example
lambda_function_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-lambda-${random_string.setup.id}"
lambda_role_name = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-lambda-role-${random_string.setup.id}"
```

### Rule 4: Security Group Integration

**MUST follow the security group pattern:**
```hcl
module "service_sg" {
  source = "./modules/vpn_sg" // v5.3.0

  name        = local.service_sg_name
  vpc_id      = module.vpc.vpc_id
  description = "Security Group for [service description]"

  ingress_with_source_security_group_id = [
    {
      rule                     = "all-all"
      source_security_group_id = module.other_sg.security_group_id
    }
  ]

  egress_rules = ["all-all"]

  tags = {
    Name = local.service_sg_name
  }
}
```

### Rule 5: IAM Policy Pattern

**MUST follow the IAM structure:**
```hcl
resource "aws_iam_role" "service_role" {
  name = local.service_role_name

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "service.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })

  tags = {
    Name = local.service_role_name
  }
}

resource "aws_iam_policy" "service_policy" {
  name = local.service_policy_name

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
```

### Rule 6: Tagging Compliance

**MUST apply standard tags:**
```hcl
tags = {
  Name = local.resource_name
}

# For resources needing additional tags:
tags = merge(local.tags, {
  Service     = "service-name"
  Environment = var.environment
  Component   = "component-name"
})
```

### Rule 7: Conditional Resource Creation

**MUST use consistent conditional patterns:**
```hcl
# Using count for simple conditional creation
resource "aws_service_resource" "main" {
  count = var.service_create ? 1 : 0
  
  name = local.service_name
  # ... configuration
}

# Using for_each for multiple similar resources
resource "aws_service_resource" "multiple" {
  for_each = local.service_configs

  name = local.service_names[each.key]
  # ... configuration
}
```

### Rule 8: Module Integration

**MUST use modules when available:**
```hcl
module "service_name" {
  source = "./modules/service_name" // v{version}

  # Required parameters first
  name   = local.service_name
  vpc_id = module.vpc.vpc_id

  # Optional parameters grouped logically
  instance_class = var.service_instance_class
  
  # Tags last
  tags = {
    Name = local.service_name
  }
}
```

## AWS Best Practices Integration

Based on AWS prescriptive guidance and industry best practices:

### Security Best Practices

**MUST implement:**
1. **Least Privilege IAM**: Only grant necessary permissions
2. **Encryption**: Enable encryption at rest and in transit
3. **Network Security**: Use security groups and NACLs appropriately
4. **Secrets Management**: Use AWS Secrets Manager or Parameter Store
5. **Logging**: Enable CloudTrail and service-specific logging

### Reliability Best Practices

**MUST implement:**
1. **Multi-AZ Deployment**: For production workloads
2. **Health Checks**: Implement comprehensive health monitoring
3. **Backup Strategy**: Automated backups for stateful resources
4. **Disaster Recovery**: Cross-region replication where appropriate

### Performance Best Practices

**MUST implement:**
1. **Right-sizing**: Choose appropriate instance types
2. **Auto Scaling**: Implement where applicable
3. **Caching**: Use ElastiCache for performance optimization
4. **CDN**: CloudFront for content delivery

### Cost Optimization

**MUST implement:**
1. **Resource Tagging**: Comprehensive cost allocation tags
2. **Lifecycle Policies**: S3 lifecycle management
3. **Reserved Instances**: For predictable workloads
4. **Monitoring**: Cost and usage monitoring

## Component Addition Workflow

### Step 1: Planning Phase

**Ask Kiro to:**
> "Analyze the poc-idp repository structure and help me plan adding [new-component]"

This will:
- Review existing patterns
- Identify integration points
- Suggest variable structure
- Recommend security considerations

### Step 2: Variable Definition

**Ask Kiro to:**
> "Create variables for [new-component] following poc-idp patterns"

This will:
- Generate properly structured variables
- Add validation rules
- Follow naming conventions
- Include default values

### Step 3: Resource Implementation

**Ask Kiro to:**
> "Implement [new-component] resources following poc-idp architecture"

This will:
- Create service-specific .tf file
- Add resource names to locals.tf
- Implement security groups
- Add IAM roles and policies

### Step 4: Integration and Testing

**Ask Kiro to:**
> "Integrate [new-component] with existing infrastructure and validate"

This will:
- Update tfvars templates
- Add monitoring and alarms
- Test with terraform plan
- Validate security compliance

## Common Component Patterns

### Adding Database Services (RDS, DynamoDB, DocumentDB)

**Required Elements:**
- Subnet groups for network isolation
- Security groups for access control
- Parameter groups for configuration
- Backup and maintenance windows
- Encryption configuration
- Monitoring and alarms

### Adding Compute Services (Lambda, ECS, EC2)

**Required Elements:**
- IAM roles and policies
- Security groups
- VPC configuration
- Logging configuration
- Monitoring and alarms
- Auto-scaling (where applicable)

### Adding Storage Services (S3, EFS, EBS)

**Required Elements:**
- Encryption configuration
- Access policies
- Lifecycle management
- Backup strategies
- Monitoring and alarms
- Cross-region replication (if needed)

### Adding Networking Services (VPN, NAT, Load Balancers)

**Required Elements:**
- Route table updates
- Security group rules
- Health checks
- SSL/TLS certificates
- Monitoring and alarms
- High availability configuration

## Validation Checklist

Before adding any new component, ensure:

**Architecture Compliance:**
- [ ] Follows file organization pattern
- [ ] Uses consistent naming conventions
- [ ] Implements proper tagging
- [ ] Includes appropriate outputs

**Security Compliance:**
- [ ] Implements least privilege IAM
- [ ] Enables encryption where applicable
- [ ] Uses security groups appropriately
- [ ] Follows network security best practices

**Operational Compliance:**
- [ ] Includes monitoring and alarms
- [ ] Implements backup strategies
- [ ] Supports high availability
- [ ] Includes proper documentation

**Cost Optimization:**
- [ ] Uses appropriate instance sizes
- [ ] Implements lifecycle policies
- [ ] Includes cost allocation tags
- [ ] Considers reserved capacity

## Example: Adding AWS Lambda

Here's a complete example of adding Lambda following all rules:

### 1. Variables (add to variables.tf)
```hcl
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
    environment = optional(map(string), {})
  }))
  default = {}
}
```

### 2. Locals (add to locals.tf)
```hcl
# Lambda naming
lambda_functions = {
  for name, config in var.lambda_functions : name => merge(config, {
    function_name = "${substr(var.application, 0, 38)}-${substr(var.environment, 0, 5)}-${name}-${random_string.setup.id}"
    role_name     = "${substr(var.application, 0, 21)}-${substr(var.environment, 0, 4)}-${name}-lambda-role-${random_string.setup.id}"
  })
}
```

### 3. Resources (create lambda.tf)
```hcl
# Lambda execution role
resource "aws_iam_role" "lambda_execution_role" {
  for_each = var.lambda_create ? local.lambda_functions : {}

  name = each.value.role_name

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })

  tags = {
    Name = each.value.role_name
  }
}

# Lambda function
resource "aws_lambda_function" "functions" {
  for_each = var.lambda_create ? local.lambda_functions : {}

  function_name = each.value.function_name
  role         = aws_iam_role.lambda_execution_role[each.key].arn
  handler      = each.value.handler
  runtime      = each.value.runtime
  memory_size  = each.value.memory_size
  timeout      = each.value.timeout

  environment {
    variables = each.value.environment
  }

  tags = {
    Name = each.value.function_name
  }
}
```

### 4. Outputs (add to outputs.tf)
```hcl
output "lambda_function_arns" {
  description = "ARNs of created Lambda functions"
  value = {
    for name, func in aws_lambda_function.functions : name => func.arn
  }
}
```

This example demonstrates complete compliance with all established patterns and best practices.