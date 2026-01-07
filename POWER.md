---
name: "ecs-infrastructure-power"
displayName: "ECS Infrastructure Deployment"
description: "Deploy and manage ECS infrastructure using poc-idp repository patterns with AI-DLC methodology and MCP automation"
keywords: ["ecs", "terraform", "aws", "infrastructure", "deployment", "poc-idp", "ai-dlc", "docker", "containers", "microservices", "microservice", "architecture", "fargate", "cluster", "setup", "deploy"]
mcpServers: ["awslabs.terraform-mcp-server", "filesystem", "git"]
---

# ECS Infrastructure Power

Deploy production-ready ECS infrastructure using battle-tested Terraform templates.

## Prerequisites

Verify before starting:

```bash
aws sts get-caller-identity    # AWS CLI configured
terraform --version            # Terraform 1.0+
git --version                  # Git installed
```

**Required:** S3 bucket for Terraform state (setup included in deployment workflow)

---

## Quick Start

### 1. Determine Setup Size

| Setup | Services | Use Case | Cost |
|-------|----------|----------|------|
| **Small** | <5 | Dev, MVP, testing | $50-100/mo |
| **Medium** | 5-13 | Staging, small prod | $200-400/mo |
| **Large** | 13-23 | Production, full HA | $800-1500/mo |

### 2. Follow Spec Workflow

```
Requirements → Design → Tasks → Implementation
```

**Repository cloning only happens during Implementation phase.**

### 3. Deploy

See `#ecs-deployment-workflow` for complete steps.

---

## Activation Keywords

Power activates on: `ecs`, `microservices`, `aws docker deploy`, `ecs setup`, `fargate`, `container deployment`, `aws infrastructure`, `terraform aws`

---

## MCP Servers & AWS Integration

| Component | Purpose | Automation Capabilities |
|-----------|---------|------------------------|
| **Terraform MCP** | AWS CLI execution, Terraform operations | ✅ **Direct AWS resource analysis, credential verification, infrastructure operations** |
| **Filesystem MCP** | File operations, template management | ✅ Configuration file generation, template copying, automated adjustments |
| **Git MCP** | Repository operations, version control | ✅ Repository cloning, branch management, file operations |

**Direct AWS Integration:**
- Uses developer's existing AWS credentials through MCP servers
- No additional script installation or setup required
- Real-time AWS resource analysis through AWS CLI
- Intelligent conflict detection and resolution
- Cross-platform compatibility through MCP servers

**Key Analysis Features:**
- VPC CIDR conflict detection and automatic resolution
- Resource naming conflict prevention
- AWS service limit validation
- S3 backend security configuration
- Comprehensive resource inventory

**Automation Level:** Complete deployment automation through MCP servers using developer's AWS credentials, including comprehensive AWS account resource scanning, intelligent conflict prevention, and secure S3 backend setup.

---

## Steering Files

| Guide | When to Use |
|-------|-------------|
| `#ecs-deployment-workflow` | **Complete deployment process** (includes setup size selection and S3 backend setup) |
| `#ai-dlc-infrastructure-workflow` | Structured planning methodology |
| `#component-extension-guide` | Adding and extending components |
| `#poc-idp-style-guide` | **Repository coding style and patterns** |
| `#troubleshooting-guide` | Common issues and solutions |

---

## Critical Rules

### Always Ask First

**For ECS requests**, ask about service count:
- 1-4 services → Small Setup
- 5-13 services → Medium Setup  
- 13-23 services → Large Setup
- Unsure → Ask follow-up questions

**For generic Docker requests**, clarify deployment approach first (ECS, EKS, EC2, Fargate).

### Never Assume

- ❌ Don't assume service count from vague requests
- ❌ Don't assume environment type without asking
- ❌ Don't assume setup size from keywords alone
- ✅ Always ask at least one clarifying question
- ✅ Always follow spec workflow before implementation

### Terraform Only

This power exclusively uses Terraform with poc-idp repository templates. Never generate new Terraform code - always use existing templates.

### Repository URL

**Correct:** `https://github.com/santyar0/poc-idp.git`

---

## Configuration Collection

During implementation, collect through interactive dialog:

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `aws_profile` | AWS CLI profile to use | `default` | Yes |
| `account_id` | 12-digit AWS account ID | `253650698585` | Yes |
| `region` | AWS region | `us-east-1` | Yes |
| `application` | Application name (AWS resource naming) | `web` | Yes |
| `environment` | Environment type | `dev` | Yes |
| `additional_tags` | Optional resource tags | `{}` | No |

**Interactive Process:** User can provide values or skip to use defaults.

**AWS Resource Verification & Automation:** 
- **Direct AWS CLI integration**: Comprehensive account scanning through MCP servers
- **Developer credential usage**: Leverages user's existing AWS access and profiles
- **Intelligent conflict detection**: Automatic CIDR and naming conflict resolution
- **Real-time analysis**: Direct AWS API calls for current resource state
- **Cross-platform compatibility**: Works through MCP servers on all platforms
- **No additional setup**: Uses existing AWS CLI and MCP infrastructure

**Automated S3 Backend Setup:**
- Creates S3 buckets with security best practices through AWS CLI
- Sets up DynamoDB tables for state locking
- Validates existing configurations and fixes issues automatically
- Generates Terraform backend configuration files
- All operations performed through MCP servers using developer credentials

**File Generation:** 
- Template: `tfvars-sample/example-{size}-setup.tfvars`
- Generated: `tfvars/[application]-[account_id]-[region].tfvars`
- Backend: `tfbackends/[application]-[account_id]-[region].tfbackend`

---

## Setup Descriptions

### Small Setup
- **Network:** VPC, 3 subnets per type, single NAT, bastion
- **Database:** RDS Aurora (1 instance, no MultiAZ)
- **Application:** ECS, up to 5 services, Fargate
- **Monitoring:** Basic CloudWatch alarms
- **Backups:** None

### Medium Setup
- **Network:** VPC + private endpoints
- **Database:** RDS Aurora (2 instances), ElastiCache, DocumentDB
- **Application:** ECS, up to 13 services, Fargate
- **Monitoring:** CloudWatch + SNS notifications
- **Backups:** AWS Backup enabled

### Large Setup
- **Network:** Full HA NAT, Client VPN, VPC flow logs
- **Database:** RDS Serverless V2 (MultiAZ), ElastiCache (MultiAZ), DocumentDB cluster
- **Application:** ECS, up to 23 services, Fargate with autoscaling
- **Monitoring:** Comprehensive CloudWatch + SNS
- **Backups:** Full AWS Backup coverage

---

## AI-DLC Integration

For complex deployments, use AI-DLC methodology:

**Activate:** "Using AI-DLC, I want to [infrastructure intent]"

**Phases:**
1. **Inception** - Requirements and architecture
2. **Construction** - Design and implementation
3. **Operations** - Deployment and monitoring

See `#ai-dlc-infrastructure-workflow` for details.
