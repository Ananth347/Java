# The Complete AWS Tutorial — From Zero to Job-Ready
### A hands-on, plain-English guide for a Java / Spring Boot backend developer

---

## Why this document exists

You already know Java, Spring Boot, and backend systems in a banking environment.
This guide does not re-teach you programming. It teaches you **AWS**, service by
service, in plain English, with **real code** (mostly Java SDK v2, AWS CLI, and
Spring Boot integrations) that you can type out yourself, run, break, and fix.

Every section follows the same pattern so your brain doesn't have to keep
re-learning "how to read this doc":

1. **What is it, in one sentence a 10-year-old could understand**
2. **Why does it exist / what problem does it solve**
3. **Key concepts you must know** (the vocabulary)
4. **Hands-on: Console + CLI + Java SDK** (with comments in every code block)
5. **Spring Boot integration** (where relevant)
6. **Real-world banking/fintech example** (since that's your domain)
7. **Common mistakes / gotchas**
8. **Interview questions and model answers**
9. **Cheat sheet** (quick revision before an interview)

---

## How to use this guide

- Don't just read it. Open an AWS Free Tier account and **type every command**.
- Read top to bottom the first time — the order is intentional (IAM first,
  because everything else needs permissions; VPC before EC2, because EC2 lives
  inside a VPC; and so on).
- Keep a scratch AWS account. Delete resources after every lab so you don't
  get billed. Each section reminds you what to delete.
- At the very end there is a **Capstone Project** that wires all 15 services
  (plus the extra ones we add) into one real banking-style system — the kind
  of architecture you'd be expected to explain in a Wipro/fintech interview.

---

## Table of Contents

1. Cloud Computing & AWS Fundamentals
2. IAM — Identity and Access Management
3. VPC — Virtual Private Cloud
4. EC2 — Elastic Compute Cloud
5. S3 — Simple Storage Service
6. RDS — Relational Database Service
7. CloudWatch — Monitoring & Observability
8. SQS — Simple Queue Service
9. SNS — Simple Notification Service
10. Lambda — Serverless Functions
11. API Gateway
12. Secrets Manager
13. Docker + ECR — Containerizing Your App
14. ECS — Elastic Container Service
15. Auto Scaling + Elastic Load Balancer
16. CodePipeline, CodeBuild, CodeDeploy — CI/CD
17. Bonus Must-Know Services
    - 17.1 Route 53 (DNS)
    - 17.2 CloudFront (CDN)
    - 17.3 DynamoDB (NoSQL)
    - 17.4 KMS (Key Management Service)
    - 17.5 Systems Manager (Parameter Store & Session Manager)
    - 17.6 CloudFormation (Infrastructure as Code)
    - 17.7 CloudTrail (Audit Logging)
    - 17.8 EventBridge (Event Bus)
    - 17.9 Step Functions (Workflow Orchestration)
    - 17.10 WAF & Shield, ACM (Security basics)
    - 17.11 Cost Explorer & Budgets
    - 17.12 EKS vs ECS (Kubernetes on AWS, brief)
18. Capstone Project — A Banking Microservices Platform on AWS
19. AWS Well-Architected Framework — The 6 Pillars
20. Interview Question Bank (all services, quick fire)
21. Glossary
22. Further Resources

---

# 1. Cloud Computing & AWS Fundamentals

## 1.1 What is "the cloud", really?

Forget the buzzwords for a second. Before cloud computing, if your bank wanted
a new application, someone had to:

1. Buy physical servers (takes weeks, costs lakhs of rupees)
2. Put them in a data center (needs power, cooling, security)
3. Install an OS, patch it, maintain it forever
4. Guess how much capacity you'll need (guess wrong = waste money or crash
   under load)

**The cloud** is just: *"Amazon already built thousands of data centers full
of servers, storage, and networking. You rent exactly what you need, by the
second or hour, through an API or website, and give it back when you're
done."*

That's it. AWS (Amazon Web Services) is the biggest of these cloud providers
(others are Azure = Microsoft, GCP = Google Cloud).

## 1.2 Why banks and fintechs use AWS

- **Elasticity**: Salary day traffic spike? Scale up in minutes, scale down
  after. You never "own" idle hardware.
- **Pay-as-you-go**: No huge upfront capital cost.
- **Global reach**: Deploy close to your customers (low latency) in multiple
  countries without building your own data centers there.
- **Managed services**: AWS runs the boring, hard stuff (patching a database
  engine, replicating data across data centers) so your team focuses on
  business logic — loan calculations, fraud detection, ledgers.
- **Compliance**: AWS has certifications (PCI-DSS, ISO 27001, SOC 2, RBI
  guidelines compatibility) that banks require from any infrastructure
  provider.

## 1.3 Core Vocabulary (learn this before anything else)

| Term | Plain English Meaning |
|---|---|
| **Region** | A physical geographic area with multiple data centers, e.g. `ap-south-1` (Mumbai), `us-east-1` (N. Virginia). Pick a region close to your users for low latency, and sometimes for legal/data-residency reasons (RBI wants Indian customer data to stay in India). |
| **Availability Zone (AZ)** | One or more physically separate data centers *inside* a Region, each with independent power/cooling/network. A Region usually has 3+ AZs. You spread your servers across AZs so if one data center catches fire, your app still runs. |
| **Edge Location** | A small AWS location (there are 400+) used by CloudFront (CDN) and Route 53 to serve content close to end users, even in cities that don't have a full Region. |
| **Service** | A capability AWS offers, e.g. "EC2" (virtual servers), "S3" (file storage). Each service is really just a set of APIs. |
| **Resource** | A specific *instance* of a service you created, e.g. one particular EC2 virtual machine, one particular S3 bucket. |
| **ARN (Amazon Resource Name)** | The unique "full address" of any AWS resource, like a file path but for cloud resources. Example: `arn:aws:s3:::my-bucket-name` or `arn:aws:iam::123456789012:user/ananth`. You will see ARNs *everywhere* — in permissions, in error messages, in code. |
| **Console** | The AWS website UI (console.aws.amazon.com) — good for learning and quick checks, bad for repeatable/automatable work. |
| **CLI** | `aws` command-line tool — scriptable, used in real DevOps work. |
| **SDK** | A library for a programming language (we'll use `software.amazon.awssdk` — AWS SDK for Java v2) so your Spring Boot app can call AWS APIs directly. |
| **IaC (Infrastructure as Code)** | Describing your infrastructure (servers, networks, databases) as code/config files instead of manually clicking in the console, so it's repeatable and version-controlled. (CloudFormation, Terraform, CDK.) |

## 1.4 Setting up your environment

### Step 1 — Create an AWS Free Tier account
Go to https://aws.amazon.com/free and sign up. You'll need a card (for
verification; Free Tier resources cost ₹0 if you stay within limits — but
**always** set a billing alarm, covered in Section 17.11).

### Step 2 — Never use the root user for daily work
The "root user" (the email/password you signed up with) has **unlimited
power** — it can delete your entire account. The very first thing you do in
AWS, always, on any account, is:

1. Log in as root **once**.
2. Enable MFA (Multi-Factor Authentication) on the root user.
3. Create an IAM user for yourself with admin permissions.
4. Log out of root. Never use root again unless absolutely required (e.g.
   closing the account, some billing settings).

