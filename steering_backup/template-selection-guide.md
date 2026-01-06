# Template Selection Guide

Choose the right setup size for your needs. Each template is pre-configured with appropriate resources for different use cases.

## AI-DLC Template Selection

For complex template selection decisions, consider using the **AI-Driven Development Lifecycle (AI-DLC)** approach:

**Start with AI-DLC:** "Using AI-DLC, I want to select the optimal template for [your specific requirements]"

**AI-DLC provides:**
- **Requirements Analysis**: Systematic evaluation of your needs
- **Cost-Benefit Analysis**: Detailed comparison of setup sizes
- **Growth Planning**: Consideration of future scaling needs
- **Risk Assessment**: Evaluation of availability and performance requirements

**When to use AI-DLC for template selection:**
- Complex requirements with multiple constraints
- Budget optimization needs
- Compliance or regulatory requirements
- Multi-environment planning (dev/staging/prod)
- Uncertain growth projections

**For detailed AI-DLC guidance:** See `#ai-dlc-infrastructure-workflow`

## Small Setup (`example-small-setup.tfvars`)

**Best for:** Development, testing, proof-of-concepts

**Estimated Cost:** $50-100/month

**Resources:**
- **RDS:** Single `db.t4g.medium` instance, no deletion protection
- **ElastiCache:** Disabled (not created)
- **DocumentDB:** Disabled (not created)
- **VPC:** Single NAT gateway, no flow logs
- **Backup Services:** None configured
- **Monitoring:** Basic CloudWatch alarms only

**Use Cases:**
- Development environments
- Testing and experimentation
- Small applications with minimal traffic
- Cost-sensitive deployments

**Limitations:**
- Single point of failure (no high availability)
- No caching layer
- No document database
- Minimal monitoring

## Medium Setup (`example-medium-setup.tfvars`)

**Best for:** Staging, medium-scale applications

**Estimated Cost:** $200-400/month

**Resources:**
- **RDS:** 2-instance cluster with `db.t4g.medium`, deletion protection enabled
- **ElastiCache:** Single `cache.t4g.medium` node, single AZ
- **DocumentDB:** Single `db.t4g.medium` instance
- **VPC:** Single NAT gateway, no flow logs
- **Backup Services:** Enabled for RDS, DocumentDB, EFS, S3, EC2
- **Monitoring:** CloudWatch alarms with SNS notifications

**Use Cases:**
- Staging environments
- Medium-traffic applications
- Applications requiring caching
- Document-based workloads
- Pre-production testing

**Benefits:**
- Database clustering for better performance
- Caching layer for improved response times
- Document database for flexible data models
- Comprehensive backup strategy
- Enhanced monitoring with notifications

## Large Setup (`example-large-setup.tfvars`)

**Best for:** Production, high-availability applications

**Estimated Cost:** $800-1500/month

**Resources:**
- **RDS:** 2-instance cluster with `db.m8g.large`, deletion protection, CloudWatch logs
- **ElastiCache:** 3-cluster setup with `cache.m7g.large`, multi-AZ enabled
- **DocumentDB:** 3-instance cluster with `db.r6g.large`
- **VPC:** High-availability NAT gateways, VPC flow logs (365-day retention)
- **VPN:** Client VPN with connection logging
- **Backup Services:** Full backup coverage for all services
- **Monitoring:** Comprehensive CloudWatch alarms and SNS notifications

**Use Cases:**
- Production environments
- High-traffic applications
- Mission-critical workloads
- Applications requiring 99.9%+ uptime
- Compliance-sensitive deployments

**Benefits:**
- Multi-AZ deployment for high availability
- High-performance instance classes
- Comprehensive logging and monitoring
- VPN access for secure management
- Enterprise-grade backup and recovery
- Network flow logging for security analysis

## Decision Matrix

| Requirement | Small | Medium | Large |
|-------------|-------|--------|-------|
| High Availability | ❌ | ⚠️ Partial | ✅ Full |
| Caching Layer | ❌ | ✅ Basic | ✅ Advanced |
| Document Database | ❌ | ✅ Basic | ✅ Clustered |
| Backup Strategy | ❌ | ✅ Comprehensive | ✅ Enterprise |
| VPN Access | ❌ | ❌ | ✅ |
| Flow Logs | ❌ | ❌ | ✅ |
| Cost Optimization | ✅ | ⚠️ Balanced | ❌ |
| Production Ready | ❌ | ⚠️ Staging | ✅ |

## Customization Options

All templates can be customized by modifying the tfvars file:

### Common Customizations:
- **Instance sizes:** Adjust based on your performance needs
- **Backup retention:** Modify backup schedules and retention periods
- **Monitoring:** Add custom CloudWatch alarms
- **Networking:** Adjust CIDR blocks and subnet configurations
- **Security:** Modify security group rules and IAM policies

### Template-Specific Variables:
Each template includes detailed comments explaining:
- Required vs. optional variables
- Recommended values for different scenarios
- Cost implications of different choices
- Security considerations

## Migration Path

You can start with a smaller setup and migrate to larger ones:

1. **Small → Medium:** Enable caching and document database
2. **Medium → Large:** Add high availability and enhanced monitoring
3. **Any → Custom:** Use templates as starting points for custom configurations

## Getting Started

1. **Choose your template** based on the criteria above
2. **Copy the template:** `cp tfvars/example-{size}-setup.tfvars tfvars/your-project.tfvars`
3. **Review variables:** Open the file and understand each configuration option
4. **Replace placeholders:** Update all `CHANGE_ME_PLEASE` values
5. **Deploy:** Follow the main deployment workflow

## Need Help Deciding?

Consider these questions:
- What's your budget range?
- Do you need high availability?
- Will you have significant traffic?
- Do you need caching for performance?
- Is this for development or production?
- Do you have compliance requirements?

**Still unsure?** Start with the **Medium** setup - it provides a good balance of features and cost, and can be scaled up or down as needed.

## AI-DLC Decision Support

For comprehensive template selection with structured analysis:

**Ask Kiro:** "Using AI-DLC, help me select the optimal template for my [application type] with [specific requirements]"

**AI-DLC will provide:**
- **Inception Phase**: Requirements gathering and constraint analysis
- **Architecture Analysis**: Template comparison with your specific needs
- **Cost Modeling**: Detailed cost projections and optimization recommendations
- **Risk Assessment**: Availability, performance, and operational risk evaluation
- **Migration Planning**: Future scaling and upgrade path recommendations

This ensures you select the template that best aligns with your current needs and future growth plans.