# Infrastructure Customization Workflow

This guide provides a comprehensive workflow for customizing existing poc-idp infrastructure with new components, following established patterns and best practices.

## AI-DLC Integration for Complex Customizations

For complex infrastructure customizations, consider using the **AI-Driven Development Lifecycle (AI-DLC)** approach:

**Start with AI-DLC:** "Using AI-DLC, I want to add [specific component] to my existing infrastructure"

**When to use AI-DLC for customization:**
- Multi-service integrations (API Gateway + Lambda + DynamoDB)
- Complex data pipelines (Kinesis + Lambda + S3)
- Service mesh implementations
- Compliance-heavy additions
- Cross-team collaboration needs

**AI-DLC provides:**
- **Inception Phase**: Requirements analysis and architecture design
- **Construction Phase**: Detailed implementation planning and execution
- **Operations Phase**: Deployment, monitoring, and documentation

**For detailed AI-DLC guidance:** See `#ai-dlc-infrastructure-workflow`

## Standard Customization Overview

The customization workflow enables you to extend the battle-tested poc-idp infrastructure with additional AWS services while maintaining consistency, security, and operational excellence.

## Customization Philosophy

### Core Principles
1. **Extend, Don't Replace**: Build upon existing patterns rather than recreating
2. **Consistency First**: Follow established naming and structural conventions
3. **Security by Design**: Implement security controls from the start
4. **Operational Excellence**: Include monitoring, logging, and maintenance from day one

### When to Customize

**Good Candidates for Extension:**
- Adding complementary services (Lambda, API Gateway, SQS, SNS)
- Enhancing existing services (adding ElastiSearch, Redis clusters)
- Implementing additional storage solutions (EFS, FSx)
- Adding monitoring and observability tools
- Implementing CI/CD pipeline components

**Consider Alternatives:**
- Core networking changes (use existing VPC patterns)
- Security model changes (extend existing IAM patterns)
- Fundamental architecture changes (consider separate deployment)

## Pre-Customization Analysis

### Step 1: Repository Structure Analysis

**Ask Kiro to:**
> "Analyze the poc-idp repository structure and identify integration points for [new-component]"

This will provide:
- Current architecture overview
- Existing service dependencies
- Integration opportunities
- Potential conflicts or considerations

### Step 2: Impact Assessment

**Ask Kiro to:**
> "Assess the impact of adding [new-component] to my current setup size [small/medium/large]"

This will analyze:
- Resource requirements and costs
- Performance implications
- Security considerations
- Operational complexity

### Step 3: Dependency Mapping

**Ask Kiro to:**
> "Map dependencies between [new-component] and existing infrastructure"

This identifies:
- Required existing resources
- New resource dependencies
- Network connectivity requirements
- IAM permission needs

## Customization Workflow

### Phase 1: Planning and Design

#### 1.1 Component Analysis
**Ask Kiro to:**
> "Research AWS [service-name] best practices and integration patterns"

**Expected Analysis:**
- Service capabilities and limitations
- Integration patterns with ECS/RDS/ALB
- Security requirements
- Cost implications
- Operational considerations

#### 1.2 Architecture Design
**Ask Kiro to:**
> "Design the integration of [new-component] with my existing infrastructure"

**Design Output:**
- Architecture diagram
- Network topology changes
- Security group modifications
- IAM role requirements
- Monitoring strategy

#### 1.3 Variable Planning
**Ask Kiro to:**
> "Plan the variable structure for [new-component] following poc-idp patterns"

**Variable Design:**
- Input variables with validation
- Local value computations
- Conditional logic for different setup sizes
- Default values and examples

### Phase 2: Implementation

#### 2.1 Variable Implementation
**Ask Kiro to:**
> "Implement variables for [new-component] in my tfvars templates"

**Implementation Steps:**
1. Add variables to `variables.tf` with proper validation
2. Update local values in `locals.tf` for naming
3. Add conditional logic for setup size differences
4. Update all three template files (small, medium, large)

#### 2.2 Resource Implementation
**Ask Kiro to:**
> "Implement [new-component] resources following poc-idp architecture patterns"

**Resource Creation:**
1. Create dedicated `{service}.tf` file
2. Implement main resources with proper configuration
3. Add supporting resources (security groups, IAM roles)
4. Include monitoring and logging resources
5. Add outputs for integration points

#### 2.3 Security Implementation
**Ask Kiro to:**
> "Implement security controls for [new-component] following best practices"

