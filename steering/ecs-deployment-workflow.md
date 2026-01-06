# ECS Infrastructure Deployment Workflow

**CRITICAL: This workflow is for the IMPLEMENTATION PHASE only, after completing the spec workflow**

**Complete Workflow Order:**
1. **Requirements Phase** - Gather and document requirements (use this power's questioning logic)
2. **Design Phase** - Create comprehensive design document with architecture
3. **Tasks Phase** - Break down implementation into discrete coding tasks  
4. **Implementation Phase** - Execute tasks using this deployment workflow

This guide walks you through the implementation phase of deploying ECS infrastructure using the battle-tested poc-idp repository. The approach is:
1. **Clone the repository** (during task execution only)
2. **Use existing templates** (small/medium/large)
3. **Customize templates** with your values
4. **Deploy using proven patterns**

## Repository-First Implementation Philosophy

**Why we clone instead of generate (during implementation):**
- ✅ **Battle-tested**: Templates are production-proven across multiple deployments
- ✅ **Complete**: All components work together seamlessly
- ✅ **Maintained**: Regular updates and security patches
- ✅ **Documented**: Comprehensive README and examples
- ❌ **Never generate**: Creating new Terraform code introduces untested patterns

**The Golden Rule: Spec First, Clone Second, Customize Third, Deploy Fourth**

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
- **Small**: Development environments (<5 services)
- **Medium**: Staging environments (5-13 services)  
- **Large**: Production environments (13-23 services)

## Prerequisites

Before starting, ensure you have:
- AWS CLI configured with appropriate permissions
- Terraform installed (version 1.0+)
- An S3 bucket for Terraform state storage

## Step 1: Clone the Repository (IMPLEMENTATION PHASE ONLY)

**IMPORTANT: This step only happens during task execution, after completing the spec workflow**

**Prerequisites:**
- Requirements.md has been created and approved
- Design.md has been created and approved  
- Tasks.md has been created and approved
- You are now executing implementation tasks

I'll merge the poc-idp repository into your current working directory using the merge approach:

**Immediate Actions Required:**
```bash
# Step 1: Clone to temporary directory
git clone https://github.com/santyar0/poc-idp.git temp-poc-idp

# Step 2: Move git history to current directory
mv temp-poc-idp/.git ./

# Step 3: Move all files to current directory
mv temp-poc-idp/* ./

# Step 4: Clean up temporary directory
rm -rf temp-poc-idp
```

**What this does:**
- Clones the complete repository to a temporary directory
- Merges all repository content into your current working directory
- Provides access to all battle-tested templates in current workspace
- Maintains git history for version control
- Cleans up temporary files to keep workspace organized

**After merging, you'll have access to:**
- `example-small-setup.tfvars` - Development/MVP template
- `example-medium-setup.tfvars` - Staging/small production template  
- `example-large-setup.tfvars` - Large production template
- Complete Terraform modules and configurations
- `README.md` - Detailed deployment instructions

**Alternative Approach (if you prefer isolated directory):**
```bash
# Create dedicated directory
mkdir ecs-terraform

# Clone to dedicated directory
git clone https://github.com/santyar0/poc-idp.git ecs-terraform

# Work within dedicated directory
cd ecs-terraform
```

**IMPORTANT:** This repository cloning only happens during task execution, not during spec creation phases.

## Step 2: Collect Configuration Information

**Before creating any configuration files, collect all mandatory information through structured dialog:**

### Application Name Collection
**Ask using userInput tool:**
```json
{
  "question": "**What is your application name?** (This will be used for resource naming and must be suitable for AWS resource names)",
  "reason": "spec-entry-point-choice"
}
```

### Environment Selection
**Ask using userInput tool:**
```json
{
  "question": "**What environment is this for?**",
  "reason": "spec-entry-point-choice",
  "options": [
    {
      "title": "dev",
      "description": "Development environment for testing and development work"
    },
    {
      "title": "staging", 
      "description": "Staging environment for pre-production testing"
    },
    {
      "title": "prod",
      "description": "Production environment for live applications"
    }
  ]
}
```

### AWS Account ID Collection
**Ask using userInput tool:**
```json
{
  "question": "**What is your AWS Account ID?** (12-digit number, you can get this with 'aws sts get-caller-identity')",
  "reason": "spec-entry-point-choice"
}
```

### AWS Region Selection
**Ask using userInput tool:**
```json
{
  "question": "**Which AWS region do you want to deploy to?**",
  "reason": "spec-entry-point-choice",
  "options": [
    {
      "title": "us-east-1",
      "description": "US East (N. Virginia) - Most services available, lowest latency for US East Coast"
    },
    {
      "title": "us-west-2", 
      "description": "US West (Oregon) - Good for US West Coast, comprehensive service availability"
    },
    {
      "title": "eu-west-1",
      "description": "Europe (Ireland) - Primary EU region with full service availability"
    },
    {
      "title": "ap-southeast-1",
      "description": "Asia Pacific (Singapore) - Good for Southeast Asia"
    },
    {
      "title": "ap-northeast-1",
      "description": "Asia Pacific (Tokyo) - Good for Japan and Northeast Asia"
    },
    {
      "title": "Other region",
      "description": "Specify a different AWS region"
    }
  ]
}
```

### Configuration Summary
After collecting all information, display summary and confirm:
```
Configuration Summary:
- Application: [application-name]
- Environment: [environment]  
- AWS Account ID: [account-id]
- AWS Region: [region]

Generated file names:
- tfbackend: [application-name]-[account-id]-[region].tfbackend
- tfvars: [application-name]-[account-id]-[region].tfvars
```

## Step 3: Create Terraform Backend Configuration

**Using the collected configuration information:**

**Ask Kiro to:**
> "Create tfbackend file using collected configuration"

This will use the Filesystem MCP server to:
- Create `tfbackend/[application-name]-[account-id]-[region].tfbackend` file
- Use existing `*.tfbackend` files as reference
- Configure S3 backend without embedding credentials
- Reference: https://developer.hashicorp.com/terraform/language/backend/s3

**Backend file format:**
```hcl
bucket = "your-terraform-state-bucket"
key    = "path/to/your/terraform.tfstate"
region = "[region]"
```

**IMPORTANT:** 
- Do not specify any credentials in the tfbackend file
- Ensure S3 bucket exists with proper permissions
- User executing Terraform must have appropriate AWS permissions

## Step 4: Create Project Configuration (tfvars)

**Using the collected configuration information:**

**Ask Kiro to:**
> "Create tfvars file using collected configuration and [size] setup template"

This will use the Filesystem MCP server to:
- Create `tfvars/[application-name]-[account-id]-[region].tfvars` file
- Copy from appropriate template based on your setup size:
  - `tfvars/example-small-setup.tfvars` - For Small setup
  - `tfvars/example-medium-setup.tfvars` - For Medium setup  
  - `tfvars/example-large-setup.tfvars` - For Large setup
- Automatically populate mandatory variables with collected information

**Mandatory variables automatically populated:**
- `environment = "[environment]"` - From dialog selection
- `account_id = "[account-id]"` - From dialog input
- `region = "[region]"` - From dialog selection
- `application = "[application-name]"` - From dialog input

**Additional configuration options:**
- Check `README.md` file for complete list of configuration options
- Customize based on your specific requirements
- Review all remaining `CHANGE_ME_PLEASE` values in the template

## Step 5: Configure AWS Access and Verify Account

**Ask Kiro to:**
> "Verify AWS configuration matches collected information"

This will:
- Check AWS environment variables are configured
- Run `aws sts get-caller-identity` to confirm AWS account matches collected account ID
- Verify user has necessary permissions for infrastructure creation
- Validate S3 bucket access for Terraform state storage
- Confirm region configuration matches collected region

**AWS Configuration Requirements:**
- Configure AWS access via environment variables or AWS CLI
- Ensure user has permissions for ECS, VPC, RDS, S3, IAM, and other AWS services
- Verify you're working with the correct AWS account (must match collected account ID)
- Confirm AWS CLI region matches collected region

## Step 6: Initialize Terraform

**Ask Kiro to:**
> "Initialize Terraform with collected configuration"

This will run:
```bash
terraform init -backend-config=tfbackends/[application-name]-[account-id]-[region].tfbackend -reconfigure
```

**What this does:**
- Initializes Terraform with your specific backend configuration
- Downloads required providers and modules
- Configures remote state storage in S3
- Uses `-reconfigure` flag to ensure clean initialization

## Step 7: Review Terraform Plan

**Ask Kiro to:**
> "Show me the Terraform plan using collected configuration"

This will run:
```bash
terraform plan -var-file=tfvars/[application-name]-[account-id]-[region].tfvars
```

**What this does:**
- Shows all resources that will be created using your configuration
- Displays estimated costs and resource counts
- Highlights any configuration issues
- Uses the automatically populated mandatory variables
- Allows you to review before deployment

**If modifications needed:**
- Update your tfvars file based on plan review
- Repeat the plan step to verify changes
- Continue iterating until plan looks correct

## Step 8: Deploy Infrastructure

**Ask Kiro to:**
> "Deploy my ECS infrastructure using collected configuration"

This will run:
```bash
terraform apply -var-file=tfvars/[application-name]-[account-id]-[region].tfvars
```

**What this does:**
- Executes the Terraform plan with confirmation
- Creates all AWS resources as defined in your configuration
- Uses the automatically populated configuration values
- Monitors deployment progress with real-time status updates
- Displays final outputs including endpoints and resource details

## Step 9: Verify Deployment

After deployment, Kiro will automatically:
- Validate ECS cluster is running and healthy
- Check Application Load Balancer health and status
- Verify database connectivity (if RDS/DocumentDB is configured)
- Display all important endpoints, URLs, and connection details
- Provide next steps for application deployment
- Show resource names using your application name and environment

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