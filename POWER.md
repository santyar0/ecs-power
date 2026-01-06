---
name: ecs-infrastructure-power
displayName: ECS Infrastructure Deployment
description: Deploy and manage ECS infrastructure using poc-idp repository patterns with AI-DLC methodology and MCP automation
version: 1.0.0
keywords: [ecs, terraform, aws, infrastructure, deployment, poc-idp, ai-dlc]
---

# Power: ECS Infrastructure Deployment

## Purpose

Deploy and manage ECS infrastructure using the battle-tested poc-idp repository workflow with AI-DLC methodology and intelligent MCP automation. This power provides comprehensive infrastructure deployment with cost optimization, security validation, and quality assurance.

## Activation signals

- Keywords: `ecs`, `terraform`, `aws`, `infrastructure`, `deployment`, `poc-idp`, `ai-dlc`
- Typical prompts:
  - "I want to deploy ECS infrastructure for my staging environment"
  - "Using AI-DLC, I want to deploy production ECS infrastructure"
  - "Help me set up ECS infrastructure with Terraform"
  - "Deploy AWS infrastructure using poc-idp templates"

## Agent behavior

- Do: Use poc-idp repository templates and follow established patterns
- Do: Apply AI-DLC methodology for structured development lifecycle
- Do: Leverage MCP servers for intelligent automation and validation
- Do: Follow AWS and Terraform best practices consistently
- Do: Provide cost estimates and security recommendations
- Don't: Generate new infrastructure code from scratch
- Don't: Skip validation steps or bypass established workflows
- Don't: Ignore security or cost optimization opportunities

## Golden path

1. **Clone poc-idp repository** using Git MCP server
2. **Choose setup size** (small/medium/large) based on requirements
3. **Customize configuration** with project-specific values
4. **Setup S3 backend** for Terraform state management
5. **Initialize and plan** Terraform deployment
6. **Deploy infrastructure** with monitoring and validation
7. **Verify deployment** and configure applications

## Examples

**Prompt:** "I want to deploy ECS infrastructure for my staging environment"
**Expected outcome:** Agent guides through poc-idp workflow, recommends medium setup, helps customize configuration, and deploys infrastructure with proper monitoring.

**Prompt:** "Using AI-DLC, I want to deploy production ECS infrastructure"
**Expected outcome:** Agent activates AI-DLC three-phase workflow (Inception, Construction, Operations) for structured production deployment with comprehensive documentation and monitoring.

**Prompt:** "Help me add RDS database to my existing ECS infrastructure"
**Expected outcome:** Agent uses component extension guide to add RDS following established patterns, updates Terraform configuration, and maintains consistency with existing infrastructure.