**Security Elements:**
1. IAM roles with least privilege
2. Security groups with minimal access
3. Encryption configuration
4. Network isolation
5. Audit logging

#### 2.4 Monitoring Implementation
**Ask Kiro to:**
> "Add monitoring and alerting for [new-component]"

**Monitoring Elements:**
1. CloudWatch metrics and alarms
2. Log group configuration
3. SNS notification integration
4. Dashboard updates
5. Health check implementation

### Phase 3: Integration and Testing

#### 3.1 Configuration Integration
**Ask Kiro to:**
> "Integrate [new-component] configuration with existing setup sizes"

**Integration Tasks:**
1. Update tfvars templates with new variables
2. Add component-specific configurations per setup size
3. Update documentation and comments
4. Validate configuration syntax

#### 3.2 Terraform Validation
**Ask Kiro to:**
> "Validate the Terraform configuration for [new-component]"

**Validation Steps:**
1. Run `terraform fmt` for formatting
2. Execute `terraform validate` for syntax
3. Perform `terraform plan` for resource preview
4. Review plan output for correctness

#### 3.3 Security Validation
**Ask Kiro to:**
> "Perform security validation for [new-component] implementation"

**Security Checks:**
1. IAM policy analysis
2. Security group rule review
3. Encryption configuration verification
4. Network access validation
5. Compliance checking

### Phase 4: Deployment and Verification

#### 4.1 Staged Deployment
**Ask Kiro to:**
> "Deploy [new-component] using staged approach"

**Deployment Strategy:**
1. Deploy to development environment first
2. Validate functionality and integration
3. Deploy to staging for full testing
4. Deploy to production with monitoring

#### 4.2 Integration Testing
**Ask Kiro to:**
> "Test integration between [new-component] and existing services"

**Testing Areas:**
1. Network connectivity
2. Service communication
3. Authentication and authorization
4. Data flow validation
5. Performance impact

#### 4.3 Operational Validation
**Ask Kiro to:**
> "Validate operational aspects of [new-component]"

**Operational Checks:**
1. Monitoring and alerting functionality
2. Backup and recovery procedures
3. Scaling behavior
4. Log aggregation
5. Cost tracking

## Common Customization Patterns

### Adding Serverless Components

**Example: AWS Lambda Integration**

**Planning Questions:**
- What triggers will invoke the Lambda functions?
- How will Lambda integrate with ECS services?
- What data will Lambda access (RDS, ElastiCache)?
- How will errors be handled and monitored?

**Implementation Pattern:**
```hcl
# Lambda-specific variables
variable "lambda_functions" {
  description = "Map of Lambda functions to create"
  type = map(object({
    runtime         = string
    handler         = string
    memory_size     = optional(number, 128)
    timeout         = optional(number, 30)
    environment     = optional(map(string), {})
    vpc_config      = optional(bool, false)
    database_access = optional(bool, false)
    cache_access    = optional(bool, false)
  }))
  default = {}
}

# Setup size specific configurations
locals {
  lambda_configs = {
    small = {
      default_memory = 128
      default_timeout = 30
      vpc_enabled = false
    }
    medium = {
      default_memory = 256
      default_timeout = 60
      vpc_enabled = true
    }
    large = {
      default_memory = 512
      default_timeout = 300
      vpc_enabled = true
    }
  }
}
```

### Adding Storage Solutions

**Example: Amazon EFS Integration**

**Planning Questions:**
- Which services need shared file storage?
- What are the performance requirements?
- How will access be controlled?
- What backup strategy is needed?

**Implementation Pattern:**
```hcl
# EFS-specific variables
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
    error_message = "Performance mode must be generalPurpose or maxIO."
  }
}

# Setup size specific EFS configurations
locals {
  efs_configs = {
    small = {
      throughput_mode = "provisioned"
      provisioned_throughput = 100
      backup_enabled = false
    }
    medium = {
      throughput_mode = "provisioned"
      provisioned_throughput = 500
      backup_enabled = true
    }
    large = {
      throughput_mode = "provisioned"
      provisioned_throughput = 1000
      backup_enabled = true
    }
  }
}
```

### Adding Message Queuing

**Example: Amazon SQS/SNS Integration**

**Planning Questions:**
- What messaging patterns are needed?
- How will messages be processed?
- What are the durability requirements?
- How will dead letter queues be handled?

