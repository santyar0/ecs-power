# Troubleshooting Guide

Common issues and solutions when deploying ECS infrastructure using the poc-idp repository.

## AI-DLC Troubleshooting Support

For complex troubleshooting scenarios, consider using the **AI-Driven Development Lifecycle (AI-DLC)** approach:

**Start with AI-DLC:** "Using AI-DLC, I need help troubleshooting [specific issue] with my infrastructure"

**AI-DLC provides:**
- **Systematic Diagnosis**: Structured problem analysis and root cause identification
- **Solution Planning**: Step-by-step resolution approach with risk assessment
- **Implementation Guidance**: Detailed execution steps with validation checkpoints
- **Prevention Strategies**: Recommendations to prevent similar issues

**When to use AI-DLC for troubleshooting:**
- Complex multi-service failures
- Performance degradation issues
- Security incidents or compliance violations
- State corruption or deployment failures
- Cross-team coordination needs

**For detailed AI-DLC guidance:** See `#ai-dlc-infrastructure-workflow`

## Standard Troubleshooting Procedures

## AWS Authentication Issues

### Error: "Unable to locate credentials"

**Symptoms:**
```
Error: unable to locate credentials
```

**Solutions:**
1. **Configure AWS CLI:**
   ```bash
   aws configure
   # Enter your Access Key ID, Secret Access Key, region, and output format
   ```

2. **Verify credentials:**
   ```bash
   aws sts get-caller-identity
   ```

3. **Use environment variables:**
   ```bash
   export AWS_ACCESS_KEY_ID="your-access-key"
   export AWS_SECRET_ACCESS_KEY="your-secret-key"
   export AWS_DEFAULT_REGION="us-east-1"
   ```

4. **Use IAM roles (EC2/ECS):**
   - Attach appropriate IAM role to your EC2 instance
   - No additional configuration needed

### Error: "Access Denied" or "UnauthorizedOperation"

**Symptoms:**
```
Error: UnauthorizedOperation: You are not authorized to perform this operation
```

**Solutions:**
1. **Check IAM permissions** - Ensure your user/role has required permissions
2. **Review policy attachments** - Verify policies are attached to your user/role
3. **Check service limits** - Some operations require specific permissions

**Required IAM permissions for ECS deployment:**
- EC2 (VPC, subnets, security groups, load balancers)
- ECS (clusters, services, task definitions)
- RDS (if using database)
- ElastiCache (if using caching)
- IAM (roles and policies)
- CloudWatch (logs and alarms)
- Route53 (if using DNS)

## S3 Backend Issues

### Error: "Bucket does not exist"

**Symptoms:**
```
Error: Failed to get existing workspaces: S3 bucket does not exist
```

**Solutions:**
1. **Create the bucket:**
   ```bash
   aws s3 mb s3://your-terraform-state-bucket --region us-east-1
   ```

2. **Verify bucket name** in your tfbackend file
3. **Check region** - bucket must exist in the specified region

### Error: "Access denied to S3 bucket"

**Solutions:**
1. **Check bucket permissions:**
   ```bash
   aws s3 ls s3://your-terraform-state-bucket
   ```

2. **Verify IAM permissions** for S3 operations
3. **Check bucket policy** - ensure it allows your user/role

### Error: "DynamoDB table not found"

**Symptoms:**
```
Error: ResourceNotFoundException: Table not found: terraform-state-lock
```

**Solutions:**
1. **Create DynamoDB table:**
   ```bash
   aws dynamodb create-table \
     --table-name terraform-state-lock \
     --attribute-definitions AttributeName=LockID,AttributeType=S \
     --key-schema AttributeName=LockID,KeyType=HASH \
     --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
   ```

2. **Verify table name** in tfbackend file matches actual table name

## Terraform Configuration Issues

### Error: "CHANGE_ME_PLEASE values not replaced"

**Symptoms:**
- Terraform plan shows resources with "CHANGE_ME_PLEASE" values
- Invalid resource names or configurations