This single habit is the #1 thing that separates a "toy AWS account" from
one that behaves like a real company's account — and it's also one of the
most common interview questions ("What's the first thing you do when you get
a new AWS account?").

### Step 3 — Install the AWS CLI

```bash
# On Ubuntu/Debian (or WSL on Windows)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# verify installation
aws --version
# Output looks like: aws-cli/2.15.30 Python/3.11.6 Linux/5.15.0 exe/x86_64.prompt
```

### Step 4 — Configure the CLI with your IAM user's credentials

```bash
aws configure
# It will ask you 4 things, one by one:
# AWS Access Key ID [None]: AKIAxxxxxxxxxxxxxxxx
# AWS Secret Access Key [None]: wJalrxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# Default region name [None]: ap-south-1        <-- Mumbai region, good for India
# Default output format [None]: json
```

> **Where do I get an Access Key?** IAM Console → Users → your user →
> Security credentials tab → Create access key. Keep it as secret as a bank
> password — anyone with it can spin up servers, read your S3 files, or run
> up your bill. **Never** commit it to Git. (More on this in the
> Secrets Manager section.)

### Step 5 — Add the AWS SDK to your Spring Boot project

```xml
<!-- pom.xml — using AWS SDK for Java v2 (the modern one; v1 is legacy) -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>bom</artifactId>
            <!-- BOM = Bill Of Materials, keeps all AWS SDK module versions in sync -->
            <version>2.25.60</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Add only the specific service modules you need, e.g.: -->
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>s3</artifactId>
    </dependency>
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>sqs</artifactId>
    </dependency>
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>sns</artifactId>
    </dependency>
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>secretsmanager</artifactId>
    </dependency>
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>dynamodb</artifactId>
    </dependency>
</dependencies>
```

Now let's start with the single most important service to understand first.

---

# 2. IAM — Identity and Access Management

## 2.1 What is it?

**IAM answers two questions for every single request made to AWS:
"Who are you?" (Authentication) and "What are you allowed to do?"
(Authorization).**

Think of AWS as a huge bank vault with millions of rooms (services). IAM is
the security desk that checks your ID badge (authentication) and your access
list (authorization) before letting you into any room, and even then, only
lets you do what your badge permits (read-only in this room, full access in
that one).

IAM itself is **free** and **global** (not tied to a Region).

## 2.2 Why it exists

Without IAM, anyone who got your AWS password could do *anything* — read
every customer's data, delete every server, run up a crores-worth bill
mining cryptocurrency on your account. IAM lets you follow the
**Principle of Least Privilege**: give every person and every application
*only* the exact permissions they need, nothing more.

This is the single most tested AWS concept in interviews, because it's the
one that causes real-world security breaches when done wrong.

## 2.3 Key Concepts (the vocabulary you must know cold)

| Concept | Meaning |
|---|---|
| **User** | Represents one human being (or sometimes one application) with long-term credentials (password for console, access keys for CLI/SDK). |
| **Group** | A collection of Users. You attach permissions to the Group, and every User in it inherits them. (E.g., a "Developers" group with EC2 + S3 read/write.) |
| **Role** | Like a User, but with **no permanent credentials**. Instead, it's "assumed" temporarily — by an EC2 instance, a Lambda function, another AWS account, or a human doing a one-time task. Roles are the *correct* way for applications to get AWS permissions (never hardcode keys in an app if you can use a Role instead). |
| **Policy** | A JSON document describing permissions: which Actions (e.g. `s3:GetObject`), on which Resources (e.g. a specific bucket ARN), are Allowed or Denied. |
| **Managed Policy** | A reusable, standalone Policy — either AWS-created (e.g. `AmazonS3ReadOnlyAccess`) or one you write yourself (Customer Managed Policy). |
| **Inline Policy** | A Policy embedded directly inside one User/Group/Role — not reusable, tightly coupled. Generally avoid unless there's a specific one-off reason. |
| **Trust Policy** | A special policy on a **Role** that says *who is allowed to assume this Role* (e.g. "the EC2 service can assume this role" or "AWS account 999999999999 can assume this role"). |
| **MFA (Multi-Factor Authentication)** | A second proof of identity (an OTP from an app like Google Authenticator) in addition to your password. |
| **Principal** | The "who" in a policy — a user, role, account, or service, that is making the request. |
| **Federation / SSO** | Letting people log into AWS using an existing identity (Google Workspace, Okta, Active Directory) instead of creating a separate IAM User for every employee. Real companies almost never create individual IAM Users for hundreds of employees — they use Federation via **IAM Identity Center** (formerly AWS SSO). |

## 2.4 Anatomy of a Policy (read this JSON slowly, it's simpler than it looks)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadOnlyOnOneBucket",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::bank-statements-bucket",
        "arn:aws:s3:::bank-statements-bucket/*"
      ]
    },
    {
      "Sid": "DenyDeleteAlways",
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::bank-statements-bucket/*"
    }
  ]
}
```

Read it out loud: *"Allow this identity to GetObject and ListBucket on the
bank-statements-bucket and everything inside it. But explicitly Deny
DeleteObject on anything inside it, no matter what."*

**Golden rule**: An explicit `Deny` **always** wins over an `Allow`, even if
another policy attached to the same user says Allow. This is tested in almost
every AWS interview.

## 2.5 Hands-on: Creating a User, Group, and Policy (Console + CLI)

### Via AWS CLI

```bash
# 1. Create a group for backend developers
aws iam create-group --group-name BackendDevelopers

# 2. Write a policy document to a local file
cat > s3-readonly-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::bank-statements-bucket",
        "arn:aws:s3:::bank-statements-bucket/*"
      ]
    }
  ]
}
EOF

# 3. Create the policy in AWS from that file
aws iam create-policy \
  --policy-name S3ReadOnlyBankStatements \
  --policy-document file://s3-readonly-policy.json
# This returns an ARN like:
# arn:aws:iam::123456789012:policy/S3ReadOnlyBankStatements
# COPY that ARN, you need it in the next step.

# 4. Attach the policy to the group
aws iam attach-group-policy \
  --group-name BackendDevelopers \
  --policy-arn arn:aws:iam::123456789012:policy/S3ReadOnlyBankStatements

# 5. Create a user and add them to the group
aws iam create-user --user-name ananth
aws iam add-user-to-group --user-name ananth --group-name BackendDevelopers

# 6. Give the user CLI access keys (returns AccessKeyId + SecretAccessKey — save immediately, secret is shown ONCE)
aws iam create-access-key --user-name ananth
```

## 2.6 Roles — the correct way to give an EC2 instance or Lambda permissions

**Never** put an IAM User's access key inside an EC2 instance or a Lambda's
code/environment variables. Instead, create a **Role**, attach a policy to
it, and attach the Role to the EC2 instance/Lambda. AWS automatically rotates
temporary credentials behind the scenes — your code just calls the SDK with
no keys configured, and it works.

```json
// Trust Policy for a Role — this says "the EC2 service is allowed to assume this role"
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

```bash
# Create the role with that trust policy
aws iam create-role \
  --role-name EC2-S3-ReadOnly-Role \
  --assume-role-policy-document file://trust-policy.json

# Attach an AWS-managed policy to it (AWS ships hundreds of ready-made policies)
aws iam attach-role-policy \
  --role-name EC2-S3-ReadOnly-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create an "instance profile" (a wrapper that lets EC2 actually use the role) and add the role to it
aws iam create-instance-profile --instance-profile-name EC2-S3-ReadOnly-Profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-ReadOnly-Profile \
  --role-name EC2-S3-ReadOnly-Role
```

Later, when you launch an EC2 instance (Section 4), you attach
`EC2-S3-ReadOnly-Profile` to it. Any Java code running on that instance using
the AWS SDK will *automatically* pick up these permissions — no keys
anywhere in your code. This is called the **credential provider chain**, and
it's how production Java apps almost always authenticate to AWS.

## 2.7 Java SDK — checking who you are (STS) and using default credentials

```java
// pom.xml needs: software.amazon.awssdk:sts

import software.amazon.awssdk.services.sts.StsClient;
import software.amazon.awssdk.services.sts.model.GetCallerIdentityResponse;

public class WhoAmI {
    public static void main(String[] args) {
        // StsClient.create() automatically looks for credentials in this order:
        // 1. Environment variables (AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY)
        // 2. Java system properties
        // 3. The ~/.aws/credentials file (what `aws configure` wrote for you)
        // 4. An IAM Role attached to the EC2/ECS/Lambda this code runs on
        // This search order is called the "Default Credentials Provider Chain"
        try (StsClient sts = StsClient.create()) {
            GetCallerIdentityResponse identity = sts.getCallerIdentity();
            System.out.println("Account ID: " + identity.account());
            System.out.println("My identity ARN: " + identity.arn());
            System.out.println("User ID: " + identity.userId());
        }
    }
}
```

## 2.8 Real banking example

Imagine three microservices in your bank's system:

- `loan-service` — needs to read/write to a specific RDS database and read
  PDF documents from one S3 bucket (loan applications).
- `notification-service` — needs to publish to SNS only.
- `fraud-detection-service` — needs read-only access to a DynamoDB table of
  transaction logs, nothing else.

Each gets **its own IAM Role**, attached to its own ECS Task or EC2 instance,
with a tightly scoped policy. If `notification-service` gets compromised by
a bug or attack, the attacker still cannot touch the loan database — because
that role was never given permission to it. This is "blast radius reduction"
— a phrase you should use in interviews.

## 2.9 Common mistakes / gotchas

- **Using root user for daily work.** Interviewers love asking "what's wrong
  with this setup" screenshots that show root access keys in use.
- **Access keys committed to GitHub.** This happens constantly in the real
  world. Always use `git-secrets` or pre-commit hooks, and prefer Roles over
  keys wherever possible.
- **`"Resource": "*"` policies** ("give access to everything") — a huge
  red flag; always scope to specific ARNs.
- **Forgetting MFA** on privileged users.
- **Confusing a Role's Trust Policy (who can assume it) with its Permission
  Policy (what it can do once assumed).** These are two different JSON
  documents attached to the same Role — a very common interview trip-up.

## 2.10 Interview Q&A

**Q: What's the difference between a Role and a User?**
A: A User has long-term credentials meant for a specific human or app and
exists permanently. A Role has no credentials of its own — it's *assumed*
temporarily by a trusted entity (an AWS service, another account, a
federated user), and AWS hands out short-lived, auto-rotating credentials
behind the scenes. Roles are safer and are the standard way applications get
AWS permissions.

**Q: Explicit Deny vs. Allow — which wins?**
A: Explicit Deny always wins, even over another statement's Allow, even
across multiple policies attached to the same identity.

**Q: How would you give a Lambda function permission to write to a specific
S3 bucket only?**
A: Create an Execution Role for the Lambda with a policy scoped to
`s3:PutObject` on that bucket's ARN only, attach the role to the Lambda at
creation time — never pass access keys as environment variables.

**Q: What's least privilege?**
A: Granting an identity the *minimum* set of permissions it needs to do its
job, nothing more — reduces the damage if that identity is ever compromised.

## 2.11 Cheat sheet

```text
IAM is GLOBAL (not per-region), FREE.
User        = permanent identity, has login/keys
Group       = bucket of Users, permissions inherited
Role        = temporary identity, assumed by trusted principal, NO permanent keys
Policy      = JSON: Effect + Action + Resource (+ optional Condition)
Trust Policy = on a Role only, decides WHO can assume it
Deny beats Allow, always.
Prefer Roles > Users for anything that's code (EC2/ECS/Lambda).
Always: MFA on root + humans, least privilege, no hardcoded keys.
```

---

# 3. VPC — Virtual Private Cloud

## 3.1 What is it?

**A VPC is your own private, isolated network inside AWS** — like having your
own office building inside a huge shared campus, with your own walls, your
own gates, and your own rules about who can walk in.

Every EC2 instance, every RDS database, every ECS container you launch has to
live *inside* a VPC. AWS gives every account a "Default VPC" per region so
beginners can launch things immediately, but real companies always design
their own custom VPCs.

## 3.2 Why it exists

Without network isolation, every customer's servers on AWS's shared physical
hardware would be able to see each other's traffic — a security nightmare,
especially for banking systems bound by regulations. A VPC guarantees your
resources are logically isolated, and you fully control:

- Which resources can talk to the public internet
- Which resources are completely private (e.g., your database should
  **never** be reachable from the internet directly)
- How traffic flows between your own tiers (web servers → app servers →
  database)

## 3.3 Key Concepts

| Concept | Meaning |
|---|---|
| **CIDR block** | The IP address range for your VPC, e.g. `10.0.0.0/16` gives you 65,536 IP addresses to allocate. Think of it as your network's total address space. |
| **Subnet** | A slice of your VPC's IP range, tied to one specific Availability Zone. You put resources into subnets. |
| **Public Subnet** | A subnet whose route table sends internet-bound traffic to an Internet Gateway — resources here *can* be reachable from the internet (if they also have a public IP + security group allows it). |
| **Private Subnet** | A subnet with no direct route to the internet. Your databases and internal app servers live here — the golden rule in banking systems: **database always in a private subnet**. |
| **Internet Gateway (IGW)** | The "front door" that connects a VPC to the public internet. One per VPC. |
| **NAT Gateway** | Lets resources in a *private* subnet initiate outbound connections to the internet (e.g., to download OS patches) *without* being reachable from the internet inbound. One-way door. |
| **Route Table** | Rules that decide where network traffic is directed, attached to subnets. |
| **Security Group (SG)** | A **stateful** virtual firewall at the instance level. "Stateful" means if you allow inbound traffic on a port, the matching outbound response is automatically allowed back, no extra rule needed. |
| **Network ACL (NACL)** | A **stateless** firewall at the subnet level (an extra optional layer). Stateless = you must explicitly allow both inbound AND outbound rules separately. Used less often than Security Groups in practice, but tested in interviews. |
| **VPC Peering** | Directly connecting two VPCs (even across accounts) so resources can talk to each other privately. |
| **VPC Endpoint** | Lets resources inside your VPC reach AWS services (like S3, DynamoDB) *without* going over the public internet at all — more secure and often cheaper. |

## 3.4 A picture in words: typical 3-tier banking VPC layout

```text
VPC: 10.0.0.0/16   (Region: ap-south-1, Mumbai)
│
├── Availability Zone: ap-south-1a
│   ├── Public Subnet   10.0.1.0/24   -> Load Balancer lives here
│   ├── Private Subnet  10.0.11.0/24  -> App servers / ECS tasks live here
│   └── Private Subnet  10.0.21.0/24  -> RDS database lives here (isolated, no NAT even)
│
└── Availability Zone: ap-south-1b   (mirror of above, for high availability)
    ├── Public Subnet   10.0.2.0/24
    ├── Private Subnet  10.0.12.0/24
    └── Private Subnet  10.0.22.0/24

Internet Gateway  <-> Public Subnets (for the Load Balancer to be reachable)
NAT Gateway (in Public Subnet) <-> Private App Subnets (so app servers can call out for OS updates, third-party APIs)
Database Subnets have NO route to the internet at all, inbound or outbound.
```

This "two AZs minimum" pattern is exactly what interviewers expect you to
draw when asked "design a highly available web application on AWS."

## 3.5 Hands-on: Building this VPC via CLI

```bash
# 1. Create the VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=banking-vpc}]'
# Note the returned VpcId, e.g. vpc-0abc123456

VPC_ID=vpc-0abc123456

# 2. Create subnets (repeat pattern per AZ)
aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 \
  --availability-zone ap-south-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-1a}]'

aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.11.0/24 \
  --availability-zone ap-south-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=private-app-1a}]'

aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.21.0/24 \
  --availability-zone ap-south-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=private-db-1a}]'

# 3. Create and attach an Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=banking-igw}]'
# Note the returned InternetGatewayId, e.g. igw-0xyz789
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id igw-0xyz789

# 4. Create a route table for the public subnet, add a route to the internet, associate it
aws ec2 create-route-table --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=public-rt}]'
# Note RouteTableId, e.g. rtb-0111

aws ec2 create-route --route-table-id rtb-0111 \
  --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0xyz789

aws ec2 associate-route-table --route-table-id rtb-0111 --subnet-id subnet-0public1a

# 5. Create a Security Group that only allows HTTPS (443) inbound from anywhere,
#    and SSH (22) only from your office IP — never 0.0.0.0/0 for SSH in real systems!
aws ec2 create-security-group --group-name web-sg \
  --description "Allows HTTPS from internet, SSH from office only" --vpc-id $VPC_ID

aws ec2 authorize-security-group-ingress --group-id sg-0webxxxx \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-id sg-0webxxxx \
  --protocol tcp --port 22 --cidr 203.0.113.25/32   # replace with YOUR office/home IP
```

## 3.6 Security Group vs NACL — the classic interview trap

```text
                SECURITY GROUP              NETWORK ACL
Level:          Instance (ENI) level         Subnet level
State:          Stateful (auto-allows        Stateless (must allow
                 the return traffic)          both directions explicitly)
Rules:          Allow rules only              Allow AND Deny rules
Evaluation:     ALL rules evaluated,          Rules evaluated in NUMBER
                 most permissive wins          ORDER, first match wins
Applies to:     Only the instances you        ALL instances in the subnet,
                 explicitly attach it to       automatically
```

## 3.7 Common mistakes / gotchas

- Putting a database in a **public subnet** — a very common finding in
  security audits, and a classic banking compliance failure.
- Forgetting the NAT Gateway costs money **per hour + per GB**, even when
  idle — a top cause of surprise AWS bills. Delete it after labs.
- SSH (port 22) open to `0.0.0.0/0` — always restrict to a specific IP, or
  better, use **Systems Manager Session Manager** (Section 17.5) which needs
  no open inbound ports at all.
- Only using one Availability Zone — no high availability if that data
  center has an outage.

## 3.8 Interview Q&A

**Q: Why put a database in a private subnet?**
A: To ensure it has no route to/from the public internet, so it can only be
reached by application servers inside the VPC — massively reducing the
attack surface, which is mandatory for banking compliance.

**Q: What does a NAT Gateway do that an Internet Gateway doesn't?**
A: An Internet Gateway allows two-way traffic (a resource can be reached
from the internet AND reach out to it). A NAT Gateway only allows one-way,
outbound-initiated traffic from private subnets — nothing from the internet
can initiate a connection in.

**Q: Stateful vs stateless firewall — give an AWS example of each.**
A: Security Groups are stateful (instance-level); Network ACLs are stateless
(subnet-level).

## 3.9 Cheat sheet

```text
VPC = your private network. CIDR = its IP range.
Public subnet  = has route to Internet Gateway
Private subnet = no direct route to internet (may use NAT Gateway to go OUT only)
Security Group = stateful, instance-level, Allow rules only
NACL           = stateless, subnet-level, Allow + Deny, ordered rules
Always design across >= 2 Availability Zones for high availability.
Database subnets: private, ideally no NAT route at all.
```

---

# 4. EC2 — Elastic Compute Cloud

## 4.1 What is it?

**EC2 gives you a virtual computer in the cloud** that you can rent by the
second, choose the CPU/RAM/disk for, install any OS on, and fully control
like your own laptop — except it lives in an AWS data center and you access
it remotely.

## 4.2 Why it exists

Before EC2, running a server meant buying physical hardware. EC2 lets you
launch a server in about 60 seconds, run it for an hour, and destroy it,
paying only for that hour. It's the foundation almost everything else in
AWS's compute story builds on (ECS and EKS ultimately run containers *on*
EC2 instances, unless you use the serverless "Fargate" mode).

## 4.3 Key Concepts

| Concept | Meaning |
|---|---|
| **AMI (Amazon Machine Image)** | A template/snapshot used to launch an instance — includes the OS and pre-installed software. AWS provides official AMIs (Amazon Linux, Ubuntu), or you can build your own "golden image." |
| **Instance Type** | The hardware profile, e.g. `t3.micro` (2 vCPU burstable, 1GB RAM, cheap, Free Tier eligible), `m5.large` (general purpose), `c5.xlarge` (compute optimized), `r5.large` (memory optimized, good for in-memory caches). |
| **Key Pair** | An SSH key pair used to securely log into a Linux instance (or decrypt the admin password on Windows) — AWS never stores the private key, only you have it. |
| **EBS (Elastic Block Store)** | Virtual hard disk attached to an instance — persists even if the instance is stopped (unlike "instance store" which is wiped). |
| **Elastic IP** | A static public IP address you can attach/detach from instances, so the IP doesn't change even if you replace the instance. |
| **User Data** | A script you can hand an instance at launch time that runs automatically on first boot (great for installing your app / Docker / running setup commands without manual SSH). |
| **Instance State** | running / stopped / terminated. Stopped instances still cost for EBS storage but not compute. Terminated is permanent deletion (unless EBS is set to persist). |
| **Placement/Spot/On-Demand/Reserved** | Pricing models — On-Demand (pay per second, no commitment), Reserved/Savings Plans (commit 1-3 years for big discount), Spot (bid on spare capacity for up to 90% discount, but AWS can reclaim it with 2 minutes notice — good for non-critical batch jobs, bad for your live banking API). |

## 4.4 Hands-on: Launching an EC2 instance via CLI, connecting to it, running a Spring Boot app

```bash
# 1. Create a key pair and save the private key locally
aws ec2 create-key-pair --key-name banking-app-key \
  --query 'KeyMaterial' --output text > banking-app-key.pem
chmod 400 banking-app-key.pem   # SSH refuses to use keys with loose permissions

# 2. Launch an instance in your public subnet, with a startup script (user-data)
cat > user-data.sh << 'EOF'
#!/bin/bash
# This entire script runs automatically the FIRST time the instance boots.
yum update -y
yum install -y java-17-amazon-corretto
# In a real pipeline, CodeDeploy or a Docker container would deliver the jar;
# this is a simplified illustration.
echo "Instance is ready for the Spring Boot app" > /home/ec2-user/ready.txt
EOF

aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name banking-app-key \
  --subnet-id subnet-0public1a \
  --security-group-ids sg-0webxxxx \
  --iam-instance-profile Name=EC2-S3-ReadOnly-Profile \
  --user-data file://user-data.sh \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=banking-app-server}]'

# 3. Once it's running, note the Public IP and connect
aws ec2 describe-instances --filters "Name=tag:Name,Values=banking-app-server" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' --output text

ssh -i banking-app-key.pem ec2-user@<PUBLIC_IP>
```

## 4.5 Running your Spring Boot jar as a systemd service (production pattern)

```bash
# On the EC2 instance, after copying your app.jar over (via scp or a pipeline):
sudo tee /etc/systemd/system/bankapp.service > /dev/null << 'EOF'
[Unit]
Description=Bank Loan Service Spring Boot App
After=network.target

[Service]
User=ec2-user
# -Xmx caps heap memory so the JVM doesn't eat the whole instance's RAM
ExecStart=/usr/bin/java -Xmx512m -jar /home/ec2-user/app.jar
SuccessExitStatus=143
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable bankapp      # start automatically on every reboot
sudo systemctl start bankapp
sudo systemctl status bankapp      # check it's running
sudo journalctl -u bankapp -f      # tail the logs live
```

## 4.6 Java SDK — launching and listing instances programmatically

```java
// pom.xml: software.amazon.awssdk:ec2

import software.amazon.awssdk.services.ec2.Ec2Client;
import software.amazon.awssdk.services.ec2.model.*;

public class Ec2Lister {
    public static void main(String[] args) {
        try (Ec2Client ec2 = Ec2Client.create()) {
            DescribeInstancesResponse response = ec2.describeInstances();

            // AWS groups instances into "Reservations" (a historical API quirk);
            // in practice you loop through reservations, then instances inside each
            for (Reservation reservation : response.reservations()) {
                for (Instance instance : reservation.instances()) {
                    System.out.printf(
                        "ID: %s | Type: %s | State: %s | Public IP: %s%n",
                        instance.instanceId(),
                        instance.instanceTypeAsString(),
                        instance.state().nameAsString(),
                        instance.publicIpAddress()
                    );
                }
            }
        }
    }
}
```

## 4.7 Common mistakes / gotchas

- Leaving instances **running** after a lab — this is the #1 cause of
  surprise bills for learners. Always `aws ec2 terminate-instances`.
- Hardcoding credentials on the instance instead of using an IAM Role
  (Section 2.6).
- Choosing an oversized instance type "just in case" — wastes money; monitor
  with CloudWatch (Section 7) and right-size instead.
- SSH key lost = you cannot get in through normal means (recovery involves
  detaching the EBS volume and mounting it on another instance — annoying,
  avoid by keeping keys safe, or by using Session Manager instead of SSH).

## 4.8 Interview Q&A

**Q: On-Demand vs Reserved vs Spot — when would you use each?**
A: On-Demand for unpredictable/short workloads (default). Reserved/Savings
Plans for steady-state production workloads you know will run for 1-3 years
(big discount for the commitment). Spot for fault-tolerant, interruptible
batch jobs (e.g., end-of-day report generation) where cost matters more than
guaranteed availability — never for a live transaction-processing service
since AWS can reclaim Spot capacity with only 2 minutes' notice.

**Q: What's the difference between stopping and terminating an instance?**
A: Stopping keeps the EBS root volume (and the instance ID) but shuts down
compute — you can start it again later, still billed for storage. Terminate
permanently deletes the instance (and by default its root EBS volume too,
unless "Delete on Termination" is disabled).

**Q: How does an EC2 instance get temporary AWS credentials without you
configuring anything?**
A: Via the IAM Role attached as an "Instance Profile" — the SDK's default
credential chain automatically queries the EC2 instance metadata service
(`169.254.169.254`) for temporary, auto-rotating credentials.

## 4.9 Cheat sheet

```text
EC2 = rentable virtual machine, billed per second.
AMI      = the OS template used to launch it
EBS      = its persistent virtual hard disk
Key Pair = SSH access credential
User Data = script that runs on FIRST boot
Always: IAM Role (not hardcoded keys), terminate after labs,
right-size instance type using CloudWatch metrics.
```

---

# 5. S3 — Simple Storage Service

## 5.1 What is it?

**S3 is a place to store files ("objects") of almost any size, forever, with
insanely high durability** (AWS advertises "11 nines" — 99.999999999% —
meaning if you store 10 million files, statistically you'd expect to lose
one roughly every 10,000 years).

It is **not** a filesystem you mount and edit files on directly (mostly) —
it's an object store you talk to via HTTP API calls: `PUT` a file in,
`GET` a file out, `DELETE` a file.

## 5.2 Why it exists

Every application needs to store files — loan application PDFs, KYC
documents, profile photos, log archives, database backups, static website
files. Building your own reliable, infinitely scalable file storage is hard.
S3 does it for you, at very low cost, with strong security controls — a core
service for any fintech handling documents.

## 5.3 Key Concepts

| Concept | Meaning |
|---|---|
| **Bucket** | A top-level container for objects. Bucket names are **globally unique** across all of AWS (like a domain name). |
| **Object** | A file plus its metadata, identified by a **Key** (its full "path" inside the bucket, e.g. `loans/2026/app-4521.pdf`). |
| **Object Storage Classes** | Standard (frequent access), Standard-IA (Infrequent Access, cheaper storage but costs to retrieve), Glacier / Glacier Deep Archive (very cheap, for compliance archives you rarely touch — perfect for 7-10 year bank record retention). |
| **Versioning** | Keeps every version of an object when overwritten — critical for audit trails in banking. |
| **Bucket Policy** | A resource-based IAM policy attached to the bucket itself, controlling who can access it. |
| **Pre-signed URL** | A temporary, time-limited URL you generate that grants access to a *specific* private object — e.g., letting a customer download their own loan statement for 15 minutes without making the bucket public. |
| **Lifecycle Rule** | Automatic rules to transition objects to cheaper storage classes over time, or delete them after N days — e.g. "move to Glacier after 1 year, delete after 10 years" for regulatory retention. |
| **Server-Side Encryption (SSE)** | S3 encrypts your data at rest automatically; you can use AWS-managed keys (SSE-S3), or your own via KMS (SSE-KMS) for tighter control (Section 17.4). |
| **Block Public Access** | An account/bucket-level setting that, when on (the default and **strongly recommended for banking**), prevents any bucket from ever being made public, even by accident. |

## 5.4 Hands-on: CLI

```bash
# 1. Create a bucket (must be globally unique, so add something unique like your name)
aws s3api create-bucket --bucket ananth-bank-loan-docs-2026 \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

# 2. Block all public access (do this on every bucket, always, unless it's genuinely a public website)
aws s3api put-public-access-block --bucket ananth-bank-loan-docs-2026 \
  --public-access-block-configuration \
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# 3. Enable versioning (important for compliance / accidental-overwrite protection)
aws s3api put-bucket-versioning --bucket ananth-bank-loan-docs-2026 \
  --versioning-configuration Status=Enabled

# 4. Upload a file
aws s3 cp loan-application-4521.pdf s3://ananth-bank-loan-docs-2026/loans/2026/app-4521.pdf

# 5. List objects
aws s3 ls s3://ananth-bank-loan-docs-2026/loans/2026/

# 6. Download a file
aws s3 cp s3://ananth-bank-loan-docs-2026/loans/2026/app-4521.pdf ./downloaded.pdf

# 7. Add a lifecycle rule: move to Glacier after 365 days, delete after 3650 days (10 years)
cat > lifecycle.json << 'EOF'
{
  "Rules": [
    {
      "ID": "ArchiveThenDeleteLoanDocs",
      "Filter": { "Prefix": "loans/" },
      "Status": "Enabled",
      "Transitions": [
        { "Days": 365, "StorageClass": "GLACIER" }
      ],
      "Expiration": { "Days": 3650 }
    }
  ]
}
EOF
aws s3api put-bucket-lifecycle-configuration \
  --bucket ananth-bank-loan-docs-2026 --lifecycle-configuration file://lifecycle.json
```

## 5.5 Java SDK + Spring Boot integration

```java
// S3Config.java — a Spring @Configuration bean, so any @Service can just @Autowired an S3Client
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;

@Configuration
public class S3Config {

    @Bean
    public S3Client s3Client() {
        // No .credentialsProvider() call needed — SDK uses the default chain
        // (IAM Role on EC2/ECS in production, ~/.aws/credentials on your laptop)
        return S3Client.builder()
                .region(Region.AP_SOUTH_1)
                .build();
    }
}
```

```java
// DocumentStorageService.java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import software.amazon.awssdk.core.sync.RequestBody;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.*;
import software.amazon.awssdk.services.s3.presigner.S3Presigner;
import software.amazon.awssdk.services.s3.presigner.model.GetObjectPresignRequest;

import java.io.IOException;
import java.time.Duration;

@Service
public class DocumentStorageService {

    private static final String BUCKET = "ananth-bank-loan-docs-2026";

    @Autowired
    private S3Client s3Client;

    /**
     * Uploads a customer's loan document to S3.
     * The "key" is the file's full path inside the bucket.
     */
    public String uploadLoanDocument(String customerId, MultipartFile file) throws IOException {
        String key = "loans/" + customerId + "/" + file.getOriginalFilename();

        PutObjectRequest request = PutObjectRequest.builder()
                .bucket(BUCKET)
                .key(key)
                // Encrypt every object at rest — mandatory for banking data
                .serverSideEncryption(ServerSideEncryption.AES256)
                .contentType(file.getContentType())
                .build();

        s3Client.putObject(request, RequestBody.fromInputStream(file.getInputStream(), file.getSize()));
        return key;
    }

    /**
     * Generates a temporary link (valid 15 minutes) so a customer can download
     * THEIR OWN document, without the bucket ever being public.
     */
    public String generateTemporaryDownloadLink(String key) {
        try (S3Presigner presigner = S3Presigner.create()) {
            GetObjectRequest getRequest = GetObjectRequest.builder()
                    .bucket(BUCKET)
                    .key(key)
                    .build();

            GetObjectPresignRequest presignRequest = GetObjectPresignRequest.builder()
                    .signatureDuration(Duration.ofMinutes(15))
                    .getObjectRequest(getRequest)
                    .build();

            return presigner.presignGetObject(presignRequest).url().toString();
        }
    }

    /** Lists every document a specific customer has uploaded. */
    public ListObjectsV2Response listCustomerDocuments(String customerId) {
        ListObjectsV2Request request = ListObjectsV2Request.builder()
                .bucket(BUCKET)
                .prefix("loans/" + customerId + "/")
                .build();
        return s3Client.listObjectsV2(request);
    }
}
```

## 5.6 Real banking example

A KYC (Know Your Customer) document upload flow:

1. Customer uploads Aadhaar/PAN scan via your Spring Boot API.
2. API calls `uploadLoanDocument()` above — stored encrypted, versioned.
3. Bucket has **Block Public Access ON** always — the file is never directly
   reachable by URL.
4. When the customer wants to view their own document later, your API
   generates a 15-minute pre-signed URL and returns *that* — not a permanent
   public link.
5. A Lifecycle rule auto-archives to Glacier after 1 year (cheap, but still
   retrievable for audits) and deletes after the regulatory retention period.

## 5.7 Common mistakes / gotchas

- Making a bucket public "just to test something quickly" — a huge number
  of real-world data breaches (leaked customer records) trace back to this
  exact mistake.
- Storing secrets (DB passwords, API keys) as plain files in S3 instead of
  using Secrets Manager (Section 12).
- Forgetting versioning means old versions still cost storage — pair with a
  lifecycle rule that expires old versions too.
- Using the AWS SDK v1 `AmazonS3Client` in new code — prefer SDK v2
  (`S3Client`) as used above; v1 is in maintenance mode.

## 5.8 Interview Q&A

**Q: How do you let a user download a private S3 file without making the
bucket public?**
A: Generate a pre-signed URL with a short expiry using the SDK — it grants
temporary, scoped access to that one object only.

**Q: What storage class would you use for 10-year regulatory bank records
you almost never access?**
A: S3 Glacier or Glacier Deep Archive — a lifecycle rule can auto-transition
objects there after an initial "hot" period, drastically cutting storage
cost while meeting retention requirements.

**Q: Is S3 strongly or eventually consistent?**
A: Since December 2020, S3 provides strong read-after-write consistency for
all operations — a GET immediately after a successful PUT will always return
the latest data.

## 5.9 Cheat sheet

```text
S3 = object storage. Bucket name = globally unique.
Key = the object's full path inside the bucket.
ALWAYS: Block Public Access ON, encryption ON, versioning ON for important data.
Pre-signed URL = temporary access to a private object.
Lifecycle rules = auto move to cheaper storage / auto delete over time.
Storage classes (hot -> cold): Standard -> Standard-IA -> Glacier -> Glacier Deep Archive
```

---

# 6. RDS — Relational Database Service

## 6.1 What is it?

**RDS is a managed relational database** — you get a real MySQL, PostgreSQL,
MariaDB, Oracle, or SQL Server database, but AWS handles patching, backups,
replication, and failover for you. You just connect with a JDBC URL, exactly
like a normal database.

## 6.2 Why it exists

Running your own database server means you patch the OS, patch the DB engine,
configure backups, set up replication for high availability, and handle
failover yourself when the primary crashes at 3 AM. RDS automates all of
that. For a banking system where data durability and uptime are non-
negotiable, this is enormous value.

## 6.3 Key Concepts

| Concept | Meaning |
|---|---|
| **DB Instance** | One running database server. |
| **Engine** | Which database software: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, or Amazon Aurora (AWS's own MySQL/PostgreSQL-compatible engine, faster and more scalable). |
| **Multi-AZ Deployment** | RDS automatically maintains a **synchronous standby replica** in a different Availability Zone. If the primary fails, RDS automatically fails over to the standby, usually within 60-120 seconds, with **no code changes** — your app just reconnects to the same DNS endpoint. This is the single most important RDS feature for banking systems. |
| **Read Replica** | An **asynchronous** copy of your database used to offload read traffic (reports, dashboards) from the primary. Unlike Multi-AZ, Read Replicas are for *scaling reads*, not automatic failover (though you can manually promote one). |
| **Parameter Group** | A set of database engine configuration settings (like `max_connections`) you can tune. |
| **Automated Backups** | Daily snapshots + transaction logs, allowing **point-in-time recovery** to any second within your retention window (up to 35 days). |
| **Snapshot** | A manual or automatic full backup you can restore into a brand-new DB instance. |
| **Storage Autoscaling** | RDS can automatically grow your storage as your data grows, so you don't run out of disk unexpectedly. |

## 6.4 Hands-on: CLI — Creating a Multi-AZ PostgreSQL instance

```bash
# 1. First, create a DB Subnet Group spanning your private DB subnets across 2 AZs
aws rds create-db-subnet-group \
  --db-subnet-group-name banking-db-subnet-group \
  --db-subnet-group-description "Private subnets for RDS" \
  --subnet-ids subnet-0dbprivate1a subnet-0dbprivate1b