**Implementation Pattern:**
```hcl
# Messaging variables
variable "messaging_create" {
  description = "Whether to create messaging resources"
  type        = bool
  default     = false
}

variable "queues" {
  description = "Map of SQS queues to create"
  type = map(object({
    visibility_timeout_seconds = optional(number, 30)
    message_retention_seconds  = optional(number, 1209600)
    max_receive_count         = optional(number, 3)
    delay_seconds            = optional(number, 0)
    fifo_queue              = optional(bool, false)
  }))
  default = {}
}

# Setup size specific messaging configurations
locals {
  messaging_configs = {
    small = {
      default_retention = 86400      # 1 day
      dlq_enabled = false
    }
    medium = {
      default_retention = 604800     # 7 days
      dlq_enabled = true
    }
    large = {
      default_retention = 1209600    # 14 days
      dlq_enabled = true
    }
  }
}
```

## Advanced Customization Scenarios

### Multi-Service Integration

**Scenario: Adding API Gateway + Lambda + DynamoDB**

**Ask Kiro to:**
> "Design a serverless API integration with API Gateway, Lambda, and DynamoDB"

**Integration Considerations:**
1. API Gateway integration with existing ALB
2. Lambda function VPC configuration for RDS access
3. DynamoDB table design and access patterns
4. Authentication and authorization flow
5. Monitoring and logging across all services

### Data Pipeline Integration

**Scenario: Adding Kinesis + Lambda + S3 for data processing**

**Ask Kiro to:**
> "Design a data processing pipeline with Kinesis, Lambda, and S3"

**Pipeline Considerations:**
1. Data ingestion from ECS applications
2. Stream processing with Lambda
3. Data storage in S3 with lifecycle policies
4. Integration with existing monitoring
5. Error handling and retry mechanisms

### Microservices Extension

**Scenario: Adding additional ECS services with service mesh**

**Ask Kiro to:**
> "Design additional microservices with service mesh integration"

**Service Mesh Considerations:**
1. Service discovery integration
2. Load balancing and traffic routing
3. Security policies and mTLS
4. Observability and tracing
5. Circuit breaker patterns

## Customization Best Practices

### 1. Incremental Development
- Start with minimal viable implementation
- Add features incrementally
- Test each addition thoroughly
- Document changes and decisions

### 2. Configuration Management
- Use setup size patterns consistently
- Provide sensible defaults
- Include validation rules
- Document configuration options

### 3. Security Integration
- Follow existing security patterns
- Implement least privilege access
- Use existing KMS keys where possible
- Integrate with existing monitoring

### 4. Operational Integration
- Use existing log groups and patterns
- Integrate with existing alerting
- Follow existing backup strategies
- Use consistent tagging

### 5. Cost Management
- Consider cost implications of additions
- Use appropriate instance sizes
- Implement lifecycle policies
- Monitor and optimize regularly

## Troubleshooting Customizations

### Common Issues

**1. Resource Naming Conflicts**
- **Symptom**: Resources already exist errors
- **Solution**: Check naming patterns and uniqueness
- **Prevention**: Use consistent local value patterns

**2. Security Group Dependencies**
- **Symptom**: Circular dependency errors
- **Solution**: Review security group references
- **Prevention**: Plan security group architecture

**3. IAM Permission Issues**
- **Symptom**: Access denied errors during deployment
- **Solution**: Review and update IAM policies
- **Prevention**: Follow least privilege patterns

**4. Network Connectivity Problems**
- **Symptom**: Services cannot communicate
- **Solution**: Review security groups and routing
- **Prevention**: Plan network architecture carefully

### Debugging Workflow

**Ask Kiro to:**
> "Debug [specific-issue] with my [new-component] customization"

**Debugging Steps:**
1. Analyze Terraform plan output
2. Review AWS console for resource states
3. Check CloudWatch logs for errors
4. Validate security group rules
5. Test network connectivity
6. Review IAM permissions

## Documentation and Maintenance

### Documentation Requirements
1. **Architecture Documentation**: Update diagrams and descriptions
2. **Configuration Guide**: Document new variables and options
3. **Operational Procedures**: Update runbooks and procedures
4. **Troubleshooting Guide**: Add component-specific troubleshooting

### Maintenance Considerations
1. **Version Updates**: Keep component versions current
2. **Security Patches**: Apply security updates promptly
3. **Performance Monitoring**: Monitor and optimize performance
4. **Cost Optimization**: Regular cost reviews and optimization

This comprehensive workflow ensures successful customization while maintaining the reliability and security of the poc-idp infrastructure foundation.