**Solutions:**
1. **Find all placeholders:**
   ```bash
   grep -n "CHANGE_ME_PLEASE" tfvars/your-project.tfvars
   ```

2. **Replace systematically:**
   - `account_id` - Your 12-digit AWS account ID
   - `region` - Your preferred AWS region
   - `application` - Your application name (alphanumeric, hyphens)
   - `environment` - Environment name (dev, staging, prod)

3. **Validate replacements:**
   ```bash
   # Should return no results
   grep "CHANGE_ME_PLEASE" tfvars/your-project.tfvars
   ```

### Error: "Invalid variable value"

**Symptoms:**
```
Error: Invalid value for variable "instance_class"
```

**Solutions:**
1. **Check variable format** - ensure strings are quoted, booleans are true/false
2. **Verify allowed values** - check AWS documentation for valid instance types
3. **Review variable types** - ensure numbers aren't quoted, objects use proper syntax

### Error: "Resource already exists"

**Symptoms:**
```
Error: ResourceAlreadyExistsException: Cluster already exists
```

**Solutions:**
1. **Import existing resource:**
   ```bash
   terraform import aws_ecs_cluster.main existing-cluster-name
   ```

2. **Use different resource names** in your tfvars file
3. **Check for naming conflicts** with existing resources

## AWS Service Limits

### Error: "LimitExceededException"

**Symptoms:**
```
Error: LimitExceededException: Cannot exceed quota for PrefixListsPerVpc
```

**Solutions:**
1. **Check service quotas:**
   ```bash
   aws service-quotas list-service-quotas --service-code ec2
   ```

2. **Request quota increases** through AWS Support Console
3. **Use different regions** if limits are regional
4. **Optimize resource usage** - reduce number of resources if possible

### Error: "InsufficientCapacity"

**Symptoms:**
```
Error: InsufficientCapacity: Insufficient capacity in availability zone
```

**Solutions:**
1. **Try different AZs** - modify availability zones in configuration
2. **Use different instance types** - try smaller or different instance families
3. **Deploy in different region** if capacity issues persist

## Network Configuration Issues

### Error: "InvalidVpcID.NotFound"

**Symptoms:**
```
Error: InvalidVpcID.NotFound: The vpc ID 'vpc-12345' does not exist
```

**Solutions:**
1. **Let Terraform create VPC** - don't reference external VPCs unless necessary
2. **Verify VPC ID** if using existing VPC
3. **Check region** - VPC must exist in the same region

### Error: "InvalidSubnet"

**Solutions:**
1. **Check CIDR blocks** - ensure no overlap with existing subnets
2. **Verify availability zones** - ensure AZs exist in your region
3. **Review subnet configuration** - public vs private subnet settings

## Database Issues

### Error: "DBClusterAlreadyExistsFault"

**Solutions:**
1. **Use unique cluster identifier** in tfvars file
2. **Import existing cluster** if you want to manage it with Terraform
3. **Check for naming conflicts** across regions

### Error: "InvalidDBInstanceClass"

**Solutions:**
1. **Verify instance class** is available in your region
2. **Check engine compatibility** - not all instance classes work with all engines
3. **Use supported instance types** - check AWS documentation

### RDS Connection Issues

**Symptoms:**
- Can't connect to database from applications
- Timeout errors

**Solutions:**
1. **Check security groups** - ensure proper ingress rules
2. **Verify subnet groups** - database should be in private subnets
3. **Check route tables** - ensure proper routing for private subnets
4. **Verify DNS resolution** - use RDS endpoint from Terraform outputs

## ECS Service Issues

### Error: "Service failed to stabilize"

**Symptoms:**
```
Error: service didn't stabilize within the expected time
```

**Solutions:**
1. **Check task definition** - ensure container images are accessible
2. **Verify resource allocation** - CPU/memory limits appropriate
3. **Check health checks** - ensure health check path is correct
4. **Review logs:**
   ```bash
   aws logs describe-log-groups --log-group-name-prefix /ecs/your-app
   aws logs get-log-events --log-group-name /ecs/your-app/container-name
   ```