# 2. Create a Security Group for the DB, allowing inbound 5432 (Postgres) ONLY from the app tier's SG
aws ec2 create-security-group --group-name db-sg \
  --description "Allows Postgres only from app servers" --vpc-id $VPC_ID

aws ec2 authorize-security-group-ingress --group-id sg-0dbxxxx \
  --protocol tcp --port 5432 --source-group sg-0appxxxx
  # NOTE: source is another Security Group, not an IP range — this means
  # "only instances that are members of the app Security Group can connect"

# 3. Launch the Multi-AZ database
aws rds create-db-instance \
  --db-instance-identifier banking-prod-db \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --engine-version 16.3 \
  --master-username bankadmin \
  --master-user-password 'ChangeMeInSecretsManager!123' \
  --allocated-storage 100 \
  --storage-type gp3 \
  --multi-az \
  --db-subnet-group-name banking-db-subnet-group \
  --vpc-security-group-ids sg-0dbxxxx \
  --backup-retention-period 7 \
  --no-publicly-accessible \
  --storage-encrypted
```

> **Never** actually put a real password on the command line like above in
> production — this is for learning only. In real pipelines, RDS integrates
> directly with Secrets Manager to auto-generate and rotate the master
> password (see Section 12).

```bash
# 4. Once available, get the connection endpoint
aws rds describe-db-instances --db-instance-identifier banking-prod-db \
  --query 'DBInstances[0].Endpoint.Address' --output text
# e.g. banking-prod-db.xxxxxxxxx.ap-south-1.rds.amazonaws.com
```

## 6.5 Spring Boot integration (this part is just... normal Spring Boot!)

The beauty of RDS: your Spring Boot code barely changes from talking to any
Postgres/MySQL database. You just point the JDBC URL at the RDS endpoint.

```properties
# application.properties
spring.datasource.url=jdbc:postgresql://banking-prod-db.xxxxxxxxx.ap-south-1.rds.amazonaws.com:5432/bankdb
spring.datasource.username=bankadmin
# NEVER hardcode the password here in a real project -- fetch it from
# Secrets Manager at startup instead (full example in Section 12.5)
spring.datasource.password=${DB_PASSWORD}

