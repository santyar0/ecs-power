# ECS Infrastructure Deployment Workflow

This guide walks you through deploying ECS infrastructure using the battle-tested poc-idp repository. Instead of generating new code, we'll use the existing templates and follow the proven README workflow.

## AI-DLC Integration

For complex deployments or when you need structured planning, consider using the **AI-Driven Development Lifecycle (AI-DLC)** approach:

**Start with AI-DLC:** "Using AI-DLC, I want to deploy ECS infrastructure for [your specific use case]"

This activates the three-phase AI-DLC workflow:
1. **Inception**: Requirements analysis and architecture design
2. **Construction**: Detailed design and implementation  
3. **Operations**: Deployment, monitoring, and documentation

**When to use AI-DLC:**
- Complex production deployments
- Multi-service architectures
- Compliance or security requirements
- Team collaboration needs
- Unclear requirements

**For detailed AI-DLC guidance:** See `#ai-dlc-infrastructure-workflow`

## Standard Deployment Overview

The poc-idp repository provides three pre-configured setup sizes:
- **Small**: Development environments ($50-100/month)
- **Medium**: Staging environments ($200-400/month)  
- **Large**: Production environments ($800-1500/month)

## Prerequisites

Before starting, ensure you have:
- AWS CLI configured with appropriate permissions
- Terraform installed (version 1.0+)
- An S3 bucket for Terraform state storage

## Step 1: Clone the Repository

I'll clone the poc-idp repository for you using the Git MCP server:

**Ask Kiro to run:**
> "Clone the poc-idp repository from https://github.com/santyar0/poc-idp.git"

This will use the Git MCP server to:
- Clone the repository to your local workspace
- Verify the repository structure
- List available template files

## Step 2: Choose Your Setup Size

Review the available templates that will be shown after cloning:

- `example-small-setup.tfvars` - Minimal resources for development
- `example-medium-setup.tfvars` - Moderate resources for staging
- `example-large-setup.tfvars` - High-availability for production

**Need help choosing?** Use the template selection guide: `#template-selection-guide`

## Step 3: Create Your Project Configuration

**Ask Kiro to:**
> "Copy the [size] template and customize it for my project [project-name]"

This will use the Filesystem MCP server to:
- Copy your chosen template (small/medium/large)
- Create a project-specific tfvars file
- Show you all the `CHANGE_ME_PLEASE` values that need customization

**Then provide your values:**
- `account_id` - Your AWS account ID
- `region` - Your preferred AWS region  
- `application` - Your application name
- `environment` - Environment name (dev, staging, prod)
- Any other placeholders specific to your setup size

## Step 4: Setup Terraform Backend

**Ask Kiro to:**
> "Help me set up S3 backend for Terraform state with bucket [bucket-name]"

This will use the AWS MCP server to:
- Validate your S3 bucket exists and is accessible
- Check if versioning is enabled (recommended)
- Create the tfbackend configuration file
- Verify DynamoDB table for state locking exists

**Need detailed S3 setup help?** See: `#s3-backend-setup`

## Step 5: Initialize Terraform

**Ask Kiro to:**
> "Initialize Terraform with my backend configuration"

This will:
- Run `terraform init` with your backend config
- Download required providers
- Configure state storage
- Verify initialization was successful

## Step 6: Review the Plan

**Ask Kiro to:**
> "Show me the Terraform plan for my infrastructure"

This will:
- Run `terraform plan` with your tfvars file
- Display what resources will be created
- Show estimated costs and resource counts
- Highlight any potential issues

## Step 7: Deploy Infrastructure

**Ask Kiro to:**
> "Deploy my ECS infrastructure"

This will:
- Execute `terraform apply` with confirmation
- Monitor the deployment progress
- Show real-time status updates
- Display final outputs and endpoints

## Step 8: Verify Deployment

After deployment, Kiro will automatically:
- Validate ECS cluster is running
- Check load balancer health
- Verify database connectivity (if applicable)
- Display all important endpoints and URLs

## MCP-Enhanced Features

With MCP servers, this power provides:

### Git Operations
- Intelligent repository cloning
- Branch management
- Template discovery
- Version tracking

### AWS Integration  
- Real-time resource validation
- S3 bucket verification
- IAM permission checking
- Cost estimation

### File Management
- Smart template customization
- Configuration validation
- Backup creation
- Diff visualization

## Next Steps

1. **Configure your applications** to use the deployed infrastructure
2. **Set up monitoring** using the CloudWatch alarms created
3. **Configure DNS** if using custom domains
4. **Set up CI/CD pipelines** to deploy your applications
5. **Customize infrastructure** with additional components using `#infrastructure-customization-workflow`
6. **Follow best practices** for ongoing maintenance using `#aws-terraform-best-practices`
7. **Extend infrastructure** with new components using `#component-extension-guide`

## AI-DLC Operations Phase

For production deployments, consider completing the AI-DLC Operations phase:

**Activate Operations Phase:** "Using AI-DLC, I want to complete the Operations phase for my deployed infrastructure"

This includes:
- **Monitoring and Observability**: CloudWatch dashboards and alerting
- **Documentation and Knowledge Transfer**: Operational runbooks and procedures
- **Performance Optimization**: Baseline establishment and tuning

**For detailed Operations guidance:** See `#ai-dlc-infrastructure-workflow`

## Need Help?

- **Deployment issues?** See: `#troubleshooting-guide`
- **Terraform execution problems?** See: `#terraform-execution-steps`
- **Template questions?** See: `#template-selection-guide`
- **Adding new components?** See: `#infrastructure-customization-workflow`
- **Extension patterns?** See: `#component-extension-guide`
- **Best practices guidance?** See: `#aws-terraform-best-practices`

## Example Conversation Flow

Here's how a typical deployment conversation might look:

**You:** "I want to deploy ECS infrastructure for my staging environment"

**Kiro:** *Uses this workflow to guide you through:*
1. Cloning poc-idp repository
2. Recommending medium setup for staging
3. Customizing template with your values
4. Setting up S3 backend
5. Running Terraform commands
6. Validating deployment

**Result:** Production-ready ECS infrastructure deployed with expert guidance and automated validation.

## Important Notes

- This deployment follows the exact workflow from the poc-idp repository README
- All infrastructure patterns are battle-tested and production-ready
- MCP servers provide intelligent automation while maintaining flexibility
- State is stored in S3 with encryption and locking for team collaboration