# Terraform Execution Steps

This guide provides detailed instructions for executing Terraform commands using MCP server integration for safer and more intelligent infrastructure deployment.

## Command Sequence Overview

The standard Terraform workflow with MCP enhancement:
1. **Initialize** - MCP validates backend and downloads providers
2. **Plan** - MCP analyzes and explains changes
3. **Apply** - MCP monitors execution and validates results
4. **Output** - MCP formats and explains deployment information

## Step 1: Initialize Terraform

**Ask Kiro to:**
> "Initialize Terraform for my project with backend configuration"

### What MCP Does
- **Terraform MCP**: Validates backend configuration syntax and compatibility
- **AWS MCP**: Checks S3 bucket accessibility and permissions
- **AWS MCP**: Verifies DynamoDB table for state locking
- **Terraform MCP**: Downloads and validates required providers
- **AWS Docs MCP**: References latest Terraform AWS provider best practices
- Confirms workspace is ready for operations

### Expected MCP Response
```
✅ Backend configuration validated against AWS best practices
✅ S3 bucket accessible with proper permissions and versioning
✅ DynamoDB table ready for state locking with correct schema
✅ AWS provider v5.x.x downloaded and configured
✅ Terraform workspace initialized successfully
✅ Best practices compliance verified
```

### MCP Troubleshooting
If initialization fails, MCP will:
- Identify specific permission issues
- Suggest IAM policy corrections
- Offer to create missing resources
- Provide step-by-step remediation

## Step 2: Plan Changes

**Ask Kiro to:**
> "Show me the Terraform plan for my infrastructure deployment"

### What MCP Does
- **Terraform MCP**: Analyzes current infrastructure state and dependencies
- **AWS MCP**: Compares with desired configuration and validates resources
- **AWS Docs MCP**: References AWS service limits and best practices
- **GitHub MCP**: Compares against poc-idp repository patterns
- Calculates resource dependencies and deployment order
- Estimates costs and deployment time using AWS pricing data
- Identifies potential issues, conflicts, or optimization opportunities

### Enhanced Plan Analysis
MCP provides intelligent insights with AWS best practices integration:

**Resource Analysis:**
```
📊 Plan Summary:
  • 15 resources to create
  • 0 resources to modify  
  • 0 resources to destroy

💰 Estimated Monthly Cost: $180-220 (based on current AWS pricing)
⏱️  Estimated Deployment Time: 12-15 minutes
🏗️  Deployment Order: VPC → Security Groups → RDS → ECS → ALB

🔍 Key Resources:
  ✅ ECS Cluster: myapp-prod-cluster (follows poc-idp naming)
  ✅ RDS Cluster: 2 instances (db.t4g.medium) - appropriate for medium setup
  ✅ ALB: myapp-prod-alb with health checks configured
  ⚠️  ElastiCache: Single AZ (consider multi-AZ for prod per AWS best practices)
  
📋 AWS Best Practices Compliance:
  ✅ Encryption enabled for all data stores
  ✅ IAM roles follow least privilege principle
  ✅ Security groups use minimal required access
  ⚠️  Consider enabling VPC Flow Logs for security monitoring
```

### MCP Validation Checks
- **Security**: Resource naming conflicts, security group rule analysis
- **Cost**: Cost optimization suggestions, right-sizing recommendations  
- **Reliability**: High availability recommendations, backup strategies
- **Performance**: Performance optimization opportunities
- **Compliance**: AWS Well-Architected Framework alignment

## Step 3: Apply Changes

**Ask Kiro to:**
> "Deploy my ECS infrastructure with monitoring"

### What MCP Does
- Executes Terraform apply with real-time monitoring
- Tracks resource creation progress
- Validates each resource after creation
- Provides status updates and ETA
- Handles errors with intelligent recovery

### Real-Time Monitoring
```
🚀 Deployment Progress:

✅ VPC and Networking (2/2) - 30s
✅ Security Groups (3/3) - 45s  
🔄 RDS Cluster (1/2) - Creating... ETA: 8 min
⏳ ECS Cluster - Waiting for RDS...
⏳ Load Balancer - Pending...

Current Status: 40% complete (5/12 resources)
```

### MCP Error Handling
If deployment fails, MCP will:
- Identify the root cause
- Suggest immediate fixes
- Offer rollback options
- Provide recovery commands
- Update team on status

## Step 4: Validate Deployment

**Ask Kiro to:**
> "Validate my deployed infrastructure"