spring.jpa.hibernate.ddl-auto=validate
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Connection pool tuning — HikariCP (Spring Boot's default pool)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.connection-timeout=30000
```

```java
// A completely normal Spring Data JPA repository — RDS is just Postgres underneath
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;
import java.util.UUID;

public interface LoanApplicationRepository extends JpaRepository<LoanApplication, UUID> {
    List<LoanApplication> findByCustomerIdAndStatus(String customerId, String status);
}
```

## 6.6 Real banking example: Multi-AZ failover in action

```text
Normal operation:
  Spring Boot App --JDBC--> banking-prod-db.xxxx.rds.amazonaws.com (Primary in AZ-a)
                                           |
                                (synchronous replication)
                                           |
                             Standby replica (AZ-b, not directly reachable)

If AZ-a has an outage:
  RDS detects the primary is unreachable
  RDS automatically promotes the standby in AZ-b to be the new primary
  RDS updates the SAME DNS endpoint to point at AZ-b
  Your Spring Boot app's connection pool reconnects (HikariCP retries automatically)
  Typical failover time: 60-120 seconds, ZERO code changes needed
```

This is exactly why Multi-AZ is considered mandatory for any core banking
database — and a favorite interview topic ("how do you achieve database high
availability on AWS without managing replication yourself?").

## 6.7 Common mistakes / gotchas

- Making an RDS instance **publicly accessible** — a serious security issue;
  should always be `--no-publicly-accessible` and live in a private subnet.
- Confusing Multi-AZ (HA/failover) with Read Replicas (read scaling) — they
  solve different problems and are frequently mixed up in interviews.
- Not enabling encryption at creation time — you **cannot** enable encryption
  on an existing unencrypted RDS instance; you'd have to snapshot, copy the
  snapshot with encryption enabled, and restore from that.
- Undersized `max_connections` combined with an oversized connection pool
  across many app instances — causes "too many connections" errors under
  load; use RDS Proxy for connection pooling at scale.

## 6.8 Interview Q&A

**Q: Multi-AZ vs Read Replica — what's the real difference?**
A: Multi-AZ is for high availability — a synchronous standby in another AZ
that RDS automatically fails over to if the primary dies; you cannot query
the standby directly. A Read Replica is an asynchronous copy used purely to
scale out read traffic; it can be in the same or a different region, and you
*can* query it directly, but it's not an automatic-failover target by
default (you'd manually promote it).

**Q: How do you achieve point-in-time recovery?**
A: RDS continuously backs up transaction logs in addition to daily
snapshots, so within your retention window (up to 35 days) you can restore
the database to any specific second — restores create a *new* DB instance,
they don't overwrite the existing one.

**Q: Why should a Spring Boot app never hardcode the RDS master password?**
A: It's a secret that should be stored and rotated via Secrets Manager and
fetched at runtime — hardcoding it risks leaking it via source control, and
makes rotation operationally painful.

## 6.9 Cheat sheet

```text
RDS = managed relational DB (MySQL/Postgres/MariaDB/Oracle/SQL Server/Aurora)
Multi-AZ      = synchronous standby, AUTOMATIC failover, for HIGH AVAILABILITY
Read Replica  = asynchronous copy, for READ SCALING, manual promotion
Always: private subnet, not publicly accessible, storage encrypted at creation,
        backups retained, password from Secrets Manager not hardcoded.
```

---

# 7. CloudWatch — Monitoring & Observability

## 7.1 What is it?

**CloudWatch is AWS's built-in monitoring service** — it collects metrics
(numbers over time, like CPU%), logs (text output from your app), and lets
you set alarms that trigger actions (send an SMS, scale up servers, restart
a task) when something crosses a threshold.

## 7.2 Why it exists

You cannot fix what you cannot see. In a banking system, if response times
spike or a service starts erroring, you need to know **before** the customer
complains, and you need historical data to debug what happened at 2:47 AM
last Tuesday. CloudWatch is the eyes and ears of your whole AWS
infrastructure.

## 7.3 Key Concepts

| Concept | Meaning |
|---|---|
| **Metric** | A time-series of numbers, e.g. `CPUUtilization`, `NetworkIn`. AWS services publish these automatically; you can also publish your own **custom metrics** (e.g., "number of loan applications processed per minute"). |
| **Namespace** | A grouping for metrics, e.g. `AWS/EC2`, `AWS/RDS`, or your own custom namespace like `BankingApp/Loans`. |
| **Dimension** | A key-value pair that identifies which specific resource a metric belongs to, e.g. `InstanceId=i-0abc123`. |
| **Alarm** | A rule that watches a metric and changes state (OK / ALARM / INSUFFICIENT_DATA) when a threshold is breached for a set number of periods, then triggers an action (e.g., publish to an SNS topic, trigger Auto Scaling). |
| **Log Group / Log Stream** | Logs are organized into Log Groups (e.g., one per application), which contain Log Streams (e.g., one per instance/container). |
| **CloudWatch Agent** | A small program you install on EC2 to push OS-level metrics (memory, disk — which AWS does NOT collect by default) and log files into CloudWatch. |
| **Dashboard** | A customizable visual board of graphs/widgets for at-a-glance monitoring. |
| **CloudWatch Logs Insights** | A query language to search/filter/aggregate log data, e.g. "count errors per 5 minutes across all instances." |

## 7.4 Hands-on: CLI — Creating an alarm and pushing a custom metric

```bash
# 1. Create an SNS topic that the alarm will notify (SNS covered fully in Section 9)
aws sns create-topic --name ops-alerts
# note the TopicArn returned

aws sns subscribe --topic-arn arn:aws:sns:ap-south-1:123456789012:ops-alerts \
  --protocol email --notification-endpoint ananth@example.com

# 2. Create a CloudWatch Alarm: alert if average CPU > 80% for 3 consecutive 5-minute periods
aws cloudwatch put-metric-alarm \
  --alarm-name HighCPU-BankAppServer \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-0abc123456 \
  --statistic Average \
  --period 300 \
  --evaluation-periods 3 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:ap-south-1:123456789012:ops-alerts

# 3. Push a CUSTOM application metric (e.g., "loan applications processed")
aws cloudwatch put-metric-data \
  --namespace "BankingApp/Loans" \
  --metric-name ApplicationsProcessed \
  --value 42 \
  --unit Count
```

## 7.5 Java SDK + Spring Boot: pushing custom metrics and structured logs

```java
// LoanMetricsPublisher.java
import org.springframework.stereotype.Component;
import software.amazon.awssdk.services.cloudwatch.CloudWatchClient;
import software.amazon.awssdk.services.cloudwatch.model.*;

import java.time.Instant;

@Component
public class LoanMetricsPublisher {

    private final CloudWatchClient cloudWatch = CloudWatchClient.create();
    private static final String NAMESPACE = "BankingApp/Loans";

    /** Call this every time a loan application is approved, so we can graph
     *  approval rate over time and alarm if it drops suspiciously (fraud signal). */
    public void recordLoanApproved(double loanAmount) {
        MetricDatum amountDatum = MetricDatum.builder()
                .metricName("LoanAmountApproved")
                .unit(StandardUnit.NONE)
                .value(loanAmount)
                .timestamp(Instant.now())
                .build();

        MetricDatum countDatum = MetricDatum.builder()
                .metricName("LoansApprovedCount")
                .unit(StandardUnit.COUNT)
                .value(1.0)
                .timestamp(Instant.now())
                .build();

        PutMetricDataRequest request = PutMetricDataRequest.builder()
                .namespace(NAMESPACE)
                .metricData(amountDatum, countDatum)
                .build();

        cloudWatch.putMetricData(request);
    }
}
```

```xml
<!-- To ship your Spring Boot application LOGS into CloudWatch Logs automatically,
     add the logback appender -->
<dependency>
    <groupId>ca.pjer</groupId>
    <artifactId>logback-awslogs-appender</artifactId>
    <version>1.6.0</version>
</dependency>
```

```xml
<!-- logback-spring.xml -->
<appender name="CLOUDWATCH" class="ca.pjer.logback.AwsLogsAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </layout>
    <logGroupName>banking-app/loan-service</logGroupName>
    <logStreamUuidPrefix>loan-service-</logStreamUuidPrefix>
    <logRegion>ap-south-1</logRegion>
    <maxBatchLogEvents>50</maxBatchLogEvents>
    <maxFlushTimeMillis>30000</maxFlushTimeMillis>
</appender>

<root level="INFO">
    <appender-ref ref="CLOUDWATCH" />
</root>
```

## 7.6 Real banking example

A fraud team wants to know within minutes if failed-login attempts spike
(possible credential-stuffing attack):

1. Spring Boot login service increments a custom CloudWatch metric
   `FailedLoginAttempts` on every failed login.
2. A CloudWatch Alarm watches that metric: "sum > 500 within 5 minutes."
3. On breach, the alarm publishes to an SNS topic subscribed by both the
   on-call engineer (SMS) and an automated Lambda that can temporarily
   tighten WAF rate limiting (Section 17.10).
4. CloudWatch Logs Insights lets the security team later query:
   `fields @timestamp, sourceIp | filter eventType="LOGIN_FAILED" | stats count() by sourceIp`

## 7.7 Common mistakes / gotchas

- Forgetting that **memory and disk usage are NOT collected by default** on
  EC2 — you must install the CloudWatch Agent for those.
- Setting alarm thresholds too sensitive (constant false alarms → team
  starts ignoring them, called "alarm fatigue") or too loose (real issues
  missed).
- Not setting a **log retention period** — by default CloudWatch Logs are
  kept forever, silently costing more over years; set an explicit retention
  (e.g. 90 days, or longer if compliance requires).
- Confusing "Alarm state = OK" with "no alarm was ever created" — INSUFFICIENT_DATA
  state means the alarm doesn't have enough data points yet.

## 7.8 Interview Q&A

**Q: How would you monitor memory usage on an EC2 instance?**
A: Install the CloudWatch Agent — EC2's default metrics only cover CPU,
network, disk I/O, and status checks; memory and actual disk space used are
not published without the Agent.

**Q: What's the difference between a Metric and a Log?**
A: A Metric is a numeric time-series meant for graphing/alarming (efficient,
low-storage). A Log is arbitrary text/event data meant for detailed
debugging and searching — richer but more expensive to store and query at
scale.

## 7.9 Cheat sheet

```text
Metric = number over time (CPUUtilization, custom app metrics)
Alarm  = watches a metric, changes state, triggers an action (SNS, Auto Scaling...)
Logs   = text data in Log Groups / Log Streams, searchable via Logs Insights
Memory/disk metrics on EC2 need the CloudWatch AGENT installed.
Set log retention explicitly -- default is "forever" (costs add up).
```

---

# 8. SQS — Simple Queue Service

## 8.1 What is it?

**SQS is a managed message queue** — one part of your system drops a message
into the queue, and another part picks it up and processes it, whenever it's
ready. Neither side has to be online at the exact same moment.

## 8.2 Why it exists

Imagine your bank's loan service receives 10,000 applications at once during
a promotional campaign. If it tried to process each one synchronously and
immediately (calling credit bureaus, running fraud checks, writing to the
DB), the API would be slow or crash under load. Instead:

1. The API just validates basic input and drops a message ("process loan
   application #4521") onto a queue — this takes milliseconds.
2. A separate pool of "worker" processes pulls messages off the queue at
   whatever pace they can handle, and does the heavy processing.

This is called **decoupling** — the two parts of the system don't need to
know about each other or run at the same speed. If the worker fleet is slow
or temporarily down, messages just wait safely in the queue instead of being
lost.

## 8.3 Key Concepts

| Concept | Meaning |
|---|---|
| **Queue** | The message buffer itself. |
| **Standard Queue** | Nearly unlimited throughput, "at-least-once" delivery (a message *might* be delivered more than once — your processing logic must be idempotent), best-effort ordering (not guaranteed). |
| **FIFO Queue** | First-In-First-Out — exact ordering guaranteed, "exactly-once" processing, but lower throughput (up to 3,000 msgs/sec with batching). Use for anything where order matters, e.g. a sequence of debit/credit transactions on the same account. |
| **Visibility Timeout** | When a consumer reads a message, it becomes temporarily invisible to other consumers for this duration, so two workers don't process the same message simultaneously. If the consumer doesn't delete the message within this window (meaning it crashed or is still working), the message becomes visible again for another consumer to retry. |
| **Dead Letter Queue (DLQ)** | A separate queue where messages get moved automatically after failing processing a set number of times — prevents a "poison pill" message from blocking the queue forever, and lets you inspect failures later. |
| **Long Polling** | Consumers ask "give me a message, and wait up to N seconds if none are available right now" instead of hammering the API constantly (short polling) — cheaper and more efficient. |
| **Message Retention** | How long an unprocessed message stays in the queue before being auto-deleted (default 4 days, max 14 days). |
| **Idempotency** | Designing your processing logic so handling the *same* message twice has no bad side effect (e.g., check "have I already processed application #4521?" before debiting an account) — essential because Standard queues can deliver a message more than once. |

## 8.4 Hands-on: CLI

```bash
# 1. Create a Dead Letter Queue first
aws sqs create-queue --queue-name loan-processing-dlq
# note QueueUrl and get its ARN:
aws sqs get-queue-attributes --queue-url https://sqs.ap-south-1.amazonaws.com/123456789012/loan-processing-dlq \
  --attribute-names QueueArn

# 2. Create the main queue, pointing its redrive policy at the DLQ
cat > redrive-policy.json << 'EOF'
{
  "deadLetterTargetArn": "arn:aws:sqs:ap-south-1:123456789012:loan-processing-dlq",
  "maxReceiveCount": "5"
}
EOF

aws sqs create-queue --queue-name loan-processing-queue \
  --attributes VisibilityTimeout=60,RedrivePolicy="$(cat redrive-policy.json | jq -c . | sed 's/"/\\"/g')"

# 3. Send a message
aws sqs send-message \
  --queue-url https://sqs.ap-south-1.amazonaws.com/123456789012/loan-processing-queue \
  --message-body '{"applicationId": "4521", "customerId": "CUST-9981", "amount": 500000}'

# 4. Receive and process a message (long poll up to 10 seconds)
aws sqs receive-message \
  --queue-url https://sqs.ap-south-1.amazonaws.com/123456789012/loan-processing-queue \
  --wait-time-seconds 10 --max-number-of-messages 1

# 5. Delete the message once successfully processed (MUST do this or it reappears!)
aws sqs delete-message \
  --queue-url https://sqs.ap-south-1.amazonaws.com/123456789012/loan-processing-queue \
  --receipt-handle "AQEB....==" # from the receive-message response
```

## 8.5 Java SDK + Spring Boot: Producer and Consumer

```java
// LoanQueueProducer.java — the API side, runs fast, just enqueues work
import org.springframework.stereotype.Service;
import software.amazon.awssdk.services.sqs.SqsClient;
import software.amazon.awssdk.services.sqs.model.SendMessageRequest;
import com.fasterxml.jackson.databind.ObjectMapper;

@Service
public class LoanQueueProducer {

    private final SqsClient sqsClient = SqsClient.create();
    private final ObjectMapper objectMapper = new ObjectMapper();
    private static final String QUEUE_URL =
        "https://sqs.ap-south-1.amazonaws.com/123456789012/loan-processing-queue";

    public void enqueueLoanApplication(LoanApplicationRequest request) throws Exception {
        String messageBody = objectMapper.writeValueAsString(request);

        SendMessageRequest sendRequest = SendMessageRequest.builder()
                .queueUrl(QUEUE_URL)
                .messageBody(messageBody)
                // MessageGroupId is required for FIFO queues, ignored for Standard.
                // Grouping by customerId ensures all of ONE customer's messages
                // stay strictly ordered relative to each other, if this were FIFO.
                .build();

        sqsClient.sendMessage(sendRequest);
    }
}
```

```java
// LoanQueueConsumer.java — the worker side, runs continuously, polls and processes
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import software.amazon.awssdk.services.sqs.SqsClient;
import software.amazon.awssdk.services.sqs.model.*;

import java.util.List;

@Component
public class LoanQueueConsumer {

    private final SqsClient sqsClient = SqsClient.create();
    private static final String QUEUE_URL =
        "https://sqs.ap-south-1.amazonaws.com/123456789012/loan-processing-queue";

    // Runs every 5 seconds. In production you'd more likely run a dedicated
    // long-lived polling thread rather than @Scheduled, but this illustrates the flow simply.
    @Scheduled(fixedDelay = 5000)
    public void pollAndProcess() {
        ReceiveMessageRequest receiveRequest = ReceiveMessageRequest.builder()
                .queueUrl(QUEUE_URL)
                .maxNumberOfMessages(10)
                .waitTimeSeconds(5) // long polling
                .build();

        List<Message> messages = sqsClient.receiveMessage(receiveRequest).messages();

        for (Message message : messages) {
            try {
                processLoanApplication(message.body());

                // Only delete AFTER successful processing.
                // If processing throws, we skip delete -> message becomes visible
                // again after the visibility timeout and gets retried automatically.
                sqsClient.deleteMessage(DeleteMessageRequest.builder()
                        .queueUrl(QUEUE_URL)
                        .receiptHandle(message.receiptHandle())
                        .build());

            } catch (Exception e) {
                // Deliberately do NOT delete -- let SQS's redrive policy handle
                // retries, and eventually route to the DLQ after maxReceiveCount.
                System.err.println("Failed to process message, will retry: " + e.getMessage());
            }
        }
    }

    private void processLoanApplication(String messageBody) {
        // IMPORTANT: this must be IDEMPOTENT -- check a "processed_applications"
        // table/flag first, because Standard queues may deliver a message MORE
        // THAN ONCE. Never blindly debit/credit an account here without a check.
        System.out.println("Processing: " + messageBody);
    }
}
```

## 8.6 Real banking example

- **Order matters (use FIFO)**: a sequence of debit → interest calculation →
  credit on the same account must happen in exact order. Use a FIFO queue
  with `MessageGroupId = accountId` so each account's messages stay ordered,
  while different accounts process in parallel.
- **Order doesn't matter (use Standard)**: sending "loan approved" push
  notifications — any order is fine, and Standard's near-unlimited
  throughput handles bursty campaign traffic better.

## 8.7 Common mistakes / gotchas

- Deleting the message **before** processing succeeds — if the worker
  crashes mid-processing, the message is lost forever with no retry.
- Not handling duplicate delivery (non-idempotent processing) on Standard
  queues — can cause double-charging a customer.
- Setting Visibility Timeout too short for how long processing actually
  takes — causes the SAME message to be picked up by a second worker while
  the first is still working on it.
- Forgetting a Dead Letter Queue — a permanently malformed message can retry
  forever, wasting resources, with no visibility into the failure.

## 8.8 Interview Q&A

**Q: Standard vs FIFO — when would you pick each?**
A: FIFO when exact order and exactly-once processing matter (financial
transaction sequences), accepting lower throughput. Standard when massive
throughput matters more than strict order, accepting the need for idempotent
processing due to possible duplicate delivery.

**Q: What's a Dead Letter Queue for?**
A: Isolating messages that repeatedly fail processing (after
`maxReceiveCount` retries) so they don't block the main queue forever, and
so engineers can inspect/debug them separately.

**Q: Why is long polling preferred over short polling?**
A: It reduces the number of empty API responses (and therefore cost and
latency) by having the SQS API wait up to 20 seconds for a message to become
available before responding, instead of immediately returning "nothing
found" and forcing the client to hammer the API repeatedly.

## 8.9 Cheat sheet

```text
SQS = managed message queue, decouples producer from consumer.
Standard = high throughput, best-effort order, AT-LEAST-ONCE (dedupe yourself)
FIFO     = strict order, exactly-once, lower throughput, needs MessageGroupId
Visibility Timeout = how long a received message is hidden from other consumers
DLQ = catches messages that fail repeatedly (set maxReceiveCount)
ALWAYS delete the message only AFTER successful processing.
```

---

# 9. SNS — Simple Notification Service

## 9.1 What is it?

**SNS is a managed "pub/sub" (publish-subscribe) messaging service.** One
publisher sends **one** message to a **Topic**, and SNS automatically fans it
out to **every subscriber** of that topic — could be an email address, an
SMS number, an SQS queue, a Lambda function, or an HTTP endpoint — all at
once, in parallel.

## 9.2 Why it exists: SNS vs SQS, the classic confusion

This is *the* most commonly confused pair in AWS interviews. Here's the
clean distinction:

```text
SQS = ONE message goes to ONE consumer (a queue — pull-based, point-to-point)
SNS = ONE message goes to MANY subscribers at once (a topic — push-based, fan-out)
```

Real pattern used constantly in production: **SNS + SQS Fan-out**. A
publisher sends one event to an SNS Topic. Multiple SQS Queues (each with
different consumer services) are subscribed to that same topic. SNS pushes a
copy of the message to *every* queue automatically. This way, one event
("loan approved") can independently trigger the notification-service, the
accounting-service, and the audit-service — each processing at its own
pace, from its own queue, without the publisher knowing or caring who's
listening.

## 9.3 Key Concepts

| Concept | Meaning |
|---|---|
| **Topic** | The named channel publishers send to. |
| **Subscription** | One endpoint (email, SMS, SQS queue, Lambda, HTTPS webhook, mobile push) registered to receive every message published to a topic. |
| **Publisher** | Whoever calls `Publish()` on the topic. |
| **Message Filtering** | Subscribers can set a **filter policy** so they only receive a subset of messages matching certain message attributes — e.g., only "high value loan" events, not every loan event. |
| **Fan-out pattern** | SNS Topic -> multiple SQS Queues, each independently consumed. |
| **FIFO Topics** | Like SQS FIFO — ordered, exactly-once delivery, paired with FIFO SQS subscribers. |

## 9.4 Hands-on: CLI — Setting up SNS + SQS Fan-out

```bash
# 1. Create the SNS topic
aws sns create-topic --name loan-events-topic
# note TopicArn e.g. arn:aws:sns:ap-south-1:123456789012:loan-events-topic

# 2. Create two separate SQS queues for two independent consumer services
aws sqs create-queue --queue-name notification-service-queue
aws sqs create-queue --queue-name audit-service-queue

# 3. Get each queue's ARN
aws sqs get-queue-attributes --queue-url <notification-queue-url> --attribute-names QueueArn
aws sqs get-queue-attributes --queue-url <audit-queue-url> --attribute-names QueueArn

# 4. Allow SNS to send messages INTO each queue (queue policy)
cat > sqs-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "sns.amazonaws.com" },
    "Action": "sqs:SendMessage",
    "Resource": "arn:aws:sqs:ap-south-1:123456789012:notification-service-queue",
    "Condition": {
      "ArnEquals": { "aws:SourceArn": "arn:aws:sns:ap-south-1:123456789012:loan-events-topic" }
    }
  }]
}
EOF
aws sqs set-queue-attributes --queue-url <notification-queue-url> \
  --attributes Policy="$(cat sqs-policy.json)"
# (repeat similarly for audit-service-queue)

# 5. Subscribe both queues to the topic
aws sns subscribe --topic-arn arn:aws:sns:ap-south-1:123456789012:loan-events-topic \
  --protocol sqs --notification-endpoint arn:aws:sqs:ap-south-1:123456789012:notification-service-queue

aws sns subscribe --topic-arn arn:aws:sns:ap-south-1:123456789012:loan-events-topic \
  --protocol sqs --notification-endpoint arn:aws:sqs:ap-south-1:123456789012:audit-service-queue

# 6. Publish ONE message -- both queues receive a copy automatically
aws sns publish --topic-arn arn:aws:sns:ap-south-1:123456789012:loan-events-topic \
  --message '{"event": "LOAN_APPROVED", "applicationId": "4521", "amount": 500000}' \
  --message-attributes '{"loanTier":{"DataType":"String","StringValue":"HIGH_VALUE"}}'
```

## 9.5 Java SDK + Spring Boot: Publishing events

```java
// LoanEventPublisher.java
import org.springframework.stereotype.Service;
import software.amazon.awssdk.services.sns.SnsClient;
import software.amazon.awssdk.services.sns.model.MessageAttributeValue;
import software.amazon.awssdk.services.sns.model.PublishRequest;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.Map;

@Service
public class LoanEventPublisher {

    private final SnsClient snsClient = SnsClient.create();
    private final ObjectMapper objectMapper = new ObjectMapper();
    private static final String TOPIC_ARN =
        "arn:aws:sns:ap-south-1:123456789012:loan-events-topic";

    public void publishLoanApproved(String applicationId, double amount) throws Exception {
        String messageJson = objectMapper.writeValueAsString(
                Map.of("event", "LOAN_APPROVED", "applicationId", applicationId, "amount", amount));

        // Message attributes let SUBSCRIBERS filter without parsing the whole body --
        // e.g., only the fraud-review service subscribes to loanTier=HIGH_VALUE
        String tier = amount > 1_000_000 ? "HIGH_VALUE" : "STANDARD";

        PublishRequest request = PublishRequest.builder()
                .topicArn(TOPIC_ARN)
                .message(messageJson)
                .messageAttributes(Map.of(
                        "loanTier", MessageAttributeValue.builder()
                                .dataType("String")
                                .stringValue(tier)
                                .build()
                ))
                .build();

        snsClient.publish(request);
    }
}
```

```json
// Example SNS Subscription Filter Policy -- attached to the fraud-review-queue
// subscription, so it ONLY receives high-value loan events, ignoring everything else
{
  "loanTier": ["HIGH_VALUE"]
}
```

## 9.6 Real banking example

A single "Loan Approved" event, published once, needs to:
- Send an SMS/email to the customer (SNS direct SMS/Email subscription)
- Update the accounting ledger (SQS queue -> accounting-service)
- Write an audit log entry (SQS queue -> audit-service)
- Alert the fraud team, but *only* for loans over ₹10 lakh (SQS queue with a
  filter policy -> fraud-review-service)

One `publish()` call from the loan service triggers all four, independently,
without the loan service needing to know any of them exist. This is loose
coupling — you can add a fifth subscriber next year without touching the
loan service's code at all.

## 9.7 Common mistakes / gotchas

- Forgetting the SQS queue's **access policy** must explicitly allow the SNS
  topic to send to it — a very common "why isn't my message arriving"
  debugging session.
- Using SNS alone (no SQS behind it) for something that must survive a
  consumer outage — if a Lambda subscriber is down and retries are
  exhausted, the message can be lost; fan-out to SQS first if durability
  matters.
- Sending large payloads directly in SNS/SQS (there's a 256KB size limit) —
  instead, put the large object in S3 and pass its S3 key/URL in the
  message ("claim check pattern").

## 9.8 Interview Q&A

**Q: Explain SNS vs SQS in one line each.**
A: SQS = pull-based point-to-point queue, one message goes to one consumer.
SNS = push-based pub/sub topic, one message fans out to many subscribers at
once.

**Q: How do you make an SNS subscriber only receive some messages, not all?**
A: Attach a Filter Policy to that specific subscription, matching on message
attributes — the subscriber then only receives messages whose attributes
match the policy.

**Q: Why combine SNS with SQS instead of subscribing services directly to
SNS?**
A: SQS adds durability and buffering — if a consumer service is down, the
message waits safely in its queue instead of being lost, and each consumer
processes at its own pace.

## 9.9 Cheat sheet

```text
SNS = pub/sub, ONE message -> MANY subscribers (fan-out), push-based.
SQS = ONE message -> ONE consumer, pull-based.
Classic pattern: SNS Topic -> multiple SQS Queues -> independent services.
Filter Policy = lets a subscriber receive only a subset of messages.
Max message size ~256KB -- use S3 + "claim check" for bigger payloads.
```

---

# 10. Lambda — Serverless Functions

## 10.1 What is it?

**Lambda lets you run code without managing any server at all.** You upload
your code (or a container image), tell AWS when to trigger it (an API call,
a file upload to S3, a message in SQS, a schedule), and AWS runs it,
automatically scaling from zero to thousands of parallel executions, and you
pay only for the milliseconds it actually runs.

## 10.2 Why it exists

For EC2/ECS, you're paying for a server that's "on" 24/7 even if it's idle
90% of the time. For bursty, event-driven, or infrequent tasks (resize an
uploaded image, process one queue message, run a nightly reconciliation
job), Lambda is far cheaper and requires zero server management — no
patching, no capacity planning.

## 10.3 Key Concepts

| Concept | Meaning |
|---|---|
| **Function** | Your deployed code + configuration (memory, timeout, environment variables, IAM role). |
| **Handler** | The specific method Lambda calls to start execution, e.g. `com.bank.LoanHandler::handleRequest`. |
| **Trigger / Event Source** | What invokes the function — API Gateway, S3 event, SQS message, SNS message, EventBridge schedule, DynamoDB Stream, etc. |
| **Execution Role** | The IAM Role (Section 2) the function runs as — defines exactly what AWS resources the code can touch. |
| **Cold Start** | The extra latency (often 100ms-a few seconds) the *first* invocation after a period of inactivity takes, because AWS has to initialize a new execution environment. Subsequent invocations reuse a "warm" environment and are fast. Java historically has higher cold starts than Python/Node — a known trade-off to discuss in interviews. |
| **Timeout** | Max execution time allowed (up to 15 minutes) — after this, AWS kills the invocation. |
| **Memory** | You configure memory (128MB–10GB); CPU scales proportionally with memory — a common cost/performance tuning knob. |
| **Concurrency** | How many invocations run in parallel; you can set Reserved Concurrency (guarantee capacity, or cap it to protect downstream systems like a database from being overwhelmed). |
| **Layers** | Shared code/libraries you can attach to multiple functions without repackaging them into every deployment. |

## 10.4 Hands-on: A Java Lambda function

```java
// LoanNotificationHandler.java
// pom.xml needs: com.amazonaws:aws-lambda-java-core, aws-lambda-java-events

