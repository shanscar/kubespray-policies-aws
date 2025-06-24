# Kubespray AWS IAM Policies

A comprehensive collection of AWS IAM policies designed specifically for Kubespray Kubernetes cluster deployment and management.

## 📋 Table of Contents

- [Overview](#overview)
- [Policy Architecture](#policy-architecture)
- [Core Policies](#core-policies)
- [Admin Policies](#admin-policies)
- [Installation & Usage](#installation--usage)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Overview

This repository contains 13 carefully crafted AWS IAM policies, divided into two main categories:
- **7 Core Policies** - For daily Kubespray cluster deployment and operations
- **6 Admin Policies** - For cluster management and resource cleanup

These policies follow the principle of least privilege and include appropriate conditional restrictions to ensure security.

## Policy Architecture

### Core vs Admin Concept

| Type | Purpose | Permission Scope | Target Users |
|------|---------|------------------|--------------|
| **Core Policies** | Cluster deployment and daily operations | Create, modify, query resources | Developers, DevOps Engineers |
| **Admin Policies** | Cluster management and resource cleanup | Delete, advanced management operations | Platform Administrators, Senior Operations |

### Security Design Principles

- ✅ **Conditional Restrictions** - Use resource tags and naming conventions to limit permission scope
- ✅ **MFA Requirements** - Critical operations require multi-factor authentication
- ✅ **Environment Isolation** - Additional protection for production environments
- ✅ **Least Privilege** - Grant only necessary permissions

## Core Policies

### 1. KubesprayCore-EC2Management
**Function:** EC2 instance lifecycle management
```json
Permissions include:
- EC2 instance creation, start, stop, reboot
- Instance attribute modification
- IAM instance profile association
- EBS volume management
- Snapshot and key pair management
```

**Key Features:**
- Supports instance creation with proper tagging
- Allows instance state management for operational needs
- Enables EBS volume attachment/detachment for storage management
- Restricted by project tags to prevent cross-project interference

### 2. KubesprayCore-NetworkingCreate
**Function:** Network infrastructure creation and configuration
```json
Permissions include:
- VPC, subnet creation and modification
- Route table and route management
- Internet gateway management
- Security group rule configuration
- Network ACL management
```

**Key Features:**
- Comprehensive VPC setup capabilities
- Security group management for cluster communications
- Internet gateway setup for external access
- Conditional restrictions based on project tags

### 3. KubesprayCore-ConnectionEndpoints
**Function:** Connection endpoints and remote access
```json
Permissions include:
- VPC endpoint creation and management
- EC2 Instance Connect endpoints
- SSM Session Manager access
- SSH tunnel establishment
- Remote command execution
```

**Key Features:**
- Secure remote access without direct SSH
- VPC endpoints for private service access
- SSM integration for secure shell access
- Support for Instance Connect endpoints

### 4. KubesprayCore-IAMRoles
**Function:** IAM role and instance profile management
```json
Permissions include:
- Creating Kubespray-related IAM roles
- Instance profile management
- Role policy attachment and detachment
- Role permission queries
- PassRole permissions (restricted to specific services)
```

**Key Features:**
- Restricted role creation with naming conventions
- Secure PassRole permissions limited to EC2, ELB, and Auto Scaling
- Instance profile management for EC2 integration
- Project-based resource isolation

### 5. KubesprayCore-LoadBalancingAutoScaling
**Function:** Load balancing and auto scaling
```json
Permissions include:
- Elastic Load Balancer creation and configuration
- Target group management
- Listener and rule configuration
- Auto Scaling group management
- Launch configuration management
```

**Key Features:**
- Full ELB v2 (ALB/NLB) support
- Target group health check configuration
- Auto Scaling integration for node management
- Load balancer tagging for resource tracking

### 6. KubesprayCore-ServiceLinkedRoles
**Function:** AWS service-linked role management
```json
Permissions include:
- Creating service-linked roles for AWS services
- Supported services: Auto Scaling, ELB, VPC Endpoints, EC2, SSM
- Service-linked role queries and management
```

**Key Features:**
- Automated service-linked role creation
- Support for essential AWS services
- No tag conditions required (AWS-managed resources)
- Service name restrictions for security

### 7. KubesprayCore-ContainerRegistry
**Function:** Container image registry access
```json
Permissions include:
- ECR authorization token retrieval
- Container image push and pull
- Repository management (restricted by project tags)
- CloudWatch logs basic operations
- Container monitoring data upload
```

**Key Features:**
- Full ECR read access for image pulling
- Restricted write access based on project tags
- CloudWatch integration for container monitoring
- Support for both public and private repositories

## Admin Policies

### 1. KubesprayAdmin-ResourceCleanup
**Function:** Infrastructure resource cleanup
```json
Permissions include:
- EC2 resource deletion (volumes, snapshots, key pairs)
- Network resource cleanup (VPC, subnets, security groups)
- VPC endpoint and connection endpoint deletion
- Conditions: Must have project tags + Terraform managed tags
```

**Key Features:**
- Comprehensive infrastructure teardown
- Double-safety with project and ManagedBy tags
- Network resource dependencies handling
- Prevents accidental deletion of non-Kubespray resources

### 2. KubesprayAdmin-IAMCleanup
**Function:** IAM resource cleanup and management
```json
Permissions include:
- IAM role and instance profile deletion
- Service-linked role cleanup
- Role policy management
- Conditions: Naming conventions + MFA requirements (specific operations)
```

**Key Features:**
- Safe IAM resource deletion with naming restrictions
- MFA requirement for critical operations
- Service-linked role management
- Prevention of boundary policy modification

### 3. KubesprayAdmin-LoadBalancerCleanup
**Function:** Load balancer and auto scaling cleanup
```json
Permissions include:
- Load balancer deletion
- Target group and listener cleanup
- Auto Scaling group deletion
- Conditions: Project tags + Kubernetes cluster tags
```

**Key Features:**
- Complete load balancer teardown
- Auto Scaling group cleanup
- Kubernetes-aware resource identification
- Tag-based safety controls

### 4. KubesprayAdmin-AdvancedMonitoring
**Function:** Monitoring and logging management
```json
Permissions include:
- CloudWatch alarm and dashboard deletion
- Log group and log stream management
- SSM advanced operations (associations, parameter management)
- Conditions: Resource tag restrictions
```

**Key Features:**
- CloudWatch resource cleanup
- Log management with naming conventions
- SSM parameter and association management
- Resource-based access control

### 5. KubesprayAdmin-TerraformStateManagement
**Function:** Terraform state management
```json
Permissions include:
- S3 state file access and management
- DynamoDB state locking table operations
- Multi-project state isolation support
- Conditions: Naming conventions + project tags
```

**Key Features:**
- Secure Terraform state backend access
- DynamoDB lock table management
- Project-based state isolation
- S3 bucket and object level controls

### 6. KubesprayAdmin-EmergencyCleanup
**Function:** Emergency resource cleanup
```json
Permissions include:
- Emergency instance termination (dev/test environments)
- Extended service-linked role management
- Conditions: Strict MFA requirements + environment restrictions
```

**Key Features:**
- Emergency response capabilities
- Environment-restricted operations
- Mandatory MFA for all operations
- Extended AWS service support

## Installation & Usage

### 1. Policy Deployment

```bash
# Clone repository
git clone https://github.com/shanscar/kubespray-policies-aws.git
cd kubespray-policies-aws

# Create Core Policies using AWS CLI
for policy in KubesprayCore-*.json; do
    policy_name=$(basename "$policy" .json)
    aws iam create-policy \
        --policy-name "$policy_name" \
        --policy-document file://"$policy" \
        --description "Kubespray Core Policy: $policy_name"
done

# Create Admin Policies
for policy in KubesprayAdmin-*.json; do
    policy_name=$(basename "$policy" .json)
    aws iam create-policy \
        --policy-name "$policy_name" \
        --policy-document file://"$policy" \
        --description "Kubespray Admin Policy: $policy_name"
done
```

### 2. User Role Configuration

#### Developer Configuration
```bash
# Attach Core Policies for development work
aws iam attach-user-policy --user-name kubespray-developer \
    --policy-arn arn:aws:iam::ACCOUNT-ID:policy/KubesprayCore-EC2Management

aws iam attach-user-policy --user-name kubespray-developer \
    --policy-arn arn:aws:iam::ACCOUNT-ID:policy/KubesprayCore-NetworkingCreate

aws iam attach-user-policy --user-name kubespray-developer \
    --policy-arn arn:aws:iam::ACCOUNT-ID:policy/KubesprayCore-ConnectionEndpoints

aws iam attach-user-policy --user-name kubespray-developer \
    --policy-arn arn:aws:iam::ACCOUNT-ID:policy/KubesprayCore-IAMRoles
```

#### DevOps Engineer Configuration
```bash
# Attach all Core Policies + selected Admin Policies
CORE_POLICIES=(
    "KubesprayCore-EC2Management"
    "KubesprayCore-NetworkingCreate"
    "KubesprayCore-ConnectionEndpoints"
    "KubesprayCore-IAMRoles"
    "KubesprayCore-LoadBalancingAutoScaling"
    "KubesprayCore-ServiceLinkedRoles"
    "KubesprayCore-ContainerRegistry"
)

SELECTED_ADMIN_POLICIES=(
    "KubesprayAdmin-TerraformStateManagement"
    "KubesprayAdmin-AdvancedMonitoring"
)

for policy in "${CORE_POLICIES[@]}" "${SELECTED_ADMIN_POLICIES[@]}"; do
    aws iam attach-user-policy --user-name kubespray-devops \
        --policy-arn arn:aws:iam::ACCOUNT-ID:policy/"$policy"
done
```

#### Platform Administrator Configuration
```bash
# Attach all Core + Admin Policies
ALL_POLICIES=(
    "KubesprayCore-EC2Management"
    "KubesprayCore-NetworkingCreate"
    "KubesprayCore-ConnectionEndpoints"
    "KubesprayCore-IAMRoles"
    "KubesprayCore-LoadBalancingAutoScaling"
    "KubesprayCore-ServiceLinkedRoles"
    "KubesprayCore-ContainerRegistry"
    "KubesprayAdmin-ResourceCleanup"
    "KubesprayAdmin-IAMCleanup"
    "KubesprayAdmin-LoadBalancerCleanup"
    "KubesprayAdmin-AdvancedMonitoring"
    "KubesprayAdmin-TerraformStateManagement"
    "KubesprayAdmin-EmergencyCleanup"
)

for policy in "${ALL_POLICIES[@]}"; do
    aws iam attach-user-policy --user-name kubespray-admin \
        --policy-arn arn:aws:iam::ACCOUNT-ID:policy/"$policy"
done
```

### 3. Resource Tagging Strategy

All resources must include the following tags for policy conditions to work:

```json
{
    "Project": "kubespray-production",
    "Environment": "prod|dev|staging|test",
    "ManagedBy": "terraform",
    "Team": "platform",
    "Owner": "devops-team"
}
```

#### Terraform Tagging Example
```hcl
# terraform/locals.tf
locals {
  common_tags = {
    Project     = "kubespray-${var.environment}"
    Environment = var.environment
    ManagedBy   = "terraform"
    Team        = "platform"
    Owner       = "devops-team"
  }
  
  kubernetes_tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "owned"
  }
}

# terraform/ec2.tf
resource "aws_instance" "k8s_master" {
  count                  = var.master_count
  ami                    = var.ami_id
  instance_type         = var.master_instance_type
  subnet_id             = aws_subnet.private[count.index % length(aws_subnet.private)].id
  vpc_security_group_ids = [aws_security_group.k8s_master.id]
  
  tags = merge(
    local.common_tags,
    local.kubernetes_tags,
    {
      Name = "k8s-master-${count.index + 1}"
      Role = "master"
      Type = "control-plane"
    }
  )
}

# terraform/vpc.tf
resource "aws_vpc" "kubespray" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(
    local.common_tags,
    {
      Name = "kubespray-vpc"
      Type = "network"
    }
  )
}
```

### 4. Permission Boundaries (Recommended)

For enhanced security, consider implementing permission boundaries:

#### Developer Boundary
```bash
# Create permission boundary for developers
aws iam create-policy \
    --policy-name KubesprayBoundary-Developer \
    --policy-document file://boundaries/developer-boundary.json

# Apply boundary to developer users
aws iam put-user-permissions-boundary \
    --user-name kubespray-developer \
    --permissions-boundary arn:aws:iam::ACCOUNT-ID:policy/KubesprayBoundary-Developer
```

## Best Practices

### 1. Layered Permission Usage

| Role | Core Policies | Admin Policies | Use Case |
|------|---------------|----------------|----------|
| **Developer** | Selective usage | ❌ None | Cluster deployment and testing |
| **DevOps Engineer** | All Core | Selected Admin | Daily operations management |
| **Platform Administrator** | All Core | All Admin | Complete cluster lifecycle |

### 2. Security Recommendations

#### Multi-Factor Authentication
```bash
# Enable MFA for all admin users
aws iam create-virtual-mfa-device \
    --virtual-mfa-device-name kubespray-admin-mfa \
    --outfile QRCode.png \
    --bootstrap-method QRCodePNG

# Associate MFA device with user
aws iam enable-mfa-device \
    --user-name kubespray-admin \
    --serial-number arn:aws:iam::ACCOUNT-ID:mfa/kubespray-admin-mfa \
    --authentication-code1 123456 \
    --authentication-code2 654321
```

#### Regular Access Reviews
```bash
# Generate IAM credential report
aws iam generate-credential-report

# Get credential report
aws iam get-credential-report --output text --query 'Content' | base64 -d > iam-credential-report.csv

# Check last used dates
aws iam get-access-key-last-used --access-key-id AKIA...
```

#### Environment Separation
```hcl
# Different tag strategies for different environments
variable "environment_tags" {
  type = map(map(string))
  default = {
    "dev" = {
      Project     = "kubespray-dev"
      Environment = "development"
      CostCenter  = "engineering"
    }
    "staging" = {
      Project     = "kubespray-staging"
      Environment = "staging"
      CostCenter  = "engineering"
    }
    "prod" = {
      Project     = "kubespray-production"
      Environment = "production"
      CostCenter  = "production"
    }
  }
}
```

### 3. Monitoring and Auditing

#### CloudTrail Integration
```bash
# Monitor policy usage
aws logs filter-log-events \
    --log-group-name CloudTrail/kubespray \
    --filter-pattern "{ $.eventName = AttachUserPolicy || $.eventName = DetachUserPolicy }"
```

#### Policy Simulator
```bash
# Test policy permissions
aws iam simulate-principal-policy \
    --policy-source-arn arn:aws:iam::ACCOUNT-ID:user/kubespray-developer \
    --action-names ec2:RunInstances \
    --resource-arns arn:aws:ec2:us-west-2:ACCOUNT-ID:instance/*
```

### 4. Naming Conventions

#### IAM Role Naming
```
Pattern: kubespray-{environment}-{component}-role
Examples:
- kubespray-prod-master-role
- kubespray-dev-worker-role
- kubespray-staging-etcd-role
```

#### Resource Naming
```
EC2 Instances: k8s-{role}-{index}
VPC: kubespray-{environment}-vpc
Subnets: kubespray-{environment}-{type}-{az}
Security Groups: kubespray-{environment}-{component}-sg
```

## Troubleshooting

### Common Issues

#### 1. Permission Denied Errors
```bash
# Check resource tags
aws ec2 describe-instances \
    --instance-ids i-1234567890abcdef0 \
    --query 'Reservations[*].Instances[*].Tags'

# Verify IAM role naming
aws iam list-roles --query 'Roles[?starts_with(RoleName, `kubespray`)]'

# Check MFA status for admin operations
aws sts get-session-token --duration-seconds 3600
```

#### 2. Terraform State Access Failures
```bash
# Verify S3 bucket access
aws s3 ls s3://your-terraform-state-bucket/kubespray-prod/

# Check DynamoDB lock table
aws dynamodb describe-table --table-name terraform-state-lock

# Verify lock key format
aws dynamodb scan --table-name terraform-state-lock \
    --filter-expression "begins_with(LockID, :prefix)" \
    --expression-attribute-values '{":prefix":{"S":"kubespray"}}'
```

#### 3. Service Linked Role Creation Issues
```bash
# Check if SLR already exists
aws iam get-role --role-name AWSServiceRoleForAutoScaling

# List available services for SLR
aws iam list-roles --query 'Roles[?contains(RoleName, `AWSServiceRole`)]'

# Verify service support in region
aws ec2 describe-regions --query 'Regions[?RegionName==`us-west-2`]'
```

#### 4. ECR Access Problems
```bash
# Get ECR login token
aws ecr get-login-password --region us-west-2

# Check repository tags
aws ecr list-tags-for-resource \
    --resource-arn arn:aws:ecr:us-west-2:ACCOUNT-ID:repository/kubespray-app

# Verify repository permissions
aws ecr describe-repositories --repository-names kubespray-app
```

### Policy Validation

#### JSON Syntax Validation
```bash
# Validate policy syntax
for policy in *.json; do
    echo "Validating $policy..."
    aws iam validate-policy-document --policy-document file://"$policy"
done
```

#### Policy Simulation
```bash
# Test specific actions
aws iam simulate-custom-policy \
    --policy-input-list file://KubesprayCore-EC2Management.json \
    --action-names ec2:RunInstances,ec2:TerminateInstances \
    --resource-arns arn:aws:ec2:*:*:instance/*
```

## Contributing

We welcome contributions! Please follow these guidelines:

### Development Workflow
1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-policy`
3. Make your changes and test thoroughly
4. Commit with descriptive messages
5. Submit a Pull Request

### Policy Development Guidelines

#### 1. Naming Conventions
- Core Policies: `KubesprayCore-{FunctionDescription}`
- Admin Policies: `KubesprayAdmin-{FunctionDescription}`
- Use PascalCase for policy names
- Use kebab-case for file names

#### 2. Policy Structure
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DescriptiveName",
            "Effect": "Allow|Deny",
            "Action": ["service:action"],
            "Resource": "arn:aws:service:region:account:resource",
            "Condition": {
                "StringLike": {
                    "aws:ResourceTag/Project": ["kubespray*"]
                }
            }
        }
    ]
}
```

#### 3. Testing Requirements
- Validate JSON syntax
- Test with AWS Policy Simulator
- Verify conditional logic
- Document any special requirements

#### 4. Documentation Standards
- Update README.md for new policies
- Include permission matrix
- Document any prerequisites
- Provide usage examples

### Code Review Process
1. All changes require review
2. Test in non-production environment first
3. Verify security implications
4. Check for breaking changes
5. Update version tags appropriately

## Security Considerations

### Threat Model
- **Insider Threats**: Mitigated by permission boundaries and MFA
- **Credential Compromise**: Limited by conditional restrictions
- **Privilege Escalation**: Prevented by careful PassRole restrictions
- **Cross-Account Access**: Controlled by resource ARN patterns

### Security Controls
- ✅ **Conditional Access**: Resource tags and naming conventions
- ✅ **Multi-Factor Authentication**: Required for destructive operations
- ✅ **Time-Based Controls**: Session duration limits
- ✅ **Network Controls**: VPC endpoint usage where applicable
- ✅ **Audit Trail**: CloudTrail integration for all actions

## Support

For questions, issues, or contributions:

- 📝 **Issues**: [GitHub Issues](https://github.com/shanscar/kubespray-policies-aws/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/shanscar/kubespray-policies-aws/discussions)
- 📚 **Documentation**: [Kubespray Official Docs](https://kubespray.io/)
- 🔒 **Security**: Report security issues privately to maintainers

## Changelog

### v1.0.0 (Current)
- Initial release with 13 comprehensive policies
- Core and Admin policy separation
- Comprehensive tagging strategy
- Multi-environment support

---

**⚠️ Important Security Notice:** Please review and test these policies thoroughly in your environment before production use. Adjust conditional restrictions according to your specific security requirements and compliance needs.

**📋 Policy Matrix:** For a complete mapping of AWS services and actions covered by each policy, see the [Policy Matrix](docs/policy-matrix.md).

**🚀 Quick Start:** For rapid deployment, check out our [Quick Start Guide](docs/quick-start.md).
