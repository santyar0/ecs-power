---
name: "ecs-infrastructure-power"
displayName: "ECS Infrastructure Deployment"
description: "Deploy and manage ECS infrastructure using poc-idp repository patterns with AI-DLC methodology and MCP automation"
keywords: ["ecs", "terraform", "aws", "infrastructure", "deployment", "poc-idp", "ai-dlc", "docker", "containers", "microservices", "microservice", "architecture", "fargate", "cluster", "setup", "deploy"]
mcpServers: ["awslabs.terraform-mcp-server", "filesystem", "git"]
---

# Onboarding

## Step 1: Validate tools work

Before using ECS Infrastructure Power, ensure the following are installed and configured:

- **AWS CLI**: Configure with appropriate permissions for ECS, VPC, RDS, and other AWS services
  - Verify with: `aws sts get-caller-identity`
  - **CRITICAL**: If AWS CLI is not configured or lacks permissions, DO NOT proceed with infrastructure deployment.

- **Terraform**: Install Terraform version 1.0 or higher
  - Verify with: `terraform --version`
  - **CRITICAL**: If Terraform is not installed, infrastructure deployment will fail.

- **S3 Bucket**: Ensure you have an S3 bucket for Terraform state storage
  - Bucket should have versioning enabled for state history
  - **CRITICAL**: Without proper state storage, team collaboration will be impossible.

- **Git**: Ensure Git is installed for repository operations
  - Verify with: `git --version`

## Step 2: Smart Environment Sizing Logic

**CRITICAL: NEVER assume setup size - ALWAYS ask questions first, even for simple requests**

**IMPORTANT: ALWAYS follow the complete spec workflow before implementation:**
1. **Requirements Phase**: Gather and document requirements
2. **Design Phase**: Create comprehensive design document  
3. **Tasks Phase**: Break down implementation into discrete tasks
4. **Implementation Phase**: Execute tasks (this is where repository cloning happens)

### Mandatory First Response:
When user mentions ECS/AWS infrastructure without specifying details, ALWAYS ask using the userInput tool with these exact parameters:

```json
{
  "question": "**I'll help you set up ECS infrastructure! First, let's create a proper spec. To choose the right setup size, how many services do you plan to deploy?**",
  "reason": "spec-entry-point-choice",
  "options": [
    {
      "title": "1-4 services (Small Setup)",
      "description": "Development, MVP, or testing environments with basic monitoring and no HA",
      "recommended": true
    },
    {
      "title": "5-13 services (Medium Setup)", 
      "description": "Small production or staging environments with partial HA and backups"
    },
    {
      "title": "13-23 services (Large Setup)",
      "description": "Large production environments with full HA, comprehensive monitoring, and backups"
    },
    {
      "title": "I'm not sure yet",
      "description": "Help me determine the right size based on my requirements"
    }
  ]
}
```

**For Docker deployment requests without ECS specified:**
When user says "deploy docker to AWS" or similar, ALWAYS explain options first using userInput tool:

```json
{
  "question": "**I can help you deploy Docker containers to AWS! Let's start by creating a proper spec. Which approach interests you most?**",
  "reason": "spec-entry-point-choice", 
  "options": [
    {
      "title": "ECS (Elastic Container Service)",
      "description": "Fully managed container orchestration - recommended for most use cases",
      "recommended": true
    },
    {
      "title": "EKS (Elastic Kubernetes Service)",
      "description": "Managed Kubernetes for complex orchestration needs"
    },
    {
      "title": "EC2 with Docker",
      "description": "Direct Docker installation on virtual machines"
    },
    {
      "title": "AWS Fargate",
      "description": "Serverless containers (works with ECS/EKS)"
    }
  ]
}
```

**Only after user confirms ECS, then ask about service count using the service count question above.**

**CRITICAL: After gathering requirements, ALWAYS proceed through the complete spec workflow:**
1. **Create requirements.md** - Document all requirements with EARS patterns
2. **Create design.md** - Comprehensive design with architecture and correctness properties  
3. **Create tasks.md** - Implementation plan with discrete coding tasks
4. **Execute tasks** - This is where repository cloning and implementation happens