package com.bank.lambda;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.lambda.runtime.events.SQSEvent;
import software.amazon.awssdk.services.sns.SnsClient;
import software.amazon.awssdk.services.sns.model.PublishRequest;

/**
 * Triggered automatically every time a new message lands in the
 * "loan-approved-queue" SQS queue. Sends a customer notification via SNS.
 */
public class LoanNotificationHandler implements RequestHandler<SQSEvent, String> {

    // Initialized OUTSIDE handleRequest -- this runs ONCE per "warm" execution
    // environment, reused across invocations, saving time versus recreating
    // the client on every single call.
    private final SnsClient snsClient = SnsClient.create();
    private static final String TOPIC_ARN =
        System.getenv("NOTIFICATION_TOPIC_ARN"); // set as a Lambda environment variable

    @Override
    public String handleRequest(SQSEvent event, Context context) {
        for (SQSEvent.SQSMessage message : event.getRecords()) {
            context.getLogger().log("Processing message: " + message.getBody());

            snsClient.publish(PublishRequest.builder()
                    .topicArn(TOPIC_ARN)
                    .message("Your loan application has been approved: " + message.getBody())
                    .build());
        }
        return "Processed " + event.getRecords().size() + " messages";
    }
}
```

```bash
# Build a "fat jar" containing all dependencies (using the maven-shade-plugin
# configured in pom.xml), then deploy it:

mvn clean package

aws lambda create-function \
  --function-name loan-notification-handler \
  --runtime java17 \
  --handler com.bank.lambda.LoanNotificationHandler::handleRequest \
  --role arn:aws:iam::123456789012:role/lambda-loan-notification-role \
  --zip-file fileb://target/loan-notification-1.0.jar \
  --memory-size 512 \
  --timeout 30 \
  --environment "Variables={NOTIFICATION_TOPIC_ARN=arn:aws:sns:ap-south-1:123456789012:loan-events-topic}"

# Wire the SQS queue as the trigger
aws lambda create-event-source-mapping \
  --function-name loan-notification-handler \
  --event-source-arn arn:aws:sqs:ap-south-1:123456789012:loan-approved-queue \
  --batch-size 10
```

## 10.5 Invoking a Lambda directly (for testing) and via the Java SDK

```bash
aws lambda invoke \
  --function-name loan-notification-handler \
  --payload '{"test": true}' \
  --cli-binary-format raw-in-base64-out \
  response.json
cat response.json
```

```java
// Calling a Lambda function FROM another Java service (synchronous invocation)
import software.amazon.awssdk.services.lambda.LambdaClient;
import software.amazon.awssdk.services.lambda.model.InvokeRequest;
import software.amazon.awssdk.services.lambda.model.InvokeResponse;
import software.amazon.awssdk.core.SdkBytes;

public class LambdaInvoker {
    public static void main(String[] args) {
        try (LambdaClient lambdaClient = LambdaClient.create()) {
            InvokeRequest request = InvokeRequest.builder()
                    .functionName("loan-notification-handler")
                    .payload(SdkBytes.fromUtf8String("{\"test\": true}"))
                    .build();

            InvokeResponse response = lambdaClient.invoke(request);
            System.out.println(response.payload().asUtf8String());
        }
    }
}
```

## 10.6 When to use Lambda vs ECS/EC2 (a very common interview question)

```text
Use LAMBDA when:                       Use ECS/EC2 when:
- Event-driven, bursty, unpredictable  - Long-running, always-on services
- Short executions (< 15 min)          - Need > 15 minutes of processing
- Want zero server management          - Need fine-grained OS/runtime control
- Traffic is spiky or infrequent       - Steady, predictable, high traffic
                                          (can be cheaper at large constant scale)
- Simple, stateless units of work      - Need to maintain long-lived
                                          in-memory state/connections
```

For a core banking transaction-processing engine handling millions of steady
requests all day, ECS (Section 14) is often more cost-effective. For "resize
this image when uploaded" or "run this reconciliation job nightly," Lambda
is a much better fit.

## 10.7 Common mistakes / gotchas

- Creating a new SDK client (like `SnsClient.create()`) **inside**
  `handleRequest()` instead of outside it as a class field — wastes the
  "warm start" reuse benefit and slows every invocation.
- Not setting a Reserved Concurrency limit when a Lambda writes to RDS — an
  unexpected traffic spike could spin up thousands of concurrent
  invocations and exhaust your database's connection limit instantly.
- Ignoring cold starts for latency-sensitive synchronous APIs — Java has
  relatively higher cold-start latency; mitigations include Provisioned
  Concurrency (keeps N environments always warm, at extra cost) or choosing
  a lighter runtime for latency-critical paths.
- Giving the Lambda's execution role overly broad permissions — same least
  privilege principle as everywhere else.

## 10.8 Interview Q&A

**Q: What triggers a Lambda function?**
A: Many event sources — API Gateway requests, S3 object events, SQS/SNS
messages, DynamoDB Streams, EventBridge schedules/events, and more; each
maps the event payload into your function's input.

**Q: What's a cold start and how do you mitigate it?**
A: The extra latency when Lambda has to initialize a fresh execution
environment for the first invocation in a while. Mitigate with Provisioned
Concurrency (pre-warmed environments), smaller deployment packages, and
initializing SDK clients outside the handler so they're reused across warm
invocations.

**Q: How would you protect an RDS database from being overwhelmed by a
Lambda that scales to thousands of concurrent executions?**
A: Set Reserved Concurrency on the function to cap max parallel executions,
and/or put RDS Proxy in front of the database to pool connections
efficiently.

## 10.9 Cheat sheet

```text
Lambda = run code with NO server management, pay per millisecond, auto-scales.
Triggers: API Gateway, S3, SQS, SNS, EventBridge, DynamoDB Streams, etc.
Init SDK clients OUTSIDE handleRequest() for warm-start reuse.
Max timeout: 15 minutes. Memory 128MB-10GB (CPU scales with memory).
Use for: event-driven, bursty, short tasks. NOT for: long-running,
steady, high-throughput core services (use ECS/EC2 there).
```

---

# 11. API Gateway

## 11.1 What is it?

**API Gateway is a managed "front door" for your APIs.** It receives HTTP
requests from the internet, and routes them to backends like Lambda, ECS/EC2
(via a Load Balancer), or other AWS services — while also handling things
like authentication, rate limiting, request validation, and API versioning
for you.

## 11.2 Why it exists

You could expose your Spring Boot app directly to the internet, but then
your app itself has to handle throttling, API keys, request validation, CORS,
and auth — boilerplate you'd rather not maintain per-service. API Gateway
centralizes all of that in front of many backend services, and integrates
natively with Lambda for a fully serverless HTTP API.

## 11.3 Key Concepts

| Concept | Meaning |
|---|---|
| **REST API vs HTTP API** | Two API Gateway product types. HTTP API is newer, cheaper, faster, covers most common cases. REST API has more features (request/response transformation, more auth options, usage plans) — used when you need that extra control. |
| **Route / Resource + Method** | A URL path (`/loans/{id}`) combined with an HTTP verb (`GET`, `POST`) mapped to a specific backend integration. |
| **Integration** | What actually handles the request — Lambda function, HTTP backend (e.g. your ALB-fronted ECS service), or a direct AWS service integration. |
| **Stage** | A named deployment snapshot, e.g. `dev`, `staging`, `prod` — each can have different configuration/throttling and its own URL. |
| **Authorizer** | Controls who can call the API — IAM auth, Lambda custom authorizer (validate a custom token), or a **Cognito** User Pool authorizer (validate a JWT from a logged-in user). |
| **Throttling** | Rate limiting — protects your backend from being overwhelmed; configurable per API key/client. |
| **API Key + Usage Plan** | Issue different API keys to different partner integrations (e.g., a fintech partner calling your loan-status API) with different rate limits/quotas per key. |
| **CORS** | Cross-Origin Resource Sharing settings — needed if a browser-based frontend on a different domain calls your API. |

## 11.4 Hands-on: CLI — HTTP API in front of a Lambda function

```bash
# 1. Create the HTTP API, with a Lambda integration in one step
aws apigatewayv2 create-api \
  --name loan-status-api \
  --protocol-type HTTP \
  --target arn:aws:lambda:ap-south-1:123456789012:function:get-loan-status

# This single --target shortcut auto-creates a default route ($default) and
# a Lambda proxy integration. AWS also auto-adds the Lambda permission for
# API Gateway to invoke it, and gives you an "Invoke URL" immediately.

# 2. For more control, add explicit routes to an existing API
aws apigatewayv2 create-route \
  --api-id abc123xyz \
  --route-key "GET /loans/{loanId}" \
  --target integrations/xxxxx

# 3. Create a stage (e.g. "prod") with throttling
aws apigatewayv2 create-stage \
  --api-id abc123xyz \
  --stage-name prod \
  --auto-deploy \
  --default-route-settings ThrottlingBurstLimit=50,ThrottlingRateLimit=25
```

## 11.5 Java Lambda handling an API Gateway proxy request

```java
// GetLoanStatusHandler.java
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyResponseEvent;

import java.util.Map;

public class GetLoanStatusHandler
        implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {

    @Override
    public APIGatewayProxyResponseEvent handleRequest(
            APIGatewayProxyRequestEvent request, Context context) {

        // Path parameters (from "/loans/{loanId}") arrive here
        String loanId = request.getPathParameters().get("loanId");

        // In a real service, you'd look this up from RDS/DynamoDB
        String responseBody = String.format(
                "{\"loanId\": \"%s\", \"status\": \"APPROVED\"}", loanId);

        return new APIGatewayProxyResponseEvent()
                .withStatusCode(200)
                .withHeaders(Map.of("Content-Type", "application/json"))
                .withBody(responseBody);
    }
}
```

## 11.6 Fronting an ECS/Spring Boot service instead of Lambda

API Gateway can also forward to a **private** backend (an internal Load
Balancer in front of ECS/EC2) via a **VPC Link**, keeping your Spring Boot
services completely private (not internet-facing) while still exposing a
public, managed API surface with auth/throttling/logging centralized at the
gateway. This is a very common real-world pattern for microservices.

```bash
# Create a VPC Link so API Gateway can reach your private ALB
aws apigatewayv2 create-vpc-link \
  --name banking-vpc-link \
  --subnet-ids subnet-0appprivate1a subnet-0appprivate1b \
  --security-group-ids sg-0appxxxx

# Then create an integration pointing at your internal ALB listener, using that VPC Link
aws apigatewayv2 create-integration \
  --api-id abc123xyz \
  --integration-type HTTP_PROXY \
  --integration-uri arn:aws:elasticloadbalancing:ap-south-1:123456789012:listener/app/internal-alb/xxxx \
  --integration-method ANY \
  --connection-type VPC_LINK \
  --connection-id <vpc-link-id>
```

## 11.7 Common mistakes / gotchas

- Exposing internal microservices directly to the internet instead of
  routing everything through API Gateway — loses centralized auth,
  throttling, and monitoring.
- Not setting throttling limits — a traffic spike (or an attack) can
  overwhelm downstream services with no protection.
- Forgetting CORS configuration, causing browser-based frontends to fail
  with cryptic errors, even though `curl`/Postman work fine.
- Using IAM auth for a public-facing customer API (IAM auth is meant for
  service-to-service or AWS-internal calls, not end customers) — for
  customer-facing auth, use a Cognito authorizer or your own JWT-based
  Lambda authorizer instead.

## 11.8 Interview Q&A

**Q: Why put API Gateway in front of your microservices instead of exposing
them directly?**
A: Centralizes cross-cutting concerns — authentication, throttling, request
validation, logging/monitoring, and API versioning — without duplicating
that logic in every service, and keeps the actual backend services private.

**Q: HTTP API vs REST API — when would you choose REST API?**
A: When you need advanced features HTTP API doesn't support — e.g., request/
response data transformation (mapping templates), API keys with detailed
usage plans, or certain caching/authorizer options. Otherwise HTTP API is
cheaper and simpler for most modern use cases.

## 11.9 Cheat sheet

```text
API Gateway = managed front door for APIs: routing, auth, throttling, logging.
HTTP API = cheaper/simpler, most common choice today.
REST API = more features, higher cost, use when you need them.
Authorizers: IAM (service-to-service), Cognito (end users), Lambda custom (anything else)
VPC Link = lets API Gateway reach PRIVATE backends (internal ALB) securely.
```

---

# 12. Secrets Manager

## 12.1 What is it?

**Secrets Manager stores sensitive values (database passwords, API keys,
third-party credentials) securely, encrypted, and lets your application
fetch them at runtime** — instead of hardcoding them in config files, source
code, or environment variables that anyone with repo access could read.

## 12.2 Why it exists

If a database password is hardcoded in `application.properties` and that
file is ever committed to Git (it happens more than you'd think), the
password is compromised, possibly permanently (Git history). Secrets
Manager solves this by keeping the actual value out of your code entirely,
and can even **automatically rotate** the secret on a schedule (e.g., change
the RDS master password every 30 days) without any manual coordination or
application downtime, if configured with a rotation Lambda.

## 12.3 Key Concepts

| Concept | Meaning |
|---|---|
| **Secret** | A named, encrypted key-value blob (e.g., `{"username":"bankadmin","password":"..."}`) versioned automatically. |
| **Rotation** | Automatically generating a new secret value on a schedule and updating both Secrets Manager and the target system (e.g., RDS) — AWS provides built-in rotation Lambda templates for RDS, Redshift, DocumentDB. |
| **Resource Policy** | Controls which IAM principals can read/write a specific secret (in addition to normal IAM policies on the caller). |
| **Version Stages** | `AWSCURRENT` (the active version), `AWSPENDING` (being rotated in), `AWSPREVIOUS` (the last one) — this staging lets rotation happen with zero downtime. |
| **Secrets Manager vs Parameter Store** | Parameter Store (Section 17.5) is a simpler, cheaper key-value store — good for non-sensitive config and even some secrets at lower cost, but no built-in automatic rotation. Secrets Manager costs more per secret but adds rotation and tighter secret-specific auditing. Interviewers often ask you to justify the choice. |

## 12.4 Hands-on: CLI

```bash
# 1. Store a secret (e.g., the RDS master credentials)
aws secretsmanager create-secret \
  --name prod/banking-db/credentials \
  --description "Master credentials for banking-prod-db" \
  --secret-string '{"username":"bankadmin","password":"Sup3rS3cur3Passw0rd!"}'

# 2. Retrieve it (only works if the caller's IAM identity is permitted)
aws secretsmanager get-secret-value --secret-id prod/banking-db/credentials \
  --query 'SecretString' --output text

# 3. Enable automatic rotation every 30 days (using AWS's built-in RDS rotation Lambda)
aws secretsmanager rotate-secret \
  --secret-id prod/banking-db/credentials \
  --rotation-lambda-arn arn:aws:lambda:ap-south-1:123456789012:function:SecretsManagerRDSPostgreSQLRotationSingleUser \
  --rotation-rules AutomaticallyAfterDays=30
```

## 12.5 Java SDK + Spring Boot: fetching the DB password at startup

```java
// SecretsManagerConfig.java
// This runs BEFORE the DataSource bean is created, injecting the fetched
// password as a normal Spring property, so the rest of your app (JPA, etc.)
// never needs to know it came from Secrets Manager at all.

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import software.amazon.awssdk.services.secretsmanager.SecretsManagerClient;
import software.amazon.awssdk.services.secretsmanager.model.GetSecretValueRequest;
import software.amazon.awssdk.services.secretsmanager.model.GetSecretValueResponse;

@Configuration
public class SecretsManagerConfig {

    @Bean
    public DatabaseCredentials databaseCredentials() throws Exception {
        try (SecretsManagerClient client = SecretsManagerClient.create()) {
            GetSecretValueRequest request = GetSecretValueRequest.builder()
                    .secretId("prod/banking-db/credentials")
                    .build();

            GetSecretValueResponse response = client.getSecretValue(request);

            // The secret is stored as a JSON string, so parse it into fields
            ObjectMapper mapper = new ObjectMapper();
            JsonNode json = mapper.readTree(response.secretString());

            return new DatabaseCredentials(
                    json.get("username").asText(),
                    json.get("password").asText()
            );
        }
    }
}

record DatabaseCredentials(String username, String password) {}
```

```java
// DataSourceConfig.java -- wires the fetched credentials into the actual DataSource
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.zaxxer.hikari.HikariDataSource;

@Configuration
public class DataSourceConfig {

    @Autowired
    private DatabaseCredentials credentials;

    @Bean
    public HikariDataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:postgresql://banking-prod-db.xxxx.rds.amazonaws.com:5432/bankdb");
        dataSource.setUsername(credentials.username());
        dataSource.setPassword(credentials.password());
        return dataSource;
    }
}
```

## 12.6 IAM Policy: scoping access to exactly one secret

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:prod/banking-db/credentials-??????"
    }
  ]
}
```
The `-??????` suffix matches the random 6-character suffix AWS appends to
every secret's ARN — a subtle detail that trips people up when writing
tightly-scoped policies.

## 12.7 Real banking example: zero-downtime password rotation

```text
Day 0:   Secret has password "OldPass123", app fetches it at startup, connects fine.
Day 30:  Rotation Lambda triggers automatically:
           1. Generates a NEW password "NewPass456"
           2. Sets it as AWSPENDING in Secrets Manager
           3. Calls RDS to actually change the DB password to "NewPass456"
           4. Tests the new credentials work
           5. Promotes AWSPENDING to AWSCURRENT (old one becomes AWSPREVIOUS)
Day 30+: Any app that fetches the secret AFTER rotation gets "NewPass456" automatically.
         Apps using a long-lived connection pool don't need a restart if they
         re-fetch credentials periodically or on connection failure + retry.
```

No human ever typed or emailed the new password. No config file was edited.
No downtime.

## 12.8 Common mistakes / gotchas

- Fetching the secret on **every single request** instead of once at
  startup (or with a short cache) — unnecessarily increases latency and
  Secrets Manager API costs.
- Logging the secret value accidentally (e.g., logging the whole config
  object which includes the password) — always redact secrets from logs.
- Using Parameter Store's free tier for something that truly needs
  automatic rotation, then having to manually rotate it anyway — defeats
  the purpose; pick Secrets Manager when rotation matters.
- Not scoping the IAM policy to the *specific* secret ARN — using a
  wildcard lets that identity read every secret in the account.

## 12.9 Interview Q&A

**Q: Why not just put the DB password in an environment variable on ECS/EC2?**
A: Environment variables are visible to anyone who can describe the task
definition/instance, are harder to rotate without redeploying, and don't
have Secrets Manager's audit trail or built-in rotation support. Fetching
from Secrets Manager at startup keeps the actual secret value out of
infrastructure config entirely.

**Q: How does zero-downtime secret rotation work?**
A: Secrets Manager stages the new secret as `AWSPENDING`, has a rotation
Lambda update the actual target system (e.g., RDS) with the new value,
verifies it works, and only then promotes it to `AWSCURRENT` — the app
picks up the new value on its next fetch, without ever seeing a broken
intermediate state.