### Error: "CannotPullContainerError"

**Solutions:**
1. **Verify image exists** in ECR or Docker Hub
2. **Check ECR permissions** - ensure ECS can pull images
3. **Verify image tags** - ensure specified tag exists
4. **Check network connectivity** - ensure ECS can reach image registry

## Load Balancer Issues

### Error: "DuplicateLoadBalancerName"

**Solutions:**
1. **Use unique ALB names** in tfvars file
2. **Check existing load balancers:**
   ```bash
   aws elbv2 describe-load-balancers
   ```

### ALB Health Check Failures

**Symptoms:**
- Targets showing as unhealthy
- 502/503 errors from ALB

**Solutions:**
1. **Check health check path** - ensure application responds on specified path
2. **Verify port configuration** - container port matches target group port
3. **Check security groups** - ALB can reach ECS tasks
4. **Review application logs** for startup issues

## DNS and Route53 Issues

### Error: "HostedZoneNotFound"

**Solutions:**
1. **Create hosted zone** if it doesn't exist
2. **Verify domain ownership** - ensure you control the domain
3. **Check zone configuration** in tfvars file

### Certificate Issues

**Solutions:**
1. **Verify certificate ARN** in tfvars file
2. **Ensure certificate is in correct region** (us-east-1 for CloudFront)
3. **Check certificate validation** - ensure DNS validation is complete

## Performance Issues

### Slow Terraform Operations

**Solutions:**
1. **Use targeted operations** when possible:
   ```bash
   terraform plan -target=aws_ecs_service.app
   ```

2. **Increase parallelism:**
   ```bash
   terraform apply -parallelism=20
   ```

3. **Check network connectivity** to AWS APIs

### Resource Creation Timeouts

**Solutions:**
1. **Be patient** - some resources (RDS, ElastiCache) take 10-20 minutes
2. **Check AWS console** for resource status
3. **Increase timeout values** in Terraform configuration if needed

## State Management Issues

### Error: "State lock timeout"

**Solutions:**
1. **Wait for other operations** to complete
2. **Force unlock if necessary:**
   ```bash
   terraform force-unlock LOCK_ID
   ```
3. **Check DynamoDB table** for stuck locks

### State Corruption

**Solutions:**
1. **Restore from S3 backup:**
   ```bash
   aws s3 cp s3://your-bucket/path/to/backup.tfstate terraform.tfstate
   ```

2. **Use state recovery commands:**
   ```bash
   terraform refresh
   terraform import aws_resource.name resource-id
   ```

## Getting Help

### AWS Support
- Use AWS Support Console for service limit increases
- Check AWS Service Health Dashboard for outages
- Review AWS documentation for service-specific issues

### Terraform Community
- Terraform GitHub issues for provider bugs
- HashiCorp forums for configuration help
- Stack Overflow for common problems

### Internal Resources
- Check team documentation for organization-specific configurations
- Review previous deployments for working examples
- Consult with DevOps team for complex issues

## Prevention Tips

1. **Always run plan first** before applying changes
2. **Use version control** for all configuration files
3. **Test in development** before deploying to production
4. **Monitor AWS service limits** proactively
5. **Keep Terraform and providers updated**
6. **Document custom configurations** for team reference
7. **Set up proper monitoring** and alerting
8. **Regular backup verification** of Terraform state

## Emergency Procedures

### Complete Deployment Failure
1. **Don't panic** - Terraform tracks what was created
2. **Check Terraform state** to see what exists
3. **Fix underlying issues** (permissions, limits, etc.)
4. **Re-run apply** - Terraform will continue where it left off
5. **Use targeted destroys** if needed to clean up problematic resources

### Production Outage
1. **Check AWS Service Health** first
2. **Review recent changes** in Terraform state
3. **Use AWS console** for immediate fixes if needed
4. **Document manual changes** for later Terraform import
5. **Post-incident review** to prevent recurrence