**DURING IMPLEMENTATION PHASE - Collect Configuration Information:**
When executing implementation tasks, ALWAYS collect all mandatory configuration through structured dialog:

**Step 1: Collect Application Name**
```json
{
  "question": "**What is your application name?** (This will be used for resource naming)",
  "reason": "spec-entry-point-choice"
}
```

**Step 2: Collect Environment**
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

**Step 3: Collect AWS Account ID**
```json
{
  "question": "**What is your AWS Account ID?** (12-digit number, you can get this with 'aws sts get-caller-identity')",
  "reason": "spec-entry-point-choice"
}
```

**Step 4: Collect AWS Region**
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
      "title": "Other region",
      "description": "Specify a different AWS region"
    }
  ]
}
```

**Configuration Summary:**
After collecting all information, display summary:
```
Configuration Summary:
- Application: [application-name]
- Environment: [environment]
- AWS Account ID: [account-id]
- AWS Region: [region]

File names will be:
- tfbackend: [application-name]-[account-id]-[region].tfbackend
- tfvars: [application-name]-[account-id]-[region].tfvars
```

### Intelligent Questioning Strategy:
Use the userInput tool with predefined options to make selection easy:

1. **Service Count Question** (ALWAYS ask this first for ECS requests):
   - Use the service count userInput question above with 4 clear options
   - If user selects "I'm not sure yet", then ask follow-up questions

2. **If service count is unclear, ask about environment using userInput:**
   ```json
   {
     "question": "**What type of environment is this for?**",
     "reason": "spec-entry-point-choice",
     "options": [
       {
         "title": "Development/Testing",
         "description": "For development, MVP, or testing purposes",
         "recommended": false
       },
       {
         "title": "Staging", 
         "description": "Pre-production environment for testing"
       },
       {
         "title": "Production",
         "description": "Live production environment serving real users"
       }
     ]
   }
   ```

3. **If still unclear, ask about HA requirements using userInput:**
   ```json
   {
     "question": "**Do you need high availability and comprehensive backups?**",
     "reason": "spec-entry-point-choice",
     "options": [
       {
         "title": "Basic setup is fine",
         "description": "Single AZ, basic monitoring, no automated backups"
       },
       {
         "title": "Some reliability needed",
         "description": "Partial HA, basic backups, moderate monitoring"
       },
       {
         "title": "Full production reliability",
         "description": "Multi-AZ, comprehensive backups, full monitoring suite"
       }
     ]
   }
   ```

### NEVER Assume Rules:
- **DON'T assume** service count from vague requests
- **DON'T assume** environment type without asking
- **DON'T assume** setup size from keywords alone
- **DON'T assume** ECS is wanted for generic "docker to AWS" requests
- **ALWAYS USE** userInput tool with predefined options for better UX
- **ALWAYS FOLLOW** the complete spec workflow: Requirements → Design → Tasks → Implementation
- **NEVER SKIP** spec stages - they are mandatory for proper planning
- **ALWAYS ASK** at least one clarifying question before proceeding
- **ALWAYS CONFIRM** the chosen approach before starting spec creation

### Quick Decision Rules (ONLY after asking):
- **Definitive indicators for Small**: "development", "MVP", "testing", "prototype", <5 services
- **Definitive indicators for Medium**: "staging", "small production", 5-13 services, "some HA"
- **Definitive indicators for Large**: "production", "high availability", "enterprise", 13+ services

### Activation Keywords:
Power activates on: `ecs`, `microservices`, `microservice architecture`, `aws docker deploy`, `ecs setup`, `fargate`, `container deployment`, `aws infrastructure`, `terraform aws`

### Setup Sizing Table:
| Setup Type | Number of Services | Use Case |
|------------|-------------------|----------|
| **Small**  | <5 services       | Development, MVP, testing |
| **Medium** | 5-13 services     | Small production, staging |
| **Large**  | 13-23 services    | Large production, HA required |

### Detailed Setup Descriptions:

#### Small Setup
**Purpose**: Development or MVP environments with minimal services and basic monitoring.
**Out of scope**: High Availability (HA), backups, and automation.

**Infrastructure includes**:
- **Network**: Dedicated VPC, 3 public/private/DB subnets across AZs, single NAT gateway, bastion host
- **Database**: RDS Aurora PostgreSQL (1 instance, no MultiAZ) or DocumentDB (no MultiAZ)
- **Application**: ECS Cluster, up to 5 services (1 replica each), Fargate without autoscaling
- **Storage**: S3 buckets (no versioning), ECR
- **Public Layer**: ALB, Route53, CloudFront
- **Monitoring**: Basic ECS service availability alarms

#### Medium Setup  
**Purpose**: Small production environments with partial HA, backups, and monitoring for critical services.
**Includes Terraform IaC automation**.

**Infrastructure includes**:
- **Network**: Same as Small + private endpoints for EFS/S3
- **Database**: RDS Aurora PostgreSQL (2 instances RW+R, optional MultiAZ), ElastiCache (no MultiAZ)
- **Application**: ECS Cluster, up to 13 services (1 replica each), Fargate without autoscaling
- **Storage**: S3 buckets with versioning, EFS (30GB), ECR
- **Backups**: AWS Backup with tagged resources and vault configuration
- **Monitoring**: ECS availability, web error rate, web request time alarms

#### Large Setup
**Purpose**: Production environments with large service count, full HA, backups, and comprehensive monitoring.
**Includes full Terraform IaC automation**.

**Infrastructure includes**:
- **Network**: Full HA with NAT gateways in each AZ, Client VPN, private endpoints in each AZ
- **Database**: RDS Aurora PostgreSQL Serverless V2 (MultiAZ), ElastiCache (MultiAZ)
- **Application**: 1-2 ECS Clusters, up to 23 services, Fargate with service autoscaling
- **Storage**: S3 buckets with versioning, EFS (30GB), ECR
- **Backups**: Comprehensive AWS Backup for all tagged resources
- **Monitoring**: Full monitoring suite with ECS, web error rate, and request time alarms

### Environment Selection Rules:
- **Small Setup**: <5 services, development/MVP, no HA/backups, basic monitoring
- **Medium Setup**: 5-13 services, small production, partial HA, backups included
- **Large Setup**: 13-23 services, full production, complete HA, comprehensive monitoring

### Example Conversation:
```
User: "I want to deploy ECS infrastructure"

