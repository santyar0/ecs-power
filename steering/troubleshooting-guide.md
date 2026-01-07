# Troubleshooting Guide

Common issues and solutions for ECS infrastructure deployment.

> **For complex troubleshooting**, see `#ai-dlc-infrastructure-workflow`

---

## AWS Authentication

### "Unable to locate credentials"

```bash
# Configure AWS CLI
aws configure

# Verify credentials
aws sts get-caller-identity

# Or use environment variables
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

### "Access Denied" / "UnauthorizedOperation"

**Required permissions for ECS deployment:**
- EC2 (VPC, subnets, security groups, load balancers)
- ECS (clusters, services, task definitions)
- RDS, ElastiCache (if using)
- IAM (roles and policies)
- CloudWatch (logs and alarms)
- Route53 (if using DNS)

---

## S3 Backend Issues

### "Bucket does not exist"

```bash
# Create bucket
aws s3 mb s3://your-terraform-state-bucket --region us-east-1

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket your-terraform-state-bucket \
  --versioning-configuration Status=Enabled
```

### "Access denied to S3 bucket"

```bash
# Check bucket access
aws s3 ls s3://your-terraform-state-bucket

# Verify IAM permissions for S3 operations
```

### "DynamoDB table not found"

```bash
aws dynamodb create-table \
  --table-name terraform-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

---

## Terraform Configuration

### "CHANGE_ME_PLEASE values not replaced"

```bash
# Find all placeholders
grep -n "CHANGE_ME_PLEASE" tfvars/your-project.tfvars

# Required replacements:
# - account_id: 12-digit AWS account ID
# - region: AWS region
# - application: Application name
# - environment: dev, staging, or prod
```

### "Invalid variable value"

- Check variable format (strings quoted, booleans true/false)
- Verify allowed values in AWS documentation
- Ensure numbers aren't quoted

### "Resource already exists"

```bash
# Import existing resource
terraform import aws_ecs_cluster.main existing-cluster-name

# Or use different resource names in tfvars
```

---

## AWS Service Limits

### "LimitExceededException"

```bash
# Check service quotas
aws service-quotas list-service-quotas --service-code ec2

# Request quota increase via AWS Support Console
```

### "InsufficientCapacity"

- Try different availability zones
- Use different instance types
- Deploy in different region

---

## Network Issues

### "InvalidVpcID.NotFound"

- Let Terraform create VPC (don't reference external)
- Verify VPC ID if using existing
- Check region matches

### "InvalidSubnet"

- Check CIDR blocks for overlap
- Verify AZs exist in region
- Review public vs private subnet settings

---

## Database Issues

### "DBClusterAlreadyExistsFault"

- Use unique cluster identifier
- Import existing cluster if needed
- Check naming conflicts across regions

### "InvalidDBInstanceClass"

- Verify instance class available in region
- Check engine compatibility
- Use supported instance types

### RDS Connection Issues

- Check security groups for ingress rules
- Verify database in private subnets
- Check route tables
- Use RDS endpoint from Terraform outputs

---

## ECS Service Issues

### "Service failed to stabilize"

```bash
# Check logs
aws logs describe-log-groups --log-group-name-prefix /ecs/your-app
aws logs get-log-events --log-group-name /ecs/your-app/container-name
```

- Check task definition (container images accessible)
- Verify resource allocation (CPU/memory)
- Check health check path

### "CannotPullContainerError"

- Verify image exists in ECR/Docker Hub
- Check ECR permissions
- Verify image tags
- Check network connectivity to registry

---

## Load Balancer Issues

### "DuplicateLoadBalancerName"

```bash
# Check existing load balancers
aws elbv2 describe-load-balancers

# Use unique ALB names in tfvars
```

### ALB Health Check Failures

- Check health check path responds correctly
- Verify port configuration matches
- Check security groups allow ALB → ECS
- Review application logs for startup issues

---

## State Management

### "State lock timeout"

```bash
# Wait for other operations, or force unlock
terraform force-unlock LOCK_ID

# Check DynamoDB for stuck locks
```

### State Corruption

```bash
# Restore from S3 backup
aws s3 cp s3://your-bucket/path/to/backup.tfstate terraform.tfstate

# Refresh and import
terraform refresh
terraform import aws_resource.name resource-id
```

---

## Performance Issues

### Slow Terraform Operations

```bash
# Target specific resources
terraform plan -target=aws_ecs_service.app

# Increase parallelism
terraform apply -parallelism=20
```

### Resource Creation Timeouts

- Be patient (RDS, ElastiCache take 10-20 minutes)
- Check AWS console for status
- Increase timeout values if needed

---

## Quick Reference

| Issue | Quick Fix |
|-------|-----------|
| Credentials not found | `aws configure` |
| Bucket doesn't exist | `aws s3 mb s3://bucket` |
| State locked | `terraform force-unlock LOCK_ID` |
| Resource exists | `terraform import` |
| Service won't stabilize | Check CloudWatch logs |
| Health check failing | Verify health check path |

---

## Prevention Tips

1. Always run `terraform plan` before apply
2. Use version control for all configs
3. Test in dev before production
4. Monitor AWS service limits
5. Keep Terraform and providers updated
6. Document custom configurations
7. Set up monitoring and alerting
8. Regular backup verification

---

## Emergency Procedures

### Complete Deployment Failure

1. Don't panic - Terraform tracks what was created
2. Check Terraform state: `terraform state list`
3. Fix underlying issues
4. Re-run apply - continues where it left off
5. Use targeted destroys if needed

### Production Outage

1. Check AWS Service Health Dashboard
2. Review recent Terraform changes
3. Use AWS console for immediate fixes
4. Document manual changes for later import
5. Post-incident review

---

## Related Guides

- `#ecs-deployment-workflow` - Deployment steps (includes S3 backend setup)
- `#poc-idp-style-guide` - Coding patterns and best practices