## 12.10 Cheat sheet

```text
Secrets Manager = encrypted secret storage + optional automatic rotation.
Never hardcode secrets in code/config/env vars if avoidable.
Fetch ONCE at startup (or cache briefly), not on every request.
Scope IAM policy to the SPECIFIC secret ARN (remember the random suffix).
Vs Parameter Store: SM = built-in rotation + higher cost; PS = cheaper, simpler, no auto-rotation.
```

---

# 13. Docker + ECR — Containerizing Your Application

## 13.1 What is Docker, in plain English?

**Docker packages your application together with everything it needs to run
(the Java runtime, OS libraries, config) into one portable unit called a
container image.** That image runs identically on your laptop, in a test
environment, or on AWS — "it works on my machine" stops being an excuse,
because the "machine" ships with the app.

## 13.2 What is ECR?

**ECR (Elastic Container Registry) is a private, managed Docker registry on
AWS** — like a private version of Docker Hub. You build your image, push it
to ECR, and services like ECS/EKS/Lambda pull it from there to run it.

## 13.3 Key Docker Concepts

| Concept | Meaning |
|---|---|
| **Image** | A read-only template/snapshot of your app + its dependencies. |
| **Container** | A running instance of an image — like an object is an instance of a class. |
| **Dockerfile** | A text file with step-by-step instructions to build an image. |
| **Layer** | Each instruction in a Dockerfile creates a cached layer; Docker reuses unchanged layers on rebuild, making builds fast. |
| **Multi-stage build** | Using multiple `FROM` stages in one Dockerfile so your final image only contains the compiled artifact, not the entire build toolchain — drastically smaller, more secure images. |
| **Registry** | Where images are stored/versioned (ECR, Docker Hub). |
| **Tag** | A label/version for an image, e.g. `loan-service:1.4.2` or `loan-service:latest` (avoid relying on `latest` in production — always use explicit, immutable version tags). |

## 13.4 A production-quality multi-stage Dockerfile for Spring Boot

```dockerfile
# ---------- Stage 1: Build ----------
# Use a full JDK + Maven image ONLY for building -- this stage will be discarded
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

# Copy pom.xml FIRST and download dependencies separately from the source code.
# Why: Docker caches layers -- if only your .java files change (not pom.xml),
# this dependency-download layer is REUSED from cache, making rebuilds much faster.
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Now copy the actual source code and build the jar
COPY src ./src
RUN mvn clean package -DskipTests -B

# ---------- Stage 2: Run ----------
# A much smaller image, containing ONLY a JRE (not the full JDK or Maven) --
# this is the image that actually gets deployed. Result: far smaller, fewer
# vulnerabilities (no build tools shipped to production), faster deploys.
FROM eclipse-temurin:17-jre-alpine

# Run as a non-root user -- a security best practice, never run containers as root
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

WORKDIR /app

# Copy ONLY the built jar from the "build" stage above -- nothing else
COPY --from=build /app/target/loan-service-*.jar app.jar

EXPOSE 8080

# JVM flags tuned for containers: respect the container's memory limit
# (not the host machine's), important since containers often get a fraction
# of the host's total RAM
ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

```dockerfile
# .dockerignore -- keeps the build context small and avoids leaking local files
target/
.git/
.idea/
*.iml
.env
```

## 13.5 Hands-on: Build, run locally, push to ECR

```bash
# 1. Build the image locally
docker build -t loan-service:1.0.0 .

# 2. Run it locally to test
docker run -p 8080:8080 \
  -e SPRING_DATASOURCE_URL="jdbc:postgresql://host.docker.internal:5432/bankdb" \
  loan-service:1.0.0

# 3. Create an ECR repository (once)
aws ecr create-repository \
  --repository-name loan-service \
  --image-scanning-configuration scanOnPush=true \
  --encryption-configuration encryptionType=KMS

# 4. Authenticate Docker to ECR (token expires after 12 hours, re-run as needed)
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com

# 5. Tag the image with the full ECR repository URI
docker tag loan-service:1.0.0 \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/loan-service:1.0.0

# 6. Push it
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/loan-service:1.0.0

# 7. View vulnerability scan results (enabled via scanOnPush above)
aws ecr describe-image-scan-findings \
  --repository-name loan-service --image-id imageTag=1.0.0
```

## 13.6 Common mistakes / gotchas

- Using a single-stage Dockerfile that ships the whole JDK + Maven + source
  code into production — bloated image, larger attack surface, slower
  deploys.
- Running the container as root — if the app is ever compromised, the
  attacker has root inside the container.
- Using the `latest` tag in production deployments — makes rollbacks and
  audits ambiguous ("which actual code is running right now?"); always use
  immutable version tags (and optionally the image digest).
- Not setting `-XX:MaxRAMPercentage` — the JVM historically didn't know it
  was inside a memory-limited container and could get OOM-killed
  unexpectedly; modern JDKs handle this better by default but it's still
  worth being explicit.
- Forgetting `.dockerignore` — accidentally baking `.env` files with secrets
  into the image layers.

## 13.7 Interview Q&A

**Q: Why use a multi-stage Docker build?**
A: To keep the final production image small and secure — the build tools
(Maven, full JDK, source code) are used only in an intermediate stage and
discarded; only the compiled artifact and a minimal runtime make it into the
final image.

**Q: What's the difference between an image and a container?**
A: An image is the static, read-only template. A container is a running
(or stopped) instance created from that image — you can run many containers
from the same image simultaneously.

## 13.8 Cheat sheet

```text
Docker = package app + runtime into a portable image.
ECR = private, managed Docker registry on AWS.
Multi-stage build: separate BUILD stage (heavy tools) from RUN stage (minimal).
Always: non-root user, immutable version tags, .dockerignore, scanOnPush=true.
```

---

# 14. ECS — Elastic Container Service

## 14.1 What is it?

**ECS runs and manages your Docker containers at scale** — it decides where
containers run, restarts them if they crash, and can scale the number of
running copies up or down. Think of it as the "operations manager" for your
containers, so you don't manually SSH into servers to start/stop/monitor
them.

## 14.2 Fargate vs EC2 launch type — the first big decision

| | **Fargate (serverless)** | **EC2 launch type** |
|---|---|---|
| Who manages the underlying servers? | AWS — fully serverless, you never see an EC2 instance | You — you provision and manage the EC2 instances ECS places containers on |
| Best for | Most modern applications; simpler ops | Very cost-sensitive at large scale, or needing specific instance types/GPUs |
| Cost model | Pay per vCPU/memory reserved per task, per second | Pay for the EC2 instances regardless of container packing efficiency |

For most teams starting out (and for interviews), **Fargate is the default
recommendation** — you focus on your containers, not server fleet
management.

## 14.3 Key Concepts

| Concept | Meaning |
|---|---|
| **Cluster** | A logical grouping of tasks/services — doesn't "contain" servers itself in Fargate mode, it's just an organizational boundary. |
| **Task Definition** | A JSON blueprint describing one or more containers that run together — image, CPU/memory, environment variables, IAM roles, logging config. Like a "docker-compose file" AWS understands. |
| **Task** | One running instance of a Task Definition. |
| **Service** | Keeps a desired number of Tasks running continuously, replacing any that crash, and can integrate with a Load Balancer and Auto Scaling. |
| **Task Role** | The IAM Role your **application code inside the container** uses to call other AWS services (e.g., S3, DynamoDB) — this is the container-level equivalent of an EC2 instance profile. |
| **Task Execution Role** | A *different* role used by the **ECS agent itself** to pull the image from ECR and write logs to CloudWatch — commonly confused with the Task Role above, a classic interview trap. |
| **Service Auto Scaling** | Scaling the number of running Tasks in a Service based on a metric (CPU%, request count) — see Section 15 for the full picture including the Load Balancer side. |

## 14.4 Hands-on: Task Definition + Service via CLI

```json
// task-definition.json
{
  "family": "loan-service",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/loanServiceTaskRole",
  "containerDefinitions": [
    {
      "name": "loan-service",
      "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/loan-service:1.0.0",
      "portMappings": [{ "containerPort": 8080, "protocol": "tcp" }],
      "environment": [
        { "name": "SPRING_PROFILES_ACTIVE", "value": "prod" }
      ],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:prod/banking-db/credentials-abc123:password::"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/loan-service",
          "awslogs-region": "ap-south-1",
          "awslogs-stream-prefix": "loan-service"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8080/actuator/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

Note the `"secrets"` block — ECS natively pulls the DB password directly
from Secrets Manager and injects it as an environment variable at container
startup, so it's never baked into the image or the task definition itself.

```bash
# 1. Register the task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# 2. Create the cluster
aws ecs create-cluster --cluster-name banking-cluster

# 3. Create the service, running 2 copies, attached to a Load Balancer target group
aws ecs create-service \
  --cluster banking-cluster \
  --service-name loan-service \
  --task-definition loan-service \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-0app1a,subnet-0app1b],securityGroups=[sg-0appxxxx],assignPublicIp=DISABLED}" \
  --load-balancers "targetGroupArn=arn:aws:elasticloadbalancing:ap-south-1:123456789012:targetgroup/loan-tg/xxxx,containerName=loan-service,containerPort=8080" \
  --health-check-grace-period-seconds 60
```

## 14.5 Spring Boot Actuator — powering ECS health checks

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
# application.properties
management.endpoints.web.exposure.include=health,info
management.endpoint.health.show-details=when-authorized
# This exposes GET /actuator/health, which the Task Definition's healthCheck
# command (and the Load Balancer's health check, Section 15) both call to
# know whether a Task is actually ready to receive traffic.
```

## 14.6 Real banking example: safe rolling deployments

```text
Deploying loan-service version 1.0.1 (was running 1.0.0, desired count = 4):

1. ECS registers a NEW task definition revision pointing at image tag 1.0.1
2. `aws ecs update-service --task-definition loan-service:REVISION_N`
3. ECS starts NEW tasks running 1.0.1 (respecting minimumHealthyPercent, e.g. 100%
   means it won't kill an old task until a new one is healthy)
4. Each new task must pass its healthCheck (/actuator/health) before it's
   registered with the Load Balancer's target group and receives real traffic
5. Once new tasks are healthy and serving traffic, ECS gradually stops the
   old 1.0.0 tasks (respecting maximumPercent, e.g. 200% allows temporarily
   running double capacity during the swap)
6. If new tasks keep failing health checks, ECS can be configured to
   automatically roll back to the previous stable task definition
```

This "rolling deployment with health checks" is exactly how banking systems
achieve zero-downtime deploys — no maintenance windows needed.

## 14.7 Common mistakes / gotchas

- Confusing **Task Role** (what your app code can access) with **Task
  Execution Role** (what ECS itself needs to pull images/write logs) — a
  very frequent point of confusion and interview question.
- Not setting a proper health check — ECS will happily send traffic to a
  container that's technically "running" but not actually ready (e.g. still
  connecting to the DB), causing errors.
- Setting `desired-count: 1` for a production service — no redundancy; if
  that one task's AZ has an issue, you have zero capacity. Always run at
  least 2 tasks across 2 AZs.
- Forgetting `assignPublicIp=DISABLED` for private services — tasks meant to
  be internal-only shouldn't get public IPs; they should sit behind an
  internal Load Balancer.

## 14.8 Interview Q&A

**Q: Fargate vs EC2 launch type — what's the tradeoff?**
A: Fargate is serverless — AWS manages the underlying compute, you just
specify CPU/memory per task, simpler ops but sometimes higher per-unit cost.
EC2 launch type means you manage the underlying instances yourself — more
control and potentially cheaper at large, steady scale, but more
operational overhead (patching, capacity planning).

**Q: What's the difference between the Task Role and the Task Execution
Role?**
A: The Task Execution Role is used by the ECS agent to do infrastructure
work — pull the container image from ECR, fetch secrets, write logs to
CloudWatch. The Task Role is assumed by your actual application code inside
the running container to call other AWS APIs (e.g., read from S3). They're
often different roles with very different permission scopes.

**Q: How does ECS achieve zero-downtime deployment?**
A: Rolling deployments — new tasks with the updated task definition are
started and must pass health checks before being added to the Load
Balancer's target group; only then are old tasks drained and stopped,
respecting configured minimum/maximum healthy percentages.

## 14.9 Cheat sheet

```text
ECS = manages running Docker containers at scale.
Fargate = serverless (AWS manages servers). EC2 launch type = you manage servers.
Task Definition = blueprint (image, CPU/mem, roles, logging, secrets, health check)
Task = one running instance of a Task Definition
Service = keeps N Tasks running, integrates with Load Balancer + Auto Scaling
Task Role (app permissions) != Task Execution Role (ECS agent permissions)
Always: >=2 tasks across >=2 AZs, health checks configured, secrets via Secrets Manager.
```

---

# 15. Auto Scaling + Elastic Load Balancer

## 15.1 What are they?

**Elastic Load Balancer (ELB)** sits in front of multiple copies of your
application and spreads incoming traffic across them, while continuously
health-checking each one and routing away from unhealthy copies.

**Auto Scaling** automatically adds or removes copies of your application
(EC2 instances, or ECS Tasks) based on real-time demand, so you have enough
capacity during peak load and aren't paying for idle capacity overnight.

Together, they're the standard way to run a highly available, elastic
application on AWS.

## 15.2 Types of Load Balancer

| Type | Layer | Use case |
|---|---|---|
| **Application Load Balancer (ALB)** | Layer 7 (HTTP/HTTPS) | Most web apps and APIs — can route based on URL path/host header, supports WebSockets, integrates natively with ECS. **This is the one you'll use 90% of the time.** |
| **Network Load Balancer (NLB)** | Layer 4 (TCP/UDP) | Extreme performance/low latency, static IP requirements, non-HTTP protocols. |
| **Gateway Load Balancer (GWLB)** | Layer 3/4 | Deploying third-party virtual network appliances (firewalls) — niche, good to know exists. |

## 15.3 Key Concepts

| Concept | Meaning |
|---|---|
| **Listener** | Checks for connection requests on a specific port/protocol (e.g., HTTPS:443) and applies rules to route them. |
| **Target Group** | The set of "backends" (EC2 instances, ECS Tasks, IPs, or even Lambda functions) the load balancer routes traffic to, along with its health check config. |
| **Health Check** | The Load Balancer periodically calls a specific path (e.g., `/actuator/health`) on each target; unhealthy targets stop receiving new traffic automatically. |
| **Auto Scaling Group (ASG)** | (For EC2) A group that maintains a desired number of EC2 instances, launching/terminating them automatically per scaling policies, spread across AZs. |
| **Launch Template** | Defines what a new EC2 instance in an ASG looks like (AMI, instance type, security groups, user data) — think of it as a "cookie cutter" for new instances. |
| **Scaling Policy** | The rule that decides when to scale, e.g. **Target Tracking** ("keep average CPU at 50%" — AWS figures out add/remove counts automatically), **Step Scaling** (add N instances when a CloudWatch alarm breaches a threshold), or **Scheduled Scaling** (e.g., "scale up every weekday 9 AM for salary-day traffic"). |
| **Cooldown Period** | A pause after a scaling action before another one can trigger, preventing rapid "flapping" (scale up, down, up, down). |

## 15.4 Hands-on: CLI — ALB + Target Group + ECS Service Auto Scaling

```bash
# 1. Create an Application Load Balancer (public-facing, in public subnets)
aws elbv2 create-load-balancer \
  --name banking-alb \
  --subnets subnet-0public1a subnet-0public1b \
  --security-groups sg-0webxxxx \
  --scheme internet-facing \
  --type application

# 2. Create a Target Group with a health check pointing at Spring Boot Actuator
aws elbv2 create-target-group \
  --name loan-tg \
  --protocol HTTP --port 8080 \
  --vpc-id $VPC_ID \
  --target-type ip \
  --health-check-path /actuator/health \
  --health-check-interval-seconds 30 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3

# 3. Create an HTTPS listener (requires an ACM certificate, Section 17.10)
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:ap-south-1:123456789012:loadbalancer/app/banking-alb/xxxx \
  --protocol HTTPS --port 443 \
  --certificates CertificateArn=arn:aws:acm:ap-south-1:123456789012:certificate/xxxx \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:ap-south-1:123456789012:targetgroup/loan-tg/xxxx

# 4. Enable ECS Service Auto Scaling: keep average CPU around 50%
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/banking-cluster/loan-service \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 2 --max-capacity 10

cat > scaling-policy.json << 'EOF'
{
  "TargetValue": 50.0,
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
  },
  "ScaleInCooldown": 120,
  "ScaleOutCooldown": 60
}
EOF

aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --resource-id service/banking-cluster/loan-service \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration file://scaling-policy.json
```

Notice `ScaleOutCooldown: 60` (add capacity quickly, 1 minute) is shorter
than `ScaleInCooldown: 120` (remove capacity more cautiously, 2 minutes) —
a deliberate, common pattern: **scale out fast to protect customer
experience, scale in slowly to avoid flapping.**

## 15.5 EC2 Auto Scaling Group (if not using ECS/Fargate)

```bash
# 1. Create a Launch Template
aws ec2 create-launch-template \
  --launch-template-name loan-app-template \
  --launch-template-data '{
    "ImageId": "ami-0abcdef1234567890",
    "InstanceType": "t3.medium",
    "IamInstanceProfile": {"Name": "EC2-S3-ReadOnly-Profile"},
    "SecurityGroupIds": ["sg-0appxxxx"],
    "UserData": "'"$(base64 -w0 user-data.sh)"'"
  }'

# 2. Create the Auto Scaling Group across 2 AZs, attached to the target group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name loan-app-asg \
  --launch-template LaunchTemplateName=loan-app-template,Version='$Latest' \
  --min-size 2 --max-size 10 --desired-capacity 2 \
  --vpc-zone-identifier "subnet-0app1a,subnet-0app1b" \
  --target-group-arns arn:aws:elasticloadbalancing:ap-south-1:123456789012:targetgroup/loan-tg/xxxx \
  --health-check-type ELB \
  --health-check-grace-period 60

# 3. Target tracking scaling policy (same idea as ECS above)
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name loan-app-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 50.0,
    "PredefinedMetricSpecification": {"PredefinedMetricType": "ASGAverageCPUUtilization"}
  }'
```

## 15.6 Real banking example: Salary day traffic

```text
Normal day:  2 tasks running, handling baseline traffic comfortably.
Salary day:  Balance-check and transfer requests spike 10x within minutes.
             CloudWatch detects average CPU across tasks crossing 50% target.
             Target tracking policy scales OUT quickly (60s cooldown) --
             new tasks launch, pass health checks, join the ALB's target group.
             ALB automatically starts routing to the new healthy tasks.
             Traffic subsides in the evening.
             Target tracking scales IN slowly (120s cooldown) to avoid
             prematurely removing capacity if traffic briefly dips then returns.
```

No 2 AM page to an engineer to manually add servers — the system handles it
itself, within the min/max bounds you configured.

## 15.7 Common mistakes / gotchas

- Setting `min-size` too low (e.g., 1) for a production service — no
  redundancy if that single instance/task fails or its AZ has issues.
- Health check path pointing at something that doesn't reflect true
  readiness (e.g., just `/`, which might respond even if the DB connection
  is down) — always use a proper `/health` endpoint that checks real
  dependencies.
- Symmetric cooldowns (same value for scale-in and scale-out) — usually
  suboptimal; scale out fast, scale in cautiously, as shown above.
- Forgetting the ALB's Security Group must allow inbound from the internet
  (443), while the Target's Security Group should only allow inbound from
  the ALB's Security Group — not from the whole internet directly.

## 15.8 Interview Q&A

**Q: How does an ALB know a backend instance/task is healthy?**
A: It periodically calls a configured health check path (e.g.,
`/actuator/health`) on each target; a target must pass a set number of
consecutive successful checks to be marked healthy (and receive traffic),
and fail a set number to be marked unhealthy (and be removed from
rotation).

**Q: What's Target Tracking scaling, and why is it usually preferred over
Step Scaling?**
A: Target Tracking lets you declare a goal (e.g., "keep average CPU at
50%"), and AWS automatically calculates how much to scale in/out to hit
that target — simpler to reason about and self-tuning, versus Step Scaling
where you manually define exact add/remove amounts per alarm threshold.

**Q: Why would scale-out and scale-in cooldowns typically differ?**
A: You generally want to react to increased load quickly (protect customer
experience, short scale-out cooldown) but remove capacity more cautiously
(avoid "flapping" if load is just briefly dipping, longer scale-in
cooldown).

## 15.9 Cheat sheet

```text
ALB = Layer 7 (HTTP), most common. NLB = Layer 4, high performance. GWLB = firewalls.
Target Group = backends + health check config.
ASG (EC2) / Service Auto Scaling (ECS) = automatically adjust running capacity.
Target Tracking = "keep metric at X", AWS calculates the scaling for you.
Scale out FAST (short cooldown), scale in SLOW (long cooldown).
Always: min capacity >= 2, spread across >= 2 AZs, real health check endpoint.
```

---

# 16. CodePipeline, CodeBuild, CodeDeploy — CI/CD on AWS

## 16.1 What are they?

Three services that together automate "code committed → tested → built →
deployed to production" without a human manually running commands:

- **CodePipeline**: The overall orchestrator — defines the *stages*
  (Source → Build → Test → Deploy) and moves your code through them
  automatically on every commit.
- **CodeBuild**: A managed build service — compiles your code, runs tests,
  builds your Docker image, pushes it to ECR. (Think: a managed Jenkins
  build agent, no server for you to maintain.)
- **CodeDeploy**: Handles the actual deployment mechanics to EC2, ECS, or
  Lambda — including safer deployment strategies (blue/green, canary).

## 16.2 Why it exists

Manual deployment ("SSH in, git pull, restart the app") doesn't scale, is
error-prone, and is a compliance red flag in banking (auditors want to see
consistent, automated, reviewed deployment processes with no manual
production access needed). CI/CD pipelines make every deployment identical,
repeatable, and auditable — the exact same process every single time.

## 16.3 Key Concepts

| Concept | Meaning |
|---|---|
| **Pipeline** | The full end-to-end automated workflow definition. |
| **Stage** | A phase in the pipeline (Source, Build, Test, Deploy) — must complete successfully before moving to the next. |
| **Action** | A specific task within a stage (e.g., "pull from GitHub", "run CodeBuild project X"). |
| **Artifact** | The output of one stage, passed as input to the next (e.g., the built jar/Docker image passed from Build stage to Deploy stage). |
| **buildspec.yml** | The CodeBuild configuration file (lives in your repo) defining build commands step-by-step. |
| **appspec.yml** | The CodeDeploy configuration file defining how to deploy (used for EC2/Lambda deployments). |
| **Blue/Green Deployment** | Deploy the new version alongside the old ("green" is new, "blue" is old), shift traffic gradually or all-at-once, and keep the old version ready for instant rollback if something's wrong. ECS supports this natively via CodeDeploy. |
| **Canary Deployment** | Route a small percentage of traffic (e.g., 10%) to the new version first, monitor for errors, then gradually increase to 100% — catches bad deployments before they affect everyone. |

## 16.4 Hands-on: buildspec.yml for a Spring Boot + Docker pipeline

```yaml
# buildspec.yml -- lives in the root of your Git repository
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto17
  pre_build:
    commands:
      - echo "Logging in to ECR..."
      - aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com
      - REPOSITORY_URI=123456789012.dkr.ecr.ap-south-1.amazonaws.com/loan-service
      # Use the short Git commit hash as the immutable image tag -- NEVER "latest"
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:-latest}
  build:
    commands:
      - echo "Running unit tests..."
      - mvn clean test
      - echo "Building the application jar..."
      - mvn clean package -DskipTests
      - echo "Building Docker image..."
      - docker build -t $REPOSITORY_URI:$IMAGE_TAG .
      - docker tag $REPOSITORY_URI:$IMAGE_TAG $REPOSITORY_URI:latest
  post_build:
    commands:
      - echo "Pushing Docker image to ECR..."
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - docker push $REPOSITORY_URI:latest
      # Write an "imagedefinitions.json" -- CodeDeploy/ECS deploy stage reads
      # this to know exactly which new image tag to roll out
      - printf '[{"name":"loan-service","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
    - appspec.yaml
    - taskdef.json