Agent: [Uses userInput tool with service count question showing 4 clickable options]

User: [Clicks "5-13 services (Medium Setup)"]

Agent: "Perfect! Based on 5-13 services, I'll help you create a comprehensive spec for Medium setup ECS infrastructure. Let's start with the requirements phase..."

[Agent then proceeds through:]
1. Requirements gathering and documentation
2. Design creation with architecture details
3. Task breakdown for implementation
4. Task execution (where repository cloning happens)
```

## Step 3: MCP Server Capabilities

Your power includes these MCP servers:

### Terraform MCP Server  
- Terraform validation and planning
- Infrastructure analysis and recommendations
- AWS Labs official Terraform MCP server

### Filesystem MCP Server
- File operations in current Kiro project directory
- Template management and customization
- Local file system access (uses "." as working directory)

### Git MCP Server
- Git operations for repository management
- Version control and branch operations
- All operations happen in current project directory

## Step 4: Working Directory Setup

**IMPORTANT**: All file operations and git cloning will happen in your current Kiro project directory.

- **Git clone**: Repository will be cloned to current directory
- **File operations**: Templates and configurations managed in current directory
- **Terraform operations**: State and configuration files in current directory

This ensures everything stays organized within your project workspace.

# When to Load Steering Files

- Deploying ECS infrastructure → `ecs-deployment-workflow.md`
- Using AI-DLC methodology → `ai-dlc-infrastructure-workflow.md`
- Choosing setup templates → `template-selection-guide.md`
- Setting up S3 backend → `s3-backend-setup.md`
- Terraform operations → `terraform-execution-steps.md`
- Adding new components → `component-extension-guide.md`
- Following best practices → `aws-terraform-best-practices.md`
- Customizing infrastructure → `infrastructure-customization-workflow.md`
- Troubleshooting issues → `troubleshooting-guide.md`

# Best Practices

## Repository Cloning and Directory Management

**CRITICAL: Handle directory conflicts properly**

### Repository Cloning Strategy:
1. **Clone to temporary directory** first to avoid conflicts
2. **Move files and git history** to current working directory
3. **Clean up temporary directory** to maintain workspace cleanliness
4. **Work in current directory** with merged repository content

### Correct Repository URL:
- **Use**: `https://github.com/santyar0/poc-idp.git` (correct URL)
- **NOT**: `https://github.com/poc-idp/poc-idp.git` (does not exist)

