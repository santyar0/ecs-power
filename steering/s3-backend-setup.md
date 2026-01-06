# S3 Backend Setup for Terraform State

Terraform state management is crucial for team collaboration and infrastructure safety. This guide helps you set up S3 backend storage using AWS MCP server integration.

## Why S3 Backend?

- **Team Collaboration:** Multiple team members can work on the same infrastructure
- **State Locking:** Prevents concurrent modifications that could corrupt state
- **Backup & Recovery:** State is safely stored and versioned in S3
- **Encryption:** State data is encrypted at rest and in transit

## Prerequisites

- AWS CLI configured with appropriate permissions
- An existing S3 bucket (or permissions to create one)
- DynamoDB permissions for state locking

## Option 1: Use Existing S3 Bucket

If you already have an S3 bucket for Terraform state:

**Ask Kiro to:**
> "Validate my S3 bucket [bucket-name] for Terraform state storage"

This will use the AWS MCP server to:
- Check if bucket exists and you have access
- Verify versioning status (recommended for safety)
- Test encryption settings
- Validate region configuration
- Check for existing state files

**If versioning needs to be enabled, ask:**
> "Enable versioning on my S3 bucket [bucket-name]"

## Option 2: Create New S3 Bucket

**Ask Kiro to:**
> "Create a new S3 bucket for Terraform state in region [region]"

This will use the AWS MCP server to:
- Generate a globally unique bucket name
- Create the bucket with proper security settings
- Enable versioning for state file safety
- Configure server-side encryption
- Block public access for security
- Set up proper bucket policies

## DynamoDB Table for State Locking

**Ask Kiro to:**
> "Create DynamoDB table for Terraform state locking with bucket [bucket-name]"

This will:
- Create a DynamoDB table named `{bucket-name}-lock`
- Configure with proper LockID primary key
- Set appropriate read/write capacity
- Verify table is ready for use

## Backend Configuration Generation

**Ask Kiro to:**
> "Generate Terraform backend configuration for project [project-name]"

This will use the Filesystem MCP server to:
- Create the `tfbackends/` directory if needed
- Generate a properly formatted `.tfbackend` file
- Use consistent naming conventions for state keys
- Include all required backend parameters

**Generated configuration will include:**
```hcl
bucket         = "your-terraform-state-bucket"
key            = "terraform/your-app/your-env/terraform.tfstate"
region         = "your-region"
encrypt        = true
dynamodb_table = "your-terraform-state-bucket-lock"
```

## MCP-Enhanced Features

### Intelligent Validation
The AWS MCP server provides:
- Real-time bucket accessibility checks
- Automatic permission validation
- Versioning status verification
- Encryption configuration review
- Cost estimation for state storage

### Smart Configuration
The Filesystem MCP server:
- Generates consistent file naming
- Validates configuration syntax
- Creates backup copies
- Manages directory structure

### Security Checks
Automatic validation of:
- IAM permissions for S3 and DynamoDB
- Bucket policies and access controls
- Encryption settings
- Public access blocking

## Required IAM Permissions

The AWS MCP server will verify you have these permissions:

### S3 Permissions
- `s3:ListBucket` on the state bucket
- `s3:GetObject` on state files
- `s3:PutObject` for state updates
- `s3:DeleteObject` for cleanup operations

### DynamoDB Permissions
- `dynamodb:GetItem` for lock checking
- `dynamodb:PutItem` for lock creation
- `dynamodb:DeleteItem` for lock release

## Testing Your Backend

**Ask Kiro to:**
> "Test my Terraform backend configuration"

This will:
- Initialize Terraform with your backend config
- Verify state storage is working
- Test state locking functionality
- Confirm all permissions are correct

## Common Issues & MCP Solutions

### Issue: "Bucket does not exist"
**MCP Solution:** AWS server will automatically detect this and offer to create the bucket

### Issue: "Access Denied"
**MCP Solution:** AWS server will analyze IAM permissions and suggest required policy changes

### Issue: "DynamoDB table not found"
**MCP Solution:** AWS server will detect missing table and offer to create it with correct configuration

### Issue: "State lock timeout"
**MCP Solution:** AWS server can check lock status and offer to force unlock if safe

## Best Practices with MCP

1. **Automated Validation:** Let MCP servers verify configurations before use
2. **Consistent Naming:** Use MCP-generated naming conventions
3. **Security First:** MCP servers enforce security best practices
4. **Team Collaboration:** Share MCP-generated configurations with team
5. **Regular Monitoring:** Use MCP servers to check backend health

## Security Considerations

MCP servers automatically enforce:
- Encryption at rest and in transit
- Proper IAM policies with least privilege
- Public access blocking
- Audit trail through CloudTrail
- Regular permission reviews

## Team Collaboration with MCP

When working with a team:
1. **Share backend configs** generated by MCP servers
2. **Use consistent MCP commands** across team members
3. **Leverage MCP validation** before making changes
4. **Monitor access** through MCP AWS integration

## Example MCP Workflow

Here's a complete backend setup using MCP:

**You:** "I need to set up Terraform state storage for my project myapp-prod"

**Kiro with MCP:**
1. "Let me check if you have an existing S3 bucket..."
2. "I'll create a new bucket with proper security settings..."
3. "Creating DynamoDB table for state locking..."
4. "Generating backend configuration file..."
5. "Testing the configuration..."
6. "✅ Backend is ready! Here's your configuration file."

## Next Steps

After MCP sets up your backend:
1. **Return to main deployment workflow**
2. **Initialize Terraform** with MCP-generated config
3. **Proceed with infrastructure deployment**

**Need help with deployment?** See: `#ecs-deployment-workflow`

## Advanced MCP Features

### Automated Monitoring
- Continuous backend health checks
- State file size monitoring
- Lock duration tracking
- Cost optimization suggestions

### Backup Management
- Automated state file backups
- Version history management
- Recovery point recommendations
- Cross-region replication setup

### Compliance Checking
- Security policy validation
- Encryption standard compliance
- Access audit reporting
- Regulatory requirement checks