```

```yaml
# appspec.yaml -- for an ECS blue/green deployment via CodeDeploy
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>   # CodeDeploy substitutes this at deploy time
        LoadBalancerInfo:
          ContainerName: "loan-service"
          ContainerPort: 8080
```

## 16.5 Creating the pipeline via CLI

```bash
# 1. Create the CodeBuild project
aws codebuild create-project \
  --name loan-service-build \
  --source type=GITHUB,location=https://github.com/ananth/loan-service \
  --artifacts type=CODEPIPELINE \
  --environment type=LINUX_CONTAINER,image=aws/codebuild/amazonlinux2-x86_64-standard:5.0,computeType=BUILD_GENERAL1_SMALL,privilegedMode=true \
  --service-role arn:aws:iam::123456789012:role/codebuild-service-role
  # privilegedMode=true is required because this build runs "docker build" itself

# 2. Create the pipeline (simplified; full JSON structure has Source/Build/Deploy stages)
aws codepipeline create-pipeline --cli-input-json file://pipeline-definition.json
```

```json
// pipeline-definition.json (structure overview)
{
  "pipeline": {
    "name": "loan-service-pipeline",
    "roleArn": "arn:aws:iam::123456789012:role/codepipeline-service-role",
    "artifactStore": { "type": "S3", "location": "loan-service-pipeline-artifacts" },
    "stages": [
      {
        "name": "Source",
        "actions": [{
          "name": "GitHubSource",
          "actionTypeId": { "category": "Source", "owner": "AWS", "provider": "CodeStarSourceConnection", "version": "1" },
          "outputArtifacts": [{ "name": "SourceOutput" }],
          "configuration": {
            "ConnectionArn": "arn:aws:codestar-connections:ap-south-1:123456789012:connection/xxxx",
            "FullRepositoryId": "ananth/loan-service",
            "BranchName": "main"
          }
        }]
      },
      {
        "name": "Build",
        "actions": [{
          "name": "DockerBuildAndPush",
          "actionTypeId": { "category": "Build", "owner": "AWS", "provider": "CodeBuild", "version": "1" },
          "inputArtifacts": [{ "name": "SourceOutput" }],
          "outputArtifacts": [{ "name": "BuildOutput" }],
          "configuration": { "ProjectName": "loan-service-build" }
        }]
      },
      {
        "name": "Deploy",
        "actions": [{
          "name": "ECSBlueGreenDeploy",
          "actionTypeId": { "category": "Deploy", "owner": "AWS", "provider": "CodeDeployToECS", "version": "1" },
          "inputArtifacts": [{ "name": "BuildOutput" }],
          "configuration": {
            "ApplicationName": "loan-service-app",
            "DeploymentGroupName": "loan-service-dg"
          }
        }]
      }
    ]
  }
}
```

## 16.6 Real banking example: the full journey of one commit

```text
1. Developer pushes code to `main` branch on GitHub.
2. CodePipeline's Source stage detects the change, pulls the code.
3. Build stage (CodeBuild) runs: `mvn test` (unit tests must pass to continue),
   builds the Docker image, pushes to ECR with an immutable commit-hash tag.
4. (Optional Test stage) Deploys to a staging ECS cluster, runs integration
   tests / security scans against it.
5. Deploy stage (CodeDeploy, Blue/Green): 
   - Spins up NEW ECS tasks running the new image ("green")
   - Runs health checks against green
   - Shifts a SMALL percentage of live traffic to green first (canary-style),
     watches CloudWatch alarms for elevated error rates
   - If healthy, gradually shifts 100% of traffic to green
   - Keeps the OLD ("blue") tasks running for a short bake time, ready for
     an INSTANT rollback (just shift traffic back) if something's wrong
   - Once confident, terminates the blue tasks
6. Slack/SNS notification confirms successful deployment, with the commit hash
   and who approved it (for audit trail -- important for banking compliance).
```

Every step is logged, versioned, and requires zero manual server access —
exactly what a banking security/compliance review wants to see.

## 16.7 Common mistakes / gotchas

- Skipping the Test stage to "move fast" — defeats the entire purpose of CI/CD.
- Not using immutable image tags (commit hash) — makes rollback and audit
  ambiguous.
- Giving the CodeBuild/CodePipeline service roles overly broad permissions
  (e.g., full AdministratorAccess) instead of scoping to exactly what's
  needed (push to this ECR repo, deploy to this ECS service).
- No automated rollback trigger on elevated error rates during Blue/Green —
  should be tied to CloudWatch Alarms so a bad deploy self-heals without
  waiting for a human to notice.

## 16.8 Interview Q&A

**Q: Blue/Green vs Canary deployment — what's the difference?**
A: Blue/Green runs the new version fully alongside the old, then cuts over
traffic (often all at once, or with a brief linear shift), keeping the old
version on standby for instant rollback. Canary shifts a small percentage
of traffic to the new version first, monitors it, then gradually increases
— catching problems while limiting the blast radius to a small fraction of
users before going further.

**Q: What's the role of buildspec.yml vs appspec.yaml?**
A: `buildspec.yml` tells CodeBuild *how to build* your artifact (compile,
test, build/push a Docker image). `appspec.yaml` tells CodeDeploy *how to
deploy* that artifact (e.g., which ECS service, which container/port to
route to during a blue/green shift).

## 16.9 Cheat sheet

```text
CodePipeline = orchestrates stages (Source -> Build -> Test -> Deploy)
CodeBuild    = managed build service (buildspec.yml)
CodeDeploy   = manages deployment mechanics (appspec.yaml), supports Blue/Green & Canary
Always: automated tests must gate deployment, immutable image tags,
        least-privilege service roles, automated rollback on error-rate alarms.
```

---

# 17. Bonus Must-Know Services

The 15 services above will get you through most AWS work. But real
architectures — and interviews — routinely touch these additional services.
Here they are, at the same depth for the ones you'll use constantly, and
briefer for niche ones.

## 17.1 Route 53 — DNS

**What**: AWS's DNS (Domain Name System) service — translates human-friendly
domain names (`api.mybank.com`) into IP addresses, and can also register
domain names.

**Key concepts**:
- **Hosted Zone**: A container for DNS records for one domain.
- **Record types**: `A` (domain → IPv4), `CNAME` (domain → another domain
  name), **`Alias`** (AWS-specific, like a CNAME but works at the zone apex
  and is free to query — use this to point your domain at an ALB/CloudFront).
- **Routing policies**: Simple (one answer), Weighted (split traffic by
  percentage — useful for gradual migrations), Latency-based (route to the
  region with lowest latency for that user), Failover (route to a standby
  if the primary's health check fails), Geolocation (route based on user's
  location — useful for data-residency requirements).
- **Health Checks**: Route 53 can monitor an endpoint and automatically stop
  routing to it if unhealthy (used heavily with Failover routing).

```bash
# Point api.mybank.com at your ALB using an Alias record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456ABCDEF \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "api.mybank.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z2ABCXYZALBZONE",
          "DNSName": "banking-alb-123456.ap-south-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'
