# ECS Infrastructure Power

A Kiro Power for deploying and managing ECS infrastructure using the poc-idp repository workflow with AI-DLC methodology and MCP automation.

## Installation

Add this power to Kiro by providing the repository URL in the Powers panel.

## Features

- **AI-DLC Methodology** - Three-phase development lifecycle (Inception, Construction, Operations)
- **MCP Automation** - 6 specialized servers for intelligent operations
- **Repository Integration** - Deep integration with poc-idp patterns
- **Cost Optimization** - Built-in analysis and recommendations
- **Security Validation** - Automated compliance checking
- **Quality Assurance** - Continuous monitoring and error recovery

## Setup Sizes

- **Small** ($50-100/month) - Development environments
- **Medium** ($200-400/month) - Staging environments  
- **Large** ($800-1500/month) - Production environments

## Getting Started

### Standard Deployment
```
"I want to deploy ECS infrastructure for my staging environment"
```

### AI-DLC Deployment
```
"Using AI-DLC, I want to deploy production ECS infrastructure"
```

## Prerequisites

- AWS CLI configured
- Terraform installed (1.0+)
- S3 bucket for state storage

## MCP Servers

This power automatically configures these MCP servers:
- **git** - Repository operations
- **aws** - AWS resource management
- **filesystem** - File operations
- **aws-docs** - AWS documentation
- **github** - Repository analysis
- **terraform** - Infrastructure validation

## Structure

```
ecs-infrastructure-power/
├── POWER.md              # Main power definition
├── mcp.json              # MCP server configuration
├── README.md             # This file
├── steering/             # Workflow guides (9 files)
└── hooks/                # Automation hooks (9 files)
```