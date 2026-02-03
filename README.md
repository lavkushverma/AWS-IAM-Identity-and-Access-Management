# 🔐 AWS IAM (Identity and Access Management) - Complete Guide

![AWS](https://img.shields.io/badge/AWS-IAM-orange?logo=amazon-aws)
![Security](https://img.shields.io/badge/Security-IAM-red?logo=security)

Complete guide to understanding and implementing AWS IAM (Identity and Access Management).

---

## 📋 Table of Contents

- [What is IAM?](#what-is-iam)
- [Why IAM?](#why-iam)
- [IAM Components](#iam-components)
- [How IAM Works](#how-iam-works)
- [Authentication vs Authorization](#authentication-vs-authorization)
- [Hands-On Examples](#hands-on-examples)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Security Features](#security-features)
- [Troubleshooting](#troubleshooting)

---

## 🎯 What is IAM?

**AWS IAM (Identity and Access Management)** is a web service that helps you securely control access to AWS resources.

### Key Features:
- ✅ **Free** - No additional charge
- ✅ **Global** - Not region-specific
- ✅ **Shared Access** - Multiple users in one account
- ✅ **Granular Permissions** - Fine-grained access control
- ✅ **Secure** - MFA, password policies, encryption
- ✅ **Integration** - Works with all AWS services

---

## 🤔 Why IAM?

### Without IAM:
```
❌ Everyone uses root account (dangerous!)
❌ No access control
❌ Can't track who did what
❌ Shared passwords
❌ No principle of least privilege
```

### With IAM:
```
✅ Individual user accounts
✅ Granular permissions
✅ Audit trails (who did what, when)
✅ Temporary credentials
✅ MFA for security
✅ Least privilege access
```

---

## 🧩 IAM Components

### 1. **Users** 👤
An individual person or application that interacts with AWS.

```
┌─────────────────────┐
│   IAM User          │
├─────────────────────┤
│ Name: john-doe      │
│ Access: Console +   │
│         Programmatic│
│ Credentials:        │
│  - Password         │
│  - Access Keys      │
└─────────────────────┘
```

**When to use:**
- Individual employees
- Applications running outside AWS
- Third-party services

**Example:**
```bash
# Create user
aws iam create-user --user-name john-doe

# Create access key
aws iam create-access-key --user-name john-doe
```

---

### 2. **Groups** 👥
Collection of IAM users with same permissions.

```
┌─────────────────────────────────┐
│        Developers Group         │
├─────────────────────────────────┤
│ Users:                          │
│  ├─ alice                       │
│  ├─ bob                         │
│  └─ charlie                     │
│                                 │
│ Permissions:                    │
│  ├─ EC2 Full Access            │
│  ├─ S3 Read Only               │
│  └─ CloudWatch Logs            │
└─────────────────────────────────┘
```

**Benefits:**
- Easier management
- Consistent permissions
- Less errors

**Example:**
```bash
# Create group
aws iam create-group --group-name Developers

# Attach policy to group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

# Add user to group
aws iam add-user-to-group \
  --user-name alice \
  --group-name Developers
```

---

### 3. **Roles** 🎭
Temporary credentials for AWS services or users.

```
┌─────────────────────────────────┐
│      EC2-S3-Access-Role         │
├─────────────────────────────────┤
│ Trust Policy:                   │
│  - EC2 service can assume       │
│                                 │
│ Permissions:                    │
│  - S3 Read/Write               │
│  - CloudWatch Logs             │
│                                 │
│ Use Case:                       │
│  - EC2 instances access S3     │
└─────────────────────────────────┘
```

**When to use:**
- EC2 instances accessing AWS services
- Lambda functions
- Cross-account access
- Federated users
- External identities

**Example:**
```bash
# Create role
aws iam create-role \
  --role-name EC2-S3-Access \
  --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

**Trust Policy (trust-policy.json):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

### 4. **Policies** 📜
JSON documents that define permissions.

```
┌─────────────────────────────────┐
│         IAM Policy              │
├─────────────────────────────────┤
│ {                               │
│   "Version": "2012-10-17",      │
│   "Statement": [{               │
│     "Effect": "Allow",          │
│     "Action": "s3:GetObject",   │
│     "Resource": "arn:..."       │
│   }]                            │
│ }                               │
└─────────────────────────────────┘
```

#### **Types of Policies:**

##### **a) AWS Managed Policies** (Recommended)
Created and maintained by AWS.

```bash
# Common AWS Managed Policies
arn:aws:iam::aws:policy/AdministratorAccess
arn:aws:iam::aws:policy/PowerUserAccess
arn:aws:iam::aws:policy/ReadOnlyAccess
arn:aws:iam::aws:policy/AmazonEC2FullAccess
arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# List all AWS managed policies
aws iam list-policies --scope AWS
```

##### **b) Customer Managed Policies**
Created and maintained by you.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Environment": "Dev"
        }
      }
    }
  ]
}
```

##### **c) Inline Policies**
Embedded directly in a user, group, or role.

```bash
# Create inline policy
aws iam put-user-policy \
  --user-name john-doe \
  --policy-name MyInlinePolicy \
  --policy-document file://policy.json
```

---

### 5. **Permissions** 🔑

How permissions work:

```
User/Role/Group
      ↓
   Policy (JSON)
      ↓
   Permissions (Allow/Deny)
      ↓
   AWS Resources (S3, EC2, etc.)
```

**Permission Evaluation Logic:**

```
1. By default, everything is DENIED
2. An explicit ALLOW overrides default DENY
3. An explicit DENY overrides any ALLOW
4. If multiple policies apply, ALL are evaluated
```

**Example:**
```
Policy 1: Allow EC2 * 
Policy 2: Deny EC2 Terminate
Result: Can do everything except terminate
```

---

## 🔄 How IAM Works

### **Complete Flow:**

```
┌──────────────┐
│   User       │ 1. Request
│  (Alice)     ├─────────────┐
└──────────────┘             │
                             ↓
                    ┌────────────────┐
                    │  IAM Service   │
                    │  Evaluates:    │
                    │  - Who?        │ 2. Authenticate
                    │  - What?       │    & Authorize
                    │  - Where?      │
                    │  - When?       │
                    └────────┬───────┘
                             │
                             ↓
                    ┌────────────────┐
                    │ AWS Resource   │ 3. Access
                    │   (S3, EC2)    │    Granted/Denied
                    └────────────────┘
```

### **Step-by-Step Example:**

**Scenario:** Alice wants to upload a file to S3

```
1. Authentication (Who is this?)
   ├─ Alice provides credentials
   ├─ IAM verifies username/password
   └─ Alice is authenticated ✅

2. Authorization (What can they do?)
   ├─ IAM checks Alice's policies
   ├─ Found: S3 PutObject permission
   └─ Action is authorized ✅

3. Resource Access
   ├─ Request sent to S3
   └─ File uploaded successfully ✅
```

---

## 🔐 Authentication vs Authorization

### **Authentication** (Who are you?)

```
┌─────────────────────────────┐
│     AUTHENTICATION          │
│  "Who are you?"             │
├─────────────────────────────┤
│ Methods:                    │
│  ✓ Username + Password      │
│  ✓ Access Key + Secret Key  │
│  ✓ MFA (Multi-Factor)       │
│  ✓ SSO (Single Sign-On)     │
│  ✓ Federation (SAML, OIDC)  │
└─────────────────────────────┘
```

**Example:**
```bash
# Console login
Username: alice
Password: ********
MFA Code: 123456
Status: ✅ Authenticated

# Programmatic access
Access Key: AKIAIOSFODNN7EXAMPLE
Secret Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Status: ✅ Authenticated
```

---

### **Authorization** (What can you do?)

```
┌─────────────────────────────┐
│     AUTHORIZATION           │
│  "What can you do?"         │
├─────────────────────────────┤
│ Based on:                   │
│  ✓ IAM Policies             │
│  ✓ Resource Policies        │
│  ✓ Permission Boundaries    │
│  ✓ Service Control Policies │
└─────────────────────────────┘
```

**Example:**
```
User: alice
Authenticated: ✅

Trying to: Create EC2 instance
Permission: ✅ ALLOWED (has EC2:RunInstances)

Trying to: Delete S3 bucket
Permission: ❌ DENIED (no S3:DeleteBucket)
```

---

## 💡 Hands-On Examples

### **Example 1: Create User with Console Access**

```bash
# Step 1: Create user
aws iam create-user --user-name john-doe

# Step 2: Create login profile (console password)
aws iam create-login-profile \
  --user-name john-doe \
  --password 'MySecureP@ssw0rd!' \
  --password-reset-required

# Step 3: Attach policy
aws iam attach-user-policy \
  --user-name john-doe \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Step 4: Enable MFA (recommended)
# Done through AWS Console

# Console URL:
# https://123456789012.signin.aws.amazon.com/console
```

---

### **Example 2: Create User with Programmatic Access**

```bash
# Create user
aws iam create-user --user-name app-user

# Create access key
aws iam create-access-key --user-name app-user

# Output:
# AccessKeyId: AKIAIOSFODNN7EXAMPLE
# SecretAccessKey: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# Attach policy
aws iam attach-user-policy \
  --user-name app-user \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

# Use credentials
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# Test
aws s3 ls
```

---

### **Example 3: Create Group with Multiple Users**

```bash
# Create group
aws iam create-group --group-name Developers

# Attach policies to group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create users
aws iam create-user --user-name alice
aws iam create-user --user-name bob

# Add users to group
aws iam add-user-to-group --user-name alice --group-name Developers
aws iam add-user-to-group --user-name bob --group-name Developers

# Now alice and bob have EC2 Full + S3 Read permissions
```

---

### **Example 4: Create Role for EC2 to Access S3**

```bash
# Step 1: Create trust policy
cat > trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Step 2: Create role
aws iam create-role \
  --role-name EC2-S3-Access-Role \
  --assume-role-policy-document file://trust-policy.json

# Step 3: Attach permissions
aws iam attach-role-policy \
  --role-name EC2-S3-Access-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

# Step 4: Create instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-Profile

# Step 5: Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Profile \
  --role-name EC2-S3-Access-Role

# Step 6: Attach to EC2 instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-1234567890abcdef0 \
  --iam-instance-profile Name=EC2-S3-Profile
```

**Now EC2 instance can access S3 without hardcoded credentials!**

---

### **Example 5: Custom Policy - Start/Stop EC2 Only**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2StartStop",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus"
      ],
      "Resource": "*"
    },
    {
      "Sid": "DenyTerminate",
      "Effect": "Deny",
      "Action": [
        "ec2:TerminateInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

**Apply:**
```bash
# Save as policy.json
aws iam create-policy \
  --policy-name EC2-StartStop-Only \
  --policy-document file://policy.json

# Attach to user
aws iam attach-user-policy \
  --user-name alice \
  --policy-arn arn:aws:iam::123456789012:policy/EC2-StartStop-Only
```

---

## 🎯 Common Use Cases

### **Use Case 1: Multi-Account Setup**

```
Organization
├── Root Account (Management)
│   └── IAM Roles for cross-account
├── Production Account
│   ├── Admin Role
│   └── Developer Role
├── Development Account
│   └── Developer Full Access
└── Testing Account
    └── Tester Role
```

**Implementation:**
```bash
# In Production Account
# Create role that trusts Dev account

cat > trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::111111111111:root"
    },
    "Action": "sts:AssumeRole"
  }]
}
EOF

aws iam create-role \
  --role-name CrossAccountAccess \
  --assume-role-policy-document file://trust-policy.json

# Users in Dev account can now assume this role
```

---

### **Use Case 2: Application Running on EC2**

```
EC2 Instance
  └── IAM Role
      ├── S3 Read/Write
      ├── DynamoDB Access
      ├── SQS Send/Receive
      └── CloudWatch Logs
```

**Benefits:**
- ✅ No hardcoded credentials
- ✅ Automatic credential rotation
- ✅ Temporary credentials
- ✅ Easy to manage

---

### **Use Case 3: CI/CD Pipeline**

```
GitHub Actions
  └── IAM User (or OIDC)
      └── Permissions:
          ├── ECR Push
          ├── ECS Deploy
          └── S3 Upload
```

**Setup:**
```bash
# Create user for CI/CD
aws iam create-user --user-name github-actions

# Create policy
cat > ci-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:PutImage"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name CI-CD-Policy \
  --policy-document file://ci-policy.json

aws iam attach-user-policy \
  --user-name github-actions \
  --policy-arn arn:aws:iam::123456789012:policy/CI-CD-Policy

# Create access key
aws iam create-access-key --user-name github-actions
```

---

### **Use Case 4: External Contractor Access**

```
Contractor (Temporary)
  └── IAM User
      ├── Limited permissions
      ├── MFA required
      ├── IP restriction
      └── Time-based access
```

**Policy with restrictions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::project-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        },
        "DateGreaterThan": {
          "aws:CurrentTime": "2024-01-01T00:00:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "2024-03-31T23:59:59Z"
        },
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```

---

## 🛡️ Security Features

### **1. Multi-Factor Authentication (MFA)**

```
Password + MFA Device = Enhanced Security

Types:
├── Virtual MFA (Google Authenticator, Authy)
├── Hardware MFA (YubiKey)
└── SMS (not recommended)
```

**Enable MFA:**
```bash
# Get MFA device serial
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name alice-mfa \
  --outfile QRCode.png \
  --bootstrap-method QRCodePNG

# Enable MFA
aws iam enable-mfa-device \
  --user-name alice \
  --serial-number arn:aws:iam::123456789012:mfa/alice-mfa \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

---

### **2. Password Policy**

```bash
# Set account password policy
aws iam update-account-password-policy \
  --minimum-password-length 14 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters \
  --allow-users-to-change-password \
  --max-password-age 90 \
  --password-reuse-prevention 5 \
  --hard-expiry
```

---

### **3. Access Keys Rotation**

```bash
# List access keys
aws iam list-access-keys --user-name alice

# Create new key
aws iam create-access-key --user-name alice

# Update applications with new key

# Delete old key
aws iam delete-access-key \
  --user-name alice \
  --access-key-id AKIAIOSFODNN7EXAMPLE
```

---

### **4. Credential Report**

```bash
# Generate report
aws iam generate-credential-report

# Download report
aws iam get-credential-report \
  --output text \
  --query Content \
  | base64 --decode > credentials.csv

# Shows:
# - Users
# - Password enabled
# - MFA active
# - Access keys age
# - Last used
```

---

### **5. Access Advisor**

```bash
# See last accessed services
aws iam generate-service-last-accessed-details \
  --arn arn:aws:iam::123456789012:user/alice

# Get results
aws iam get-service-last-accessed-details \
  --job-id <job-id>
```

---

## ✅ Best Practices

### **1. Root Account**
```
❌ Don't use root account for daily tasks
✅ Enable MFA on root
✅ Delete root access keys
✅ Use root only for:
   - Account settings
   - Billing
   - Account closure
```

### **2. Least Privilege**
```
❌ Don't give AdministratorAccess to everyone
✅ Start with minimum permissions
✅ Add permissions as needed
✅ Review and remove unused permissions
```

### **3. Use Groups**
```
❌ Don't attach policies directly to users
✅ Create groups (Admins, Developers, etc.)
✅ Add users to groups
✅ Manage permissions at group level
```

### **4. Use Roles for Applications**
```
❌ Don't embed access keys in code
✅ Use IAM roles for EC2/Lambda/ECS
✅ Use temporary credentials
✅ Automatic rotation
```

### **5. Enable MFA**
```
✅ For all users (especially privileged)
✅ For root account
✅ For production access
```

### **6. Rotate Credentials**
```
✅ Rotate access keys every 90 days
✅ Rotate passwords regularly
✅ Use AWS Secrets Manager for apps
```

### **7. Monitor and Audit**
```
✅ Enable CloudTrail
✅ Review credential reports
✅ Use Access Analyzer
✅ Set up alerts for suspicious activity
```

### **8. Use Policy Conditions**
```json
{
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": "203.0.113.0/24"
    },
    "StringEquals": {
      "aws:RequestedRegion": "us-east-1"
    }
  }
}
```

---

## 🐛 Troubleshooting

### **Issue 1: Access Denied**

```bash
# Check user's permissions
aws iam list-attached-user-policies --user-name alice
aws iam list-user-policies --user-name alice

# Check group permissions
aws iam list-groups-for-user --user-name alice
aws iam list-attached-group-policies --group-name Developers

# Simulate policy
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/alice \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/file.txt
```

---

### **Issue 2: Can't Assume Role**

```bash
# Check trust policy
aws iam get-role --role-name MyRole

# Verify user has sts:AssumeRole permission
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/alice \
  --action-names sts:AssumeRole \
  --resource-arns arn:aws:iam::123456789012:role/MyRole
```

---

### **Issue 3: MFA Not Working**

```bash
# List MFA devices
aws iam list-mfa-devices --user-name alice

# Resync MFA
aws iam resync-mfa-device \
  --user-name alice \
  --serial-number arn:aws:iam::123456789012:mfa/alice-mfa \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

---

## 📊 IAM Limits

| Resource | Default Limit |
|----------|---------------|
| Users per account | 5,000 |
| Groups per account | 300 |
| Roles per account | 1,000 |
| Policies per user | 10 managed |
| Policies per group | 10 managed |
| Policies per role | 10 managed |
| Policy size | 6,144 characters |
| Access keys per user | 2 |

---

## 🔗 Quick Reference Commands

```bash
# Users
aws iam list-users
aws iam create-user --user-name john
aws iam delete-user --user-name john

# Groups
aws iam list-groups
aws iam create-group --group-name Developers
aws iam add-user-to-group --user-name john --group-name Developers

# Roles
aws iam list-roles
aws iam create-role --role-name MyRole --assume-role-policy-document file://trust.json
aws iam delete-role --role-name MyRole

# Policies
aws iam list-policies --scope Local
aws iam create-policy --policy-name MyPolicy --policy-document file://policy.json
aws iam attach-user-policy --user-name john --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Access Keys
aws iam list-access-keys --user-name john
aws iam create-access-key --user-name john
aws iam delete-access-key --user-name john --access-key-id AKIAIOSFODNN7EXAMPLE

# MFA
aws iam list-mfa-devices --user-name john
aws iam enable-mfa-device --user-name john --serial-number arn --code1 123456 --code2 789012
```

---

## 📚 Additional Resources

- [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Policy Simulator](https://policysim.aws.amazon.com/)
- [AWS Security Blog](https://aws.amazon.com/blogs/security/)

---

## 🎓 Summary

**IAM in Simple Terms:**

```
IAM = Who can do What on Which resources

Who    → Users, Groups, Roles
What   → Permissions (defined in Policies)
Which  → AWS Resources (S3, EC2, etc.)
```

**Remember:**
- 🔐 Always use least privilege
- 🔑 Never share credentials
- 📱 Enable MFA
- 🔄 Rotate keys regularly
- 📊 Monitor and audit access
- 👥 Use roles instead of users when possible

---
**Happy Learning!** 😊