```

**Interview Q**: *Alias record vs CNAME — why prefer Alias for pointing at
an ALB?* — Alias records work at the zone apex (root domain, e.g.
`mybank.com` with no subdomain, which CNAME cannot do per DNS spec), are
free to query, and Route 53 automatically updates them if the underlying
ALB's IP changes.

## 17.2 CloudFront — CDN

**What**: A Content Delivery Network — caches your content (images, CSS/JS,
even API responses) at 400+ edge locations worldwide, so users get it from a
location physically close to them instead of your origin server, reducing
latency and offloading traffic from your backend.

**Key concepts**:
- **Distribution**: The CloudFront configuration tying an Origin (S3
  bucket, ALB, custom HTTP backend) to caching behavior.
- **Origin**: Where the real content lives.
- **Cache Behavior**: Rules per URL path pattern (e.g., cache `/static/*`
  for 24 hours, never cache `/api/*`).
- **TTL (Time To Live)**: How long content stays cached at the edge before
  CloudFront re-checks the origin.
- **OAC (Origin Access Control)**: Ensures an S3-origin bucket can ONLY be
  reached through CloudFront, not directly — combine with Block Public
  Access on the bucket for a fully private-but-globally-served setup.

```bash
aws cloudfront create-distribution --distribution-config '{
  "CallerReference": "banking-static-2026",
  "Origins": {
    "Quantity": 1,
    "Items": [{
      "Id": "s3-static-origin",
      "DomainName": "banking-static-assets.s3.ap-south-1.amazonaws.com",
      "S3OriginConfig": { "OriginAccessIdentity": "" }
    }]
  },
  "DefaultCacheBehavior": {
    "TargetOriginId": "s3-static-origin",
    "ViewerProtocolPolicy": "redirect-to-https",
    "AllowedMethods": { "Quantity": 2, "Items": ["GET", "HEAD"] },
    "CachePolicyId": "658327ea-f89d-4fab-a63d-7e88639e58f6"
  },
  "Enabled": true
}'
```

For a banking app, you'd typically use CloudFront for static frontend
assets (React/Angular build files), NOT for sensitive transaction API
responses which should never be cached at the edge.

## 17.3 DynamoDB — NoSQL

**What**: A fully managed, serverless NoSQL key-value/document database —
insanely fast (single-digit millisecond latency) at any scale, no servers to
manage, pay per request or per provisioned throughput.

**When to use over RDS**: When your access pattern is simple key-based
lookups at massive scale (session storage, shopping carts, real-time
leaderboards, IoT event logs) rather than complex joins/transactions across
many tables — for that, RDS/Aurora is still the better fit. Many real
systems use **both**: RDS for core relational/transactional banking data
(accounts, ledgers), DynamoDB for high-throughput, simple-access-pattern
data (session tokens, audit event streams, rate-limit counters).

**Key concepts**:
- **Table**: Like a SQL table, but schemaless except for the key.
- **Partition Key (and optional Sort Key)**: Determines how data is
  distributed across DynamoDB's underlying storage partitions — designing
  this well is the single most important DynamoDB skill (a poorly chosen
  key causes "hot partitions" that bottleneck performance).
- **On-Demand vs Provisioned capacity**: On-Demand auto-scales and you pay
  per request (simpler, good for unpredictable traffic); Provisioned lets
  you reserve throughput (cheaper at steady, predictable, high volume).
- **DynamoDB Streams**: A time-ordered log of every change to a table — can
  trigger a Lambda for every insert/update/delete (event-driven patterns).
- **Global Secondary Index (GSI)**: Query the table by an attribute other
  than the primary key.

```java
// Java SDK v2 — storing a session token
import software.amazon.awssdk.services.dynamodb.DynamoDbClient;
import software.amazon.awssdk.services.dynamodb.model.*;
import java.util.Map;

public class SessionStore {
    private final DynamoDbClient dynamoDb = DynamoDbClient.create();

    public void saveSession(String sessionId, String customerId, long expiresAtEpoch) {
        Map<String, AttributeValue> item = Map.of(
            "sessionId", AttributeValue.builder().s(sessionId).build(),   // Partition Key
            "customerId", AttributeValue.builder().s(customerId).build(),
            "expiresAt", AttributeValue.builder().n(String.valueOf(expiresAtEpoch)).build()
        );

        dynamoDb.putItem(PutItemRequest.builder()
                .tableName("customer-sessions")
                .item(item)
                .build());
    }
}
```

## 17.4 KMS — Key Management Service

**What**: Create and manage encryption keys used to encrypt data across
almost every AWS service (S3, RDS, EBS, Secrets Manager) — critical for
banking, where encryption at rest is often a regulatory requirement (PCI-DSS,
RBI guidelines).

**Key concepts**:
- **CMK (Customer Master Key)**: The root key you manage (AWS-managed,
  or fully customer-managed for tighter control over rotation/policy).
- **Envelope Encryption**: KMS doesn't directly encrypt your (potentially
  huge) data — it encrypts a small "data key," which then encrypts your
  actual data locally, far more efficient than sending gigabytes through
  KMS's API.
- **Key Policy**: Like an IAM policy, but attached to the key itself,
  controlling who can use/manage it.
- **Automatic key rotation**: KMS can rotate the underlying key material
  yearly without you needing to re-encrypt existing data.

```bash
aws kms create-key --description "Banking customer data encryption key"
aws kms create-alias --alias-name alias/banking-customer-data --target-key-id <key-id>

# Use it when creating an encrypted RDS instance, S3 bucket, or EBS volume:
aws rds create-db-instance ... --storage-encrypted --kms-key-id alias/banking-customer-data
```

## 17.5 Systems Manager — Parameter Store & Session Manager

**What**: A grab-bag of operational tools; the two most-used features:

- **Parameter Store**: A simple, free (for standard tier) key-value store
  for configuration (and even some secrets, though Secrets Manager is
  preferred for anything needing rotation).
- **Session Manager**: Lets you get a shell into an EC2 instance **without
  opening any inbound SSH port at all** — no bastion host, no key pairs to
  manage, and every session is logged to CloudWatch/S3 for audit — a huge
  security and compliance win over traditional SSH for banking environments.

```bash
# Store a non-sensitive config value
aws ssm put-parameter --name "/banking-app/prod/max-loan-amount" \
  --value "5000000" --type String

# Connect to an EC2 instance with ZERO open inbound ports
aws ssm start-session --target i-0abc123456789
```

```java
// Fetching a Parameter Store value in Spring Boot
import software.amazon.awssdk.services.ssm.SsmClient;
import software.amazon.awssdk.services.ssm.model.GetParameterRequest;

public class ConfigLoader {
    public static void main(String[] args) {
        try (SsmClient ssm = SsmClient.create()) {
            String value = ssm.getParameter(GetParameterRequest.builder()
                    .name("/banking-app/prod/max-loan-amount")
                    .build())
                    .parameter().value();
            System.out.println("Max loan amount: " + value);
        }
    }
}
```

## 17.6 CloudFormation — Infrastructure as Code

**What**: Describe your entire AWS infrastructure (VPCs, EC2, RDS, IAM
roles — everything) as a YAML/JSON template, and CloudFormation creates,
updates, or deletes all of it as one managed "Stack" — repeatable,
version-controlled, and reviewable (crucial for banking change-management
processes, where every infra change should be code-reviewed like
application code).

```yaml
# simplified-stack.yaml
Resources:
  LoanServiceBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: ananth-bank-loan-docs-2026
      VersioningConfiguration:
        Status: Enabled
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
```

```bash
aws cloudformation create-stack \
  --stack-name loan-service-infra \
  --template-body file://simplified-stack.yaml
aws cloudformation describe-stacks --stack-name loan-service-infra
```

> Also worth knowing: **AWS CDK** (Cloud Development Kit) lets you write
> this same infrastructure in actual Java/TypeScript/Python code instead of
> YAML, which many Java developers find more natural — CDK compiles down to
> CloudFormation templates under the hood. **Terraform** (by HashiCorp) is
> a popular third-party alternative that works across multiple clouds, not
> just AWS — commonly used in real companies too.

## 17.7 CloudTrail — Audit Logging

**What**: Records **every single API call** made in your AWS account — who
did what, from where, when. Not optional for banking — it's usually a
regulatory requirement to have a complete, immutable audit trail of every
infrastructure action.

```bash
aws cloudtrail create-trail --name banking-audit-trail \
  --s3-bucket-name banking-cloudtrail-logs \
  --is-multi-region-trail \
  --enable-log-file-validation   # cryptographically proves logs weren't tampered with

aws cloudtrail start-logging --name banking-audit-trail
```

**Interview Q**: *How would you investigate who deleted a critical S3
bucket?* — Query CloudTrail (directly, or via CloudWatch Logs Insights if
CloudTrail is streaming there) for `DeleteBucket` events, which record the
identity (user/role ARN), source IP, and timestamp of the action.

## 17.8 EventBridge — Event Bus

**What**: A more advanced, flexible successor to basic CloudWatch Events —
routes events from AWS services, your own applications, or SaaS partners to
multiple targets (Lambda, SQS, SNS, Step Functions) based on flexible
pattern-matching rules, without your services needing to know about each
other directly.

```bash
# Rule: whenever any EC2 instance changes state, notify SNS
aws events put-rule --name ec2-state-change-rule \
  --event-pattern '{"source": ["aws.ec2"], "detail-type": ["EC2 Instance State-change Notification"]}'

aws events put-targets --rule ec2-state-change-rule \
  --targets "Id"="1","Arn"="arn:aws:sns:ap-south-1:123456789012:ops-alerts"
```

```bash
# Scheduled rule: run the nightly reconciliation Lambda at 1 AM IST every day
aws events put-rule --name nightly-reconciliation \
  --schedule-expression "cron(30 19 * * ? *)"   # UTC time -- 19:30 UTC = 1:00 AM IST
aws events put-targets --rule nightly-reconciliation \
  --targets "Id"="1","Arn"="arn:aws:lambda:ap-south-1:123456789012:function:reconciliation-job"
```

## 17.9 Step Functions — Workflow Orchestration

**What**: Visually design and run multi-step workflows (state machines) that
coordinate multiple Lambda functions/services, with built-in error handling,
retries, and parallel execution — much cleaner than chaining Lambdas
together manually with custom glue code.

```json
// Simplified state machine: loan approval workflow
{
  "Comment": "Loan Application Approval Workflow",
  "StartAt": "CreditCheck",
  "States": {
    "CreditCheck": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-south-1:123456789012:function:credit-check",
      "Next": "IsCreditScoreOk",
      "Retry": [{ "ErrorEquals": ["States.TaskFailed"], "MaxAttempts": 3, "IntervalSeconds": 5 }]
    },
    "IsCreditScoreOk": {
      "Type": "Choice",
      "Choices": [
        { "Variable": "$.creditScore", "NumericGreaterThan": 700, "Next": "AutoApprove" }
      ],
      "Default": "ManualReview"
    },
    "AutoApprove": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-south-1:123456789012:function:approve-loan",
      "End": true
    },
    "ManualReview": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:ap-south-1:123456789012:function:queue-for-review",
      "End": true
    }
  }
}
```

This kind of workflow — "call service A, branch on the result, retry on
failure, call service B or C" — is extremely common in loan/underwriting
systems, and Step Functions gives you visibility (a visual execution
diagram) and reliability (built-in retries) for free.

## 17.10 WAF & Shield, ACM — Security Basics

- **WAF (Web Application Firewall)**: Sits in front of your ALB/CloudFront/
  API Gateway and blocks common attack patterns (SQL injection, XSS) and
  lets you set rate-based rules (e.g., block an IP making >1000 requests/5
  min — useful against credential-stuffing attacks on a login API).
- **Shield**: DDoS protection — Shield Standard is automatically included
  free for everyone; Shield Advanced (paid) adds more sophisticated
  protection and a response team for large-scale attacks.
- **ACM (Certificate Manager)**: Provision and auto-renew free SSL/TLS
  certificates for your domains, used directly by ALB/CloudFront for HTTPS
  — you never manually manage certificate expiry again.

```bash
aws acm request-certificate --domain-name api.mybank.com --validation-method DNS
```

## 17.11 Cost Explorer & Budgets

**What**: Tools to see where your money is going and get alerted before you
overspend — essential from Day 1 of learning AWS, and a standard practice
for any production account.

```bash
# Set a budget: alert if forecasted monthly spend will exceed $50
aws budgets create-budget --account-id 123456789012 --budget '{
  "BudgetName": "monthly-learning-budget",
  "BudgetLimit": {"Amount": "50", "Unit": "USD"},
  "TimeUnit": "MONTHLY",
  "BudgetType": "COST"
}' --notifications-with-subscribers '[{
  "Notification": {"NotificationType": "FORECASTED", "ComparisonOperator": "GREATER_THAN", "Threshold": 100},
  "Subscribers": [{"SubscriptionType": "EMAIL", "Address": "ananth@example.com"}]
}]'
```

**Do this on Day 1 of your AWS account, before anything else beyond IAM
setup.** It's the single easiest way to avoid a nasty surprise bill.

## 17.12 EKS vs ECS — Kubernetes on AWS (brief)

**EKS (Elastic Kubernetes Service)** is AWS's managed Kubernetes offering —
if your organization already standardizes on Kubernetes (common in larger
companies, multi-cloud strategies, or teams with existing K8s expertise),
EKS runs it for you (manages the control plane) while you still manage
workloads via standard `kubectl`/Helm.

**When to choose ECS vs EKS**: ECS is simpler, AWS-native, less operational
overhead — great default choice, especially for smaller teams or AWS-only
shops. EKS makes sense when you need Kubernetes-specific ecosystem tools,
multi-cloud portability, or your organization already has deep Kubernetes
expertise. This is a very common "which would you choose and why" interview
question — the honest answer is almost always "it depends on team
expertise and portability needs," which is exactly what a good answer
sounds like.

---

# 18. Capstone Project — A Banking Microservices Platform on AWS

This section wires everything above into one coherent architecture — the
kind of design you should be able to draw on a whiteboard in an interview,
and roughly the kind of system you'd build at a fintech.

## 18.1 The scenario

You're building the backend for a digital lending platform: customers apply
for loans, upload KYC documents, get a credit check, and receive
notifications at each step.

## 18.2 Architecture walkthrough (top to bottom)

```text
                                   [ Customer's Browser / Mobile App ]
                                                  |
                                        Route 53 (api.mybank.com)
                                                  |
                                    CloudFront (caches static frontend assets)
                                                  |
                                    WAF (blocks common attacks, rate limits)
                                                  |
                          API Gateway (HTTP API, Cognito authorizer validates customer JWT)
                                                  |
                                  VPC Link -> Internal Application Load Balancer
                                                  |
                         -----------------------------------------------------
                         |                        |                          |
                 ECS Fargate Service      ECS Fargate Service        ECS Fargate Service
                 "loan-service"           "kyc-service"              "notification-service"
                 (2-10 tasks,             (2-10 tasks,                (2-4 tasks,
                  auto-scaled)             auto-scaled)                 auto-scaled)
                         |                        |                          |
              +----------+----------+             |                          |
              |                     |              |                          |
        RDS PostgreSQL         SQS Queue      S3 Bucket (KYC docs,      SNS Topic
        Multi-AZ                "loan-        encrypted, versioned,     "customer-notifications"
        (loan/account            processing-   Block Public Access ON)  -> Email/SMS subscribers
         data, private            queue")           |
         subnet)                     |          KMS (encrypts the
              |                Lambda: async    bucket's contents)
        Secrets Manager        credit-check
        (DB credentials,       worker (calls
        auto-rotated)          external credit
                                bureau API)
                                     |
                          Step Functions: orchestrates
                          CreditCheck -> Decision -> Approve/ManualReview
                                     |
                       SNS Topic "loan-events" (fan-out)
                          |              |               |
                   SQS: notification  SQS: audit    SQS: accounting
                   -service queue     -service       -service queue
                                       queue

        Cross-cutting, applied everywhere:
        - IAM Roles (least privilege) scoped per service, Task Role != Task Execution Role
        - CloudWatch: metrics, alarms, dashboards, log groups per service
        - CloudTrail: every API call across the account audited
        - CodePipeline: Source(GitHub) -> Build(CodeBuild, tests+Docker) -> 
          Deploy(CodeDeploy Blue/Green to ECS)
        - ECR: stores every service's Docker images, immutable commit-hash tags
        - Systems Manager Session Manager: emergency shell access, no open SSH ports
        - Budgets: monthly spend alerts
```

## 18.3 Why each design choice was made (this is the part interviewers care about)

- **Route 53 + CloudFront + WAF in front of everything**: DNS, edge caching
  for static assets, and attack protection before any request even reaches
  your compute.
- **API Gateway with a VPC Link to a private ALB**: keeps ECS services
  completely private (no public IPs), while still exposing one managed,
  authenticated public API surface.
- **Separate ECS services per bounded context** (`loan-service`,
  `kyc-service`, `notification-service`): each independently deployable,
  independently scalable, with its own IAM Task Role — a bug in
  `notification-service` cannot touch loan data.
- **RDS Multi-AZ for core transactional data**: automatic failover, since
  loan/account data absolutely cannot be lost or become unavailable for long.
- **S3 for documents, not the database**: large binary files don't belong
  in a relational database; S3 with encryption + Block Public Access + a
  presigned-URL access pattern is both cheaper and more secure.
- **SQS between the API and the credit-check worker**: the external credit
  bureau API might be slow or briefly unavailable — decoupling means the
  customer's application submission still succeeds instantly, and the
  actual check happens asynchronously with automatic retry (DLQ safety
  net).
- **Step Functions for the approval workflow**: multiple conditional steps
  (credit check → auto-approve or manual review) benefit from visual
  workflow tracking and built-in retry logic rather than custom glue code
  spread across multiple Lambdas.
- **SNS fan-out for "loan approved" events**: notification, audit, and
  accounting concerns are all independently interested in this one event,
  without the loan service needing to know about any of them.
- **Secrets Manager with rotation for DB credentials**: no hardcoded
  passwords, automatic 30-day rotation with zero downtime.
- **CodePipeline with Blue/Green deploys**: every change goes through
  automated tests before reaching production, with instant rollback
  capability — critical for a system handling money.
- **CloudTrail everywhere**: full audit trail for regulatory compliance.

## 18.4 What you should practice explaining out loud

1. Trace one request end-to-end: "Customer clicks 'Apply for Loan' →
   ... → they get an SMS." Walk through every hop above.
2. Explain what happens if `loan-service`'s single AZ goes down mid-request
   (Multi-AZ RDS failover + ECS tasks spread across AZs + ALB health checks
   routing around the failure).
3. Explain how you'd add a new consumer to the "loan-events" topic (e.g., a
   new marketing-analytics service) without touching any existing service's
   code.
4. Explain your rollback plan if a bad deploy starts throwing 500 errors
   (Blue/Green: shift traffic back to blue; investigate green's CloudWatch
   logs/alarms).

---

# 19. AWS Well-Architected Framework — The 6 Pillars

AWS's official framework for evaluating any architecture — genuinely worth
memorizing the pillar names and one line per pillar, because interviewers
reference this constantly.

| Pillar | One-line meaning | Example from this guide |
|---|---|---|
| **Operational Excellence** | Run and monitor systems to deliver business value, and continually improve processes | CodePipeline automation, CloudWatch dashboards, CloudTrail audit logs |
| **Security** | Protect data, systems, and assets | IAM least privilege, encryption everywhere (KMS/S3/RDS), private subnets, WAF, Secrets Manager |
| **Reliability** | Recover from failures, meet demand dynamically | Multi-AZ RDS, ECS across multiple AZs, Auto Scaling, DLQs, health checks |
| **Performance Efficiency** | Use resources efficiently, adapt as needs evolve | CloudFront caching, right-sized instances/tasks, DynamoDB for high-throughput simple lookups |
| **Cost Optimization** | Avoid unnecessary costs | Fargate pay-per-task, S3 lifecycle rules to Glacier, Auto Scaling scales down off-peak, Budgets alerts |
| **Sustainability** | Minimize environmental impact of workloads | Right-sizing (less waste = less energy), choosing efficient instance types, scaling to zero where possible |

---

# 20. Interview Question Bank — Quick Fire (all services)

Use this as a final revision pass before an interview — cover the answer
and test yourself.

1. **What's the difference between IAM User and Role?** → User = permanent
   credentials for a person/app; Role = temporary, assumed credentials,
   preferred for applications.
2. **Public vs Private subnet?** → Public has a route to an Internet
   Gateway; Private does not (may use a NAT Gateway for outbound-only).
3. **Security Group vs NACL?** → SG = stateful, instance-level, Allow only;
   NACL = stateless, subnet-level, Allow + Deny, ordered rules.
4. **On-Demand vs Reserved vs Spot EC2?** → Flexibility vs long-term
   discount vs cheapest-but-interruptible.
5. **S3 storage classes, hottest to coldest?** → Standard → Standard-IA →
   Glacier → Glacier Deep Archive.
6. **How do you securely share one private S3 file with a customer?** →
   Pre-signed URL with a short expiry.
7. **RDS Multi-AZ vs Read Replica?** → HA/automatic failover vs read-scaling.
8. **What can't CloudWatch see on EC2 by default?** → Memory and disk usage
   — need the CloudWatch Agent.
9. **SQS vs SNS in one line?** → Queue (pull, one consumer) vs Topic (push,
   fan-out to many).
10. **Standard vs FIFO SQS?** → Best-effort order/at-least-once vs strict
    order/exactly-once, at lower throughput.
11. **What's a Lambda cold start?** → Latency from initializing a fresh
    execution environment; mitigate with Provisioned Concurrency.
12. **HTTP API vs REST API (API Gateway)?** → HTTP API is simpler/cheaper;
    REST API has more advanced features (usage plans, transformations).
13. **Why use Secrets Manager over a hardcoded password?** → Avoids leaking
    credentials in code/config, supports automatic zero-downtime rotation.
14. **Multi-stage Docker build — why?** → Smaller, more secure final image;
    build tools discarded after compiling.
15. **ECS Task Role vs Task Execution Role?** → App permissions vs ECS
    agent's permissions (pull image, write logs).
16. **Fargate vs EC2 launch type for ECS?** → Serverless/simpler vs more
    control/potentially cheaper at scale.
17. **ALB vs NLB?** → Layer 7 HTTP-aware routing vs Layer 4 raw
    TCP/UDP performance.
18. **Target Tracking scaling — what is it?** → Declare a target metric
    value; AWS calculates the scaling actions automatically.
19. **Blue/Green vs Canary deployment?** → Full cutover with instant
    rollback option vs gradual, monitored traffic shift.
20. **DynamoDB vs RDS — when each?** → DynamoDB for simple, massive-scale
    key-based access patterns; RDS for complex relational/transactional
    data with joins.
21. **What does KMS Envelope Encryption mean?** → A data key encrypts your
    actual data locally; KMS only encrypts that small data key, not your
    whole payload — far more efficient.
22. **Alias record vs CNAME in Route 53?** → Alias works at the zone apex
    and is free to query; CNAME cannot be used at the apex per DNS spec.
23. **Why use CloudTrail?** → Full audit trail of every API call — who did
    what, when, from where; often a compliance requirement.
24. **When would you use Step Functions instead of chaining Lambdas
    manually?** → Multi-step workflows needing visual tracking, built-in
    retries/error handling, and conditional branching.
25. **First thing you do on a brand-new AWS account?** → Secure the root
    user (MFA, no daily use), create an IAM admin user, set up a Budget
    alert.

---

# 21. Glossary

| Term | Meaning |
|---|---|
| ARN | Amazon Resource Name — unique identifier for any AWS resource |
| AZ | Availability Zone — an isolated data center within a Region |
| CIDR | Notation for an IP address range, e.g. 10.0.0.0/16 |
| CI/CD | Continuous Integration / Continuous Deployment |
| Console | The AWS web UI |
| DLQ | Dead Letter Queue |
| HA | High Availability |
| IaC | Infrastructure as Code |
| IAM | Identity and Access Management |
| JWT | JSON Web Token — commonly used for API authentication |
| MFA | Multi-Factor Authentication |
| Region | A geographic area containing multiple AZs |
| SDK | Software Development Kit — a library to call AWS APIs from code |
| TTL | Time To Live — how long something is cached/valid |
| VPC | Virtual Private Cloud |

---

# 22. Further Resources

- **AWS Free Tier**: https://aws.amazon.com/free — practice everything in
  this guide at (mostly) zero cost.
- **AWS Documentation**: https://docs.aws.amazon.com — the primary source
  of truth; always check here for anything that may have changed.
- **AWS Skill Builder**: free official training courses, useful for
  certification prep (Solutions Architect Associate is a strong next step
  after working through this guide).
- **AWS Well-Architected Tool**: available free in your AWS Console — run
  it against a real project to get a structured review against the 6
  pillars in Section 19.
- **AWS SDK for Java v2 documentation**: https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/home.html

## Suggested next steps for you specifically

Given your background in Java/Spring Boot in the banking domain, a strong
learning path from here:

1. Rebuild the Capstone architecture (Section 18) for real in your own AWS
   Free Tier account — even a scaled-down version (skip WAF/CloudFront
   initially, add them once the core works).
2. Get comfortable with CloudFormation or AWS CDK (Java!) so you can
   recreate the whole thing as code, not console clicks.
3. Study for the **AWS Certified Solutions Architect – Associate**
   certification — it directly validates the exact breadth of knowledge
   this guide covers, and is well-recognized in fintech hiring.
4. Once comfortable, explore **AWS Certified Developer – Associate**, which
   goes deeper into the Lambda/API Gateway/DynamoDB/CI-CD side specifically.

---

*End of guide. Go build something — the fastest way this knowledge
actually sticks is by breaking a Free Tier account and fixing it yourself.*