### Directory Handling Examples:
```
# MAIN APPROACH - merge with current directory
git clone https://github.com/santyar0/poc-idp.git temp-poc-idp
mv temp-poc-idp/.git ./
mv temp-poc-idp/* ./
rm -rf temp-poc-idp

# Alternative - dedicated directory (if user prefers isolation)
mkdir ecs-terraform
git clone https://github.com/santyar0/poc-idp.git ecs-terraform
cd ecs-terraform
```

### Smart Cloning Instructions:
When cloning repository, ALWAYS:
1. Clone to `temp-poc-idp` directory first
2. Move all content to current directory
3. Clean up temporary directory
4. Provide clear next steps after merging
5. Verify repository exists before attempting clone

## Environment Sizing (CRITICAL)

**NEVER assume environment size - ALWAYS ask the user first:**

### Sizing Questions:
1. "How many services will you deploy?"
2. "What's your expected traffic volume?"
3. "Is this for development, staging, or production?"
4. "What's your monthly infrastructure budget?"

### Size Recommendations:
- **<5 services, development/MVP, no HA needed** → Small setup
- **5-13 services, small production, some HA needed** → Medium setup
- **13-23 services, full production, complete HA required** → Large setup

## Infrastructure Deployment

**CRITICAL: Repository cloning only happens during task execution phase, NOT during spec creation**

### Spec-Driven Development Approach:
1. **Requirements Phase** - Gather and document all requirements
2. **Design Phase** - Create comprehensive architecture and design
3. **Tasks Phase** - Break down implementation into discrete tasks
4. **Implementation Phase** - Execute tasks (repository cloning happens here)

### Repository-First Implementation (During Task Execution Only):
When executing implementation tasks, ALWAYS follow the exact poc-idp repository workflow:

```
Step 1: Clone to temp → git clone https://github.com/santyar0/poc-idp.git temp-poc-idp
Step 2: Move git history → mv temp-poc-idp/.git ./
Step 3: Move all files → mv temp-poc-idp/* ./
Step 4: Clean up temp → rm -rf temp-poc-idp
Step 5: Create tfbackend → tfbackend/[client-name]-[account-id]-[region].tfbackend
Step 6: Create S3 bucket → with proper permissions for Terraform state
Step 7: Create tfvars → tfvars/[client-name]-[account-id]-[region].tfvars
Step 8: Configure AWS → aws sts get-caller-identity (verify account)
Step 9: Initialize → terraform init -backend-config=tfbackends/[file].tfbackend -reconfigure
Step 10: Plan → terraform plan -var-file=tfvars/[file].tfvars
Step 11: Review/modify → update tfvars if needed, repeat plan
Step 12: Apply → terraform apply -var-file=tfvars/[file].tfvars
```

**Repository Cloning Workflow (Implementation Phase Only):**
- Always use the poc-idp repository templates as the foundation
- Clone repository using merge approach during task execution
- Follow exact naming conventions: [client-name]-[aws-account-id]-[region]
- Use tfbackend/ and tfvars/ folder structure from repository
- Store Terraform state in S3 with versioning and encryption
- Apply security best practices and cost optimization from day one

## MCP Server Integration

- **Terraform MCP**: Use for infrastructure validation and planning
- **Filesystem MCP**: Access files in current project directory (uses ".")
- **Git MCP**: Handle version control operations in current workspace

## Working Directory Management

- **Repository cloning**: Handle directory conflicts intelligently
- **Git clone**: Use subdirectory if current directory is not empty
- **File operations**: Use relative paths within project
- **Template customization**: Keep all files in current workspace
- **Terraform state**: Manage state files in project directory