### What MCP Does
- Tests ECS cluster health
- Validates load balancer targets
- Checks database connectivity
- Verifies security group rules
- Confirms DNS resolution

### Validation Report
```
🔍 Infrastructure Validation:

✅ ECS Cluster: Healthy (2/2 services running)
✅ Load Balancer: Healthy targets (100%)
✅ RDS Cluster: Available (2/2 instances)
✅ Security Groups: Properly configured
✅ DNS: Resolving correctly

🌐 Endpoints:
  • Application: https://myapp.example.com
  • Health Check: https://myapp.example.com/health
  • Database: myapp-db.cluster-xyz.us-east-1.rds.amazonaws.com
```

## Advanced MCP Features

### Intelligent Plan Analysis

**Ask Kiro to:**
> "Analyze my Terraform plan for security and cost optimization"

MCP provides:
- Security vulnerability scanning
- Cost optimization recommendations
- Performance improvement suggestions
- Compliance checking
- Best practice validation

### Targeted Operations

**Ask Kiro to:**
> "Update only my ECS service configuration"

MCP will:
- Identify affected resources
- Show minimal change plan
- Execute targeted deployment
- Validate specific components

### Drift Detection

**Ask Kiro to:**
> "Check for infrastructure drift"

MCP will:
- Compare actual vs. desired state
- Identify manual changes
- Suggest import or correction
- Update Terraform state if needed

## MCP-Enhanced Error Recovery

### Resource Creation Failures
MCP automatically:
- Analyzes AWS error messages
- Suggests specific solutions
- Offers alternative configurations
- Provides retry strategies

### State Management Issues
MCP can:
- Detect state corruption
- Restore from S3 backups
- Resolve state conflicts
- Import existing resources

### Permission Problems
MCP will:
- Analyze IAM policies
- Suggest required permissions
- Generate policy documents
- Test permission changes

## Best Practices with MCP

### Before Every Deployment
1. **Let MCP validate** configuration and permissions
2. **Review MCP analysis** of plan and costs
3. **Check MCP recommendations** for optimization
4. **Confirm MCP security** scan results

### During Deployment
1. **Monitor MCP progress** updates
2. **Trust MCP error handling** for recovery
3. **Use MCP insights** for troubleshooting
4. **Follow MCP guidance** for manual interventions

### After Deployment
1. **Review MCP validation** results
2. **Implement MCP suggestions** for improvements
3. **Use MCP monitoring** for ongoing health checks
4. **Document MCP findings** for team knowledge

## Common MCP Workflows

### Initial Deployment
**You:** "Deploy ECS infrastructure for myapp-prod"

**MCP Process:**
1. Validates all configurations
2. Analyzes plan with cost estimates
3. Executes deployment with monitoring
4. Validates all resources
5. Provides complete status report

### Configuration Updates
**You:** "Update my ECS service to 3 replicas"

**MCP Process:**
1. Identifies minimal changes needed
2. Shows impact analysis
3. Executes targeted update
4. Validates service scaling
5. Confirms health checks pass

### Troubleshooting
**You:** "My application isn't accessible"

**MCP Process:**
1. Runs comprehensive health checks
2. Identifies connectivity issues
3. Suggests specific fixes
4. Offers to implement solutions
5. Validates fixes work

## MCP Integration Benefits

### Intelligence
- Automated analysis and recommendations
- Predictive issue detection
- Smart error recovery
- Cost and performance optimization

### Safety
- Pre-deployment validation
- Real-time monitoring
- Automatic rollback capabilities
- State corruption prevention

### Efficiency
- Faster troubleshooting
- Automated best practices
- Intelligent resource management
- Streamlined workflows

## Next Steps with MCP

After successful deployment:
1. **Use MCP monitoring** for ongoing health checks
2. **Leverage MCP insights** for optimization
3. **Set up MCP alerts** for issues
4. **Document MCP learnings** for team

**Need help with specific issues?** See: `#troubleshooting-guide`

## Example MCP Conversation

**You:** "I need to deploy my staging environment"

**Kiro with MCP:**
1. "Let me validate your configuration first..."
2. "I found a few optimization opportunities..."
3. "Here's your deployment plan with cost estimates..."
4. "Deploying now with real-time monitoring..."
5. "✅ Deployment complete! All services healthy."
6. "Here are some post-deployment recommendations..."

This intelligent, guided approach makes Terraform deployment much safer and more efficient than manual command execution.