### Repository Cloning Best Practices (Implementation Phase Only):
```bash
# IMPORTANT: This only happens during task execution, NOT during spec creation

# Step 1: Clone to temporary directory
git clone https://github.com/santyar0/poc-idp.git temp-poc-idp

# Step 2: Move git history and files to current directory
mv temp-poc-idp/.git ./
mv temp-poc-idp/* ./

# Step 3: Clean up temporary directory
rm -rf temp-poc-idp

# Alternative approach - dedicated directory (if user prefers isolation)
# mkdir ecs-terraform
# git clone https://github.com/santyar0/poc-idp.git ecs-terraform
# cd ecs-terraform
```

**CRITICAL INSTRUCTION FOR SPEC WORKFLOW:**
When user asks for ECS infrastructure deployment:

**SPEC CREATION PHASE (First):**
1. Gather requirements using userInput tool for service count/environment
2. Create requirements.md with EARS patterns and acceptance criteria
3. Create design.md with architecture, components, and correctness properties
4. Create tasks.md with implementation plan and discrete coding tasks

**IMPLEMENTATION PHASE (After spec is complete):**
1. Clone the poc-idp repository to temporary directory using Git MCP server
2. Move all files and git history to current working directory
3. Clean up temporary directory
4. **Collect configuration information through structured dialog:**
   - Application name (for resource naming)
   - Environment (dev/staging/prod)
   - AWS Account ID (12-digit number)
   - AWS Region (us-east-1, us-west-2, eu-west-1, etc.)
5. Create tfbackend file: `tfbackend/[application-name]-[account-id]-[region].tfbackend`
6. Create S3 bucket with proper permissions for Terraform state
7. Create tfvars file: `tfvars/[application-name]-[account-id]-[region].tfvars`
8. Use appropriate template (small/medium/large) based on spec requirements
9. Modify mandatory variables with collected configuration information
10. Configure AWS access and verify account: `aws sts get-caller-identity`
11. Initialize Terraform: `terraform init -backend-config=tfbackends/[file].tfbackend -reconfigure`
12. Review plan: `terraform plan -var-file=tfvars/[file].tfvars`
13. Modify configuration if needed and repeat plan step
14. Deploy: `terraform apply -var-file=tfvars/[file].tfvars`

**File Naming Convention:**
- tfbackend file: `[application-name]-[aws-account-id]-[region].tfbackend`
- tfvars file: `[application-name]-[aws-account-id]-[region].tfvars`
- Use existing templates: `example-small-setup.tfvars`, `example-medium-setup.tfvars`, `example-large-setup.tfvars`

**NEVER create new Terraform files - ALWAYS use repository templates merged into current directory**
**NEVER skip spec phases - Requirements → Design → Tasks → Implementation is mandatory**

## Example: Smart Environment Selection

```
User: "I need to setup ECS cluster on AWS"

Agent: [Uses userInput tool with service count question]

User: [Clicks "1-4 services (Small Setup)"]

Agent: "Perfect! With 1-4 services, I'll help you create a comprehensive spec for Small setup ECS infrastructure. Let's start with gathering requirements..."

[Agent then follows complete spec workflow:]
1. Requirements Phase - Document requirements with EARS patterns
2. Design Phase - Create architecture and design document
3. Tasks Phase - Break down implementation into tasks
4. Implementation Phase - Execute tasks using poc-idp repository templates
```

**Correct behavior for generic Docker requests:**
```
User: "I need deploy docker to AWS"

Agent: [Uses userInput tool with Docker deployment options]

User: [Clicks "ECS (Elastic Container Service)"]

Agent: [Uses userInput tool with service count question]

User: [Clicks "5-13 services (Medium Setup)"]

Agent: "Excellent! I'll help you create a comprehensive spec for deploying 5-13 services on ECS using Medium setup. Let's start with the requirements phase..."

[Agent then follows complete spec workflow before any implementation]
```

## AI-DLC Phases

1. **Inception**: Requirements analysis and architecture design (with environment sizing)
2. **Construction**: Implementation with testing and validation
3. **Operations**: Deployment, monitoring, and documentation