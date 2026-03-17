# AWS Certified Cloud Practitioner (CLF-C02) — Complete Exam Notes

> **Exam Code:** CLF-C02 | **Questions:** 65 (50 scored + 15 unscored) | **Duration:** 90 minutes | **Passing Score:** 700/1000 | **Format:** Multiple choice & multiple response

---

## Exam Domain Weightage at a Glance

| # | Domain | Weight | ~Questions |
|---|--------|--------|------------|
| 3 | Cloud Technology and Services | **34%** | ~22 |
| 2 | Security and Compliance | **30%** | ~20 |
| 1 | Cloud Concepts | **24%** | ~16 |
| 4 | Billing, Pricing, and Support | **12%** | ~8 |

> **Study Strategy:** Spend proportional time — Technology & Security together make up **64%** of the exam. Nail these two and you're most of the way there.

---

# DOMAIN 3 — Cloud Technology and Services (34%)

This is the **biggest domain**. You need to know which AWS service solves which problem. You don't need to configure them — just identify and describe them.

---

## 3.1 Compute Services

### Amazon EC2 (Elastic Compute Cloud)
Virtual servers in the cloud. You choose instance type, OS, and storage. Think of it as renting a computer by the hour.

**Instance Families — Memory Trick: "CRAM GPS TH"**

| Family | Letter | Optimized For | Example Use |
|--------|--------|---------------|-------------|
| Compute | C | CPU-intensive | Batch processing, gaming servers |
| General Purpose | T, M | Balanced | Web servers, small DBs |
| Memory | R, X | RAM-intensive | In-memory caches, real-time analytics |
| Accelerated | P, G | GPU/hardware | ML training, video encoding |
| Storage | I, D, H | High disk I/O | Data warehousing, distributed file systems |

> **Memory Trick:** "**T**iny **M**achines are **G**eneral purpose" — T and M families are general purpose.

### AWS Lambda
Run code **without provisioning servers** (serverless). You pay only for the compute time consumed — zero charge when code is not running. Triggered by events (S3 upload, API call, DynamoDB change).

> **Memory Trick:** "**Lambda = Lazy servers** — they only work when triggered and bill you only for work done."

### Amazon ECS / EKS / Fargate (Containers)
- **ECS** (Elastic Container Service): Run Docker containers on AWS-managed infrastructure.
- **EKS** (Elastic Kubernetes Service): Managed Kubernetes — use this if the question mentions "Kubernetes" or "K8s".
- **Fargate**: Serverless compute engine for containers — no need to manage underlying servers.

> **Memory Trick:** "**ECS = Easy Containers**, **EKS = Elastic K8s**, **Fargate = Forget servers for containers**."

### AWS Elastic Beanstalk
Upload your code, and Beanstalk automatically handles deployment, capacity, load balancing, and health monitoring. PaaS (Platform as a Service).

> **Memory Trick:** "**Beanstalk = Just plant your code**, AWS grows the infrastructure."

### Amazon Lightsail
Simple virtual private servers — fixed monthly pricing. Best for small apps, websites, and dev environments. Think of it as "AWS made simple."

### AWS Outposts
AWS infrastructure deployed **on your premises** (your own data center). Hybrid cloud solution.

> **Memory Trick:** "**Outposts = AWS moves Out to your Post (location)**."

### AWS Batch
Run batch computing workloads at scale. AWS manages compute resources for you.

---

## 3.2 Storage Services

### Amazon S3 (Simple Storage Service)
Object storage with **11 nines (99.999999999%) durability**. Unlimited storage. Stores files as objects in buckets.

**S3 Storage Classes — Ordered from Most to Least Expensive:**

| Class | Use Case | Access Frequency |
|-------|----------|------------------|
| S3 Standard | Frequently accessed data | Frequent |
| S3 Intelligent-Tiering | Unknown/changing access patterns | Auto-tiers |
| S3 Standard-IA | Infrequent access, rapid retrieval | Infrequent |
| S3 One Zone-IA | Infrequent, non-critical, single AZ | Infrequent |
| S3 Glacier Instant Retrieval | Archive with millisecond access | Rare |
| S3 Glacier Flexible Retrieval | Archive, minutes–hours retrieval | Rare |
| S3 Glacier Deep Archive | Cheapest, 12–48 hour retrieval | Very rare |

> **Memory Trick:** "**S**tart **I**ntelligently, go **I**nfrequent, then **G**lacier for ice-cold data." Think: Hot → Warm → Cold → Frozen.

### Amazon EBS (Elastic Block Store)
Persistent block storage for EC2 instances — like a virtual hard drive. Attached to **one EC2 instance at a time** within a single AZ.

### Amazon EFS (Elastic File System)
Managed file storage — can be shared across **multiple EC2 instances** simultaneously. Think: network drive.

> **Memory Trick:** "**EBS = one-to-one (1 disk, 1 server)**. **EFS = one-to-many (shared folder)**."

### Amazon FSx
Managed file systems for Windows (FSx for Windows File Server) and high-performance computing (FSx for Lustre).

### AWS Storage Gateway
Hybrid storage service connecting on-premises environments to AWS cloud storage.

### AWS Snow Family (Data Transfer)
Physical devices for **offline data migration** to AWS:

| Device | Capacity | Use Case |
|--------|----------|----------|
| **Snowcone** | 8–14 TB | Edge computing, small transfers |
| **Snowball Edge** | 80 TB | Large data transfers |
| **Snowmobile** | 100 PB | Massive data center migrations (a literal truck!) |

> **Memory Trick:** "**Cone = small Cone ice cream**. **Ball = bigger snowball**. **Mobile = a truck full of data.**"

---

## 3.3 Database Services

### Amazon RDS (Relational Database Service)
Managed relational databases. Supports: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Amazon Aurora.

### Amazon Aurora
AWS-built relational DB, **5x faster than MySQL** and **3x faster than PostgreSQL**. Fully compatible with both. High availability built-in.

> **Memory Trick:** "**Aurora = the northern lights — fast and brilliant**."

### Amazon DynamoDB
Fully managed **NoSQL** key-value and document database. Single-digit millisecond latency at any scale. Serverless.

> **Memory Trick:** "**Dynamo = Dynamic NoSQL**. Fast, flexible, serverless."

### Amazon ElastiCache
In-memory caching service. Supports **Redis** and **Memcached**. Reduces load on databases.

> **Memory Trick:** "**ElastiCache = Elastic + Cache**. Speeds up apps by caching frequent queries in RAM."

### Amazon Redshift
Managed **data warehouse** for analytics and big data. Column-based storage for fast queries over petabytes.

> **Memory Trick:** "**Redshift = shift through Red-hot amounts of data** (analytics)."

### Amazon Neptune
Managed **graph database**. Use for social networks, recommendation engines, fraud detection.

### Amazon DocumentDB
Managed document database (MongoDB-compatible).

### Amazon QLDB (Quantum Ledger Database)
Fully managed **ledger database** — immutable, cryptographically verifiable transaction log. Think: blockchain-like but centralized.

### Amazon Keyspaces
Managed Apache Cassandra-compatible database.

### Amazon Timestream
Managed **time-series database**. Use for IoT, DevOps monitoring.

**Quick Database Cheat Sheet:**

| If the question says... | The answer is... |
|-------------------------|------------------|
| Relational / SQL | RDS or Aurora |
| NoSQL / key-value | DynamoDB |
| Caching / in-memory | ElastiCache |
| Data warehouse / analytics | Redshift |
| Graph / relationships | Neptune |
| Ledger / immutable log | QLDB |
| Document / MongoDB | DocumentDB |
| Time-series / IoT data | Timestream |

---

## 3.4 Networking Services

### Amazon VPC (Virtual Private Cloud)
Your own logically isolated network within AWS. You define IP ranges, subnets, route tables, and gateways.

- **Public Subnet:** Resources accessible from the internet.
- **Private Subnet:** Resources NOT directly accessible from the internet.
- **Internet Gateway (IGW):** Connects VPC to the internet.
- **NAT Gateway:** Allows private subnet resources to access the internet **outbound only**.
- **Security Groups:** Virtual firewalls at the **instance level** — **stateful** (return traffic automatically allowed).
- **NACLs (Network ACLs):** Firewalls at the **subnet level** — **stateless** (must define both inbound and outbound rules).

> **Memory Trick:** "**Security Groups = bouncers at the door (instance level, stateful)**. **NACLs = border patrol at the neighborhood (subnet level, stateless)**."

### Amazon Route 53
AWS DNS (Domain Name System) service. Routes end users to applications. Also does domain registration and health checks.

> **Memory Trick:** "**Route 53 — Port 53 is DNS**, so Route 53 = DNS routing."

### Amazon CloudFront
Global **CDN (Content Delivery Network)**. Caches content at **edge locations** near users for low latency. Works with S3, EC2, and ALB.

> **Memory Trick:** "**CloudFront = Front door of the cloud** — serves content from the closest edge."

### Elastic Load Balancing (ELB)
Distributes incoming traffic across multiple targets (EC2, containers, IPs).

- **ALB (Application LB):** Layer 7 — HTTP/HTTPS. Best for web apps.
- **NLB (Network LB):** Layer 4 — TCP/UDP. Best for extreme performance.
- **GLB (Gateway LB):** Layer 3 — for third-party virtual appliances.

### AWS Direct Connect
Dedicated **private physical connection** from your data center to AWS. More consistent and lower latency than internet VPN.

> **Memory Trick:** "**Direct Connect = a Direct private highway to AWS** (no internet traffic)."

### AWS VPN
Encrypted connection from your network to AWS **over the public internet**.

> **Memory Trick:** "**VPN = Virtual Private tunnel over the internet**. **Direct Connect = Physical private cable**."

### AWS Transit Gateway
Hub that connects multiple VPCs and on-premises networks. Simplifies complex networking.

### AWS Global Accelerator
Improves availability and performance of your apps by routing traffic through AWS's global network to the closest healthy endpoint.

---

## 3.5 Application Integration & Messaging

### Amazon SQS (Simple Queue Service)
Fully managed **message queue**. Decouples microservices. Messages wait in a queue until consumed.

### Amazon SNS (Simple Notification Service)
Pub/sub messaging. Sends notifications to multiple subscribers (email, SMS, Lambda, HTTP).

> **Memory Trick:** "**SQS = a queue line (one by one)**. **SNS = a megaphone (one-to-many notifications)**."

### Amazon EventBridge
Serverless **event bus** — connects applications using events from AWS services, SaaS apps, or custom sources.

### AWS Step Functions
Orchestrate **workflows** by coordinating multiple AWS services into serverless workflows using visual state machines.

### Amazon API Gateway
Create, publish, and manage **REST and WebSocket APIs** at any scale.

---

## 3.6 Analytics Services

| Service | Purpose |
|---------|---------|
| **Amazon Athena** | Query S3 data using **SQL** (serverless) |
| **Amazon Kinesis** | Real-time **streaming data** processing |
| **AWS Glue** | Serverless **ETL** (extract, transform, load) |
| **Amazon QuickSight** | Business intelligence / **dashboards & visualizations** |
| **Amazon EMR** | Managed Hadoop/Spark for big data processing |
| **AWS Lake Formation** | Build **data lakes** easily on S3 |

> **Memory Trick:** "**Athena is wise** — she queries your S3 data smartly. **Kinesis = Kinetic = movement = streaming**."

---

## 3.7 Machine Learning & AI Services

| Service | What It Does |
|---------|-------------|
| **Amazon SageMaker** | Build, train, deploy ML models |
| **Amazon Rekognition** | Image and video analysis (faces, objects) |
| **Amazon Comprehend** | NLP — sentiment analysis, key phrases |
| **Amazon Polly** | Text to speech |
| **Amazon Transcribe** | Speech to text |
| **Amazon Translate** | Language translation |
| **Amazon Lex** | Chatbots (powers Alexa) |
| **Amazon Textract** | Extract text/data from scanned documents |
| **Amazon Kendra** | Intelligent enterprise search |
| **Amazon Personalize** | Real-time personalized recommendations |

> **Memory Trick:** "**Lex = Alexa (chatbot)**. **Polly wants a cracker = Polly speaks (text-to-speech)**. **Transcribe = court reporter (speech-to-text)**."

---

## 3.8 Developer & Management Tools

| Service | Purpose |
|---------|---------|
| **AWS CloudFormation** | **Infrastructure as Code (IaC)** — define resources in JSON/YAML templates |
| **AWS CDK** | Define cloud infra using programming languages (compiles to CloudFormation) |
| **AWS CloudWatch** | **Monitoring** — metrics, logs, alarms |
| **AWS CloudTrail** | **Audit log** — who did what, when (API call logging) |
| **AWS Config** | Track **resource configurations** and compliance over time |
| **AWS Systems Manager** | Manage and patch EC2 instances at scale |
| **AWS Trusted Advisor** | Best practice **recommendations** (cost, performance, security, fault tolerance) |
| **AWS X-Ray** | Debug and analyze **distributed applications** |

> **Memory Trick:** "**CloudWatch = watching performance**. **CloudTrail = trail of breadcrumbs (audit who did what)**. **Config = configuration cop (are resources configured correctly?)**."

---

## 3.9 Migration & Transfer Services

| Service | Purpose |
|---------|---------|
| **AWS Migration Hub** | Central place to track migrations |
| **AWS DMS (Database Migration Service)** | Migrate databases to AWS |
| **AWS Application Migration Service (MGN)** | Lift-and-shift server migrations |
| **AWS Transfer Family** | SFTP/FTPS/FTP transfers to S3 or EFS |

---

# DOMAIN 2 — Security and Compliance (30%)

The **second-largest domain** — AWS has heavily increased its weight in CLF-C02. Security is woven into nearly every AWS service.

---

## 2.1 The Shared Responsibility Model (EXAM FAVORITE!)

This is the **#1 most tested concept** on the exam.

```
┌─────────────────────────────────────────────────────┐
│              CUSTOMER RESPONSIBILITY                │
│          "Security IN the Cloud"                    │
│                                                     │
│  • Customer data                                    │
│  • Platform, applications, IAM                      │
│  • Operating system, network & firewall config      │
│  • Client-side data encryption                      │
│  • Server-side encryption                           │
│  • Network traffic protection                       │
├─────────────────────────────────────────────────────┤
│              AWS RESPONSIBILITY                     │
│          "Security OF the Cloud"                    │
│                                                     │
│  • Hardware / Global Infrastructure                 │
│  • Regions, AZs, Edge Locations                     │
│  • Compute, Storage, Database, Networking           │
│  • Managed services software/patching               │
└─────────────────────────────────────────────────────┘
```

> **Memory Trick:** "**AWS secures the building (OF). You secure what's inside your apartment (IN).**"

**Key exam distinctions:**
- **Customer patches the Guest OS** on EC2. AWS patches the host/hypervisor.
- For **managed services** (Lambda, RDS, DynamoDB), AWS takes on more responsibility (OS patching, etc.).
- **Customer is ALWAYS responsible** for their data, IAM, encryption choices, and Security Group configs.

---

## 2.2 AWS Identity and Access Management (IAM)

IAM controls **who** can access **what** in your AWS account.

**Core Components:**

| Component | What It Is |
|-----------|-----------|
| **Users** | Individual people/apps with long-term credentials |
| **Groups** | Collection of users — attach policies to groups, not individual users |
| **Roles** | Temporary credentials for services or cross-account access |
| **Policies** | JSON documents defining allow/deny permissions |

**Critical IAM Principles:**
- **Root Account:** Has unlimited access. Secure with MFA and do NOT use for daily tasks.
- **Least Privilege:** Give only the minimum permissions needed.
- **MFA (Multi-Factor Authentication):** Always enable on root and privileged users.
- **IAM is global** — not tied to a specific Region.

> **Memory Trick:** "**Users = people. Groups = teams. Roles = hats (temporary, for services). Policies = rulebooks.**"

### AWS IAM Identity Center (formerly AWS SSO)
Single sign-on for **multiple AWS accounts** and business applications. Centralized access management.

### AWS Organizations
Manage **multiple AWS accounts** centrally. Use **Service Control Policies (SCPs)** to set permission guardrails across accounts.

> **Memory Trick:** "**Organizations = the parent company. SCPs = company-wide rules everyone must follow.**"

---

## 2.3 Security Services

### AWS WAF (Web Application Firewall)
Protects web apps from common web exploits (SQL injection, XSS) at **Layer 7**. Works with CloudFront, ALB, and API Gateway.

### AWS Shield
DDoS protection:
- **Shield Standard:** Free, automatic, protects all AWS customers.
- **Shield Advanced:** Paid, enhanced DDoS protection with 24/7 DDoS response team.

> **Memory Trick:** "**Shield = shield from DDoS attacks**. Standard is free. Advanced costs money but gives you a human team."

### Amazon GuardDuty
**Threat detection** service — uses ML to analyze CloudTrail logs, VPC Flow Logs, and DNS logs. Detects suspicious activity.

> **Memory Trick:** "**GuardDuty = security guard on duty**, watching for threats."

### Amazon Inspector
**Automated vulnerability assessment** for EC2 instances and container images. Checks for software vulnerabilities and unintended network exposure.

### AWS Macie
Uses ML to discover and protect **sensitive data** (like PII) in **S3 buckets**.

> **Memory Trick:** "**Macie = My data is ACE (protected)**. Specifically guards data in S3."

### AWS KMS (Key Management Service)
Create and manage **encryption keys**. Used to encrypt data across AWS services.

### AWS CloudHSM
Dedicated **hardware security module** for key management. You control the keys entirely (vs KMS where AWS manages hardware).

> **Memory Trick:** "**KMS = AWS manages keys for you. CloudHSM = You hold your own hardware key vault.**"

### AWS Secrets Manager
Rotate, manage, and retrieve **secrets** (database passwords, API keys). Automatic rotation capability.

### AWS Certificate Manager (ACM)
Provision, manage, and deploy **SSL/TLS certificates** for free (for AWS services).

### Amazon Cognito
User **sign-up, sign-in, and access control** for web and mobile apps. Supports social identity providers (Google, Facebook).

### AWS Security Hub
Central security dashboard — aggregates findings from GuardDuty, Inspector, Macie, and more.

---

## 2.4 Compliance and Governance

### AWS Artifact
Self-service portal for **compliance reports** and agreements (SOC, PCI, ISO). No cost.

> **Memory Trick:** "**Artifact = archive of compliance Artifacts (documents)**."

### AWS Config
Continuously evaluates resource configurations against **compliance rules**.

### AWS Control Tower
Set up and govern a **multi-account environment** with best practices (landing zone).

### AWS Audit Manager
Continuously audit AWS usage for **risk and compliance** assessment.

---

## 2.5 Key Security Concepts for the Exam

**Encryption at Rest vs. In Transit:**
- **At Rest:** Data stored on disk — use KMS, S3 encryption, EBS encryption.
- **In Transit:** Data moving over the network — use TLS/SSL, HTTPS.

**Principle of Least Privilege:** Always grant minimum permissions.

**AWS Compliance Programs:** AWS holds certifications like SOC 1/2/3, PCI DSS, HIPAA, ISO 27001, FedRAMP, GDPR compliance tools.

**Security Logging Stack:**
- **CloudTrail** = Who made API calls (audit trail)
- **CloudWatch** = What's happening now (monitoring/alarms)
- **VPC Flow Logs** = Network traffic in your VPC
- **GuardDuty** = What looks suspicious (threat detection)

---

# DOMAIN 1 — Cloud Concepts (24%)

---

## 1.1 Six Advantages of Cloud Computing

AWS defines these six benefits — memorize them exactly:

1. **Trade capital expense for variable expense** — pay only for what you use.
2. **Benefit from massive economies of scale** — AWS purchases at huge scale, passing savings to you.
3. **Stop guessing capacity** — scale up/down as needed.
4. **Increase speed and agility** — spin up resources in minutes vs. weeks.
5. **Stop spending money running data centers** — focus on your business, not infrastructure.
6. **Go global in minutes** — deploy across multiple Regions worldwide.

> **Memory Trick (first letters): "T-B-S-I-S-G"** — "**T**he **B**est **S**olution **I**s **S**uper **G**lobal"

---

## 1.2 Cloud Computing Models

| Model | What You Manage | What Provider Manages | Example |
|-------|----------------|----------------------|---------|
| **IaaS** | OS, Apps, Data | Servers, Storage, Networking | EC2, VPC |
| **PaaS** | Apps, Data | Everything else | Elastic Beanstalk, Heroku |
| **SaaS** | Nothing (just use it) | Everything | Gmail, Salesforce, Office 365 |

> **Memory Trick:** "**Pizza analogy** — IaaS = you cook at home with ingredients. PaaS = take-and-bake pizza. SaaS = dine at the restaurant."

---

## 1.3 Cloud Deployment Models

| Model | Description |
|-------|------------|
| **Public Cloud** | Everything runs on AWS (fully cloud) |
| **Private Cloud (On-Premises)** | Everything runs in your own data center |
| **Hybrid Cloud** | Mix of on-premises and AWS cloud |

---

## 1.4 AWS Well-Architected Framework (6 Pillars)

Memorize all six pillars — this is heavily tested.

| Pillar | Focus |
|--------|-------|
| **Operational Excellence** | Run and monitor systems, improve processes |
| **Security** | Protect data, systems, and assets |
| **Reliability** | Recover from failures, meet demand |
| **Performance Efficiency** | Use resources efficiently |
| **Cost Optimization** | Avoid unnecessary costs |
| **Sustainability** | Minimize environmental impact |

> **Memory Trick: "OS-RPeCS"** — think "**O**perations **S**ecured **R**eliably with **P**erformant **C**ost-effective **S**ustainability"

Or remember: **"OSRPCS" = "Oh Sure, Really Perfect Cloud System"**

---

## 1.5 AWS Cloud Adoption Framework (AWS CAF)

New focus area in CLF-C02. Six perspectives grouped into two categories:

**Business Capabilities:**
- **Business** — Align IT with business outcomes
- **People** — HR, training, organizational change
- **Governance** — Risk management, compliance

**Technical Capabilities:**
- **Platform** — Architecture, build infrastructure
- **Security** — Controls, compliance
- **Operations** — Run and monitor

> **Memory Trick:** "**B-P-G / P-S-O** = **B**usiness **P**eople **G**overn / **P**latforms **S**ecurely **O**perated"

---

## 1.6 AWS Global Infrastructure

### Regions
Geographic areas containing multiple Availability Zones. Choose a Region based on: compliance, latency, service availability, and cost.

### Availability Zones (AZs)
One or more discrete data centers within a Region. Each AZ has independent power, cooling, and networking. **Deploy across multiple AZs for high availability.**

### Edge Locations
Locations for **CloudFront CDN** caching. There are far more edge locations (~400+) than Regions.

### Local Zones
Extend AWS infrastructure closer to end users for ultra-low latency.

### Wavelength Zones
Bring AWS to the edge of **5G networks**.

> **Memory Trick:** "**Regions > AZs > Edge Locations** — think Country > City > Neighborhood delivery points."

---

## 1.7 High Availability, Elasticity & Scalability

| Concept | Meaning | AWS Feature |
|---------|---------|-------------|
| **High Availability** | System stays running even if parts fail | Multi-AZ deployment |
| **Elasticity** | Automatically scale OUT and IN based on demand | Auto Scaling |
| **Scalability** | Ability to handle growth | Vertical (bigger instance) or Horizontal (more instances) |
| **Fault Tolerance** | System operates even during failures | Redundancy across AZs/Regions |
| **Disaster Recovery** | Recover from catastrophic events | Multi-Region, backups |

> **Memory Trick:** "**Elasticity = rubber band (stretches and shrinks). Scalability = growing taller or wider.**"

---

# DOMAIN 4 — Billing, Pricing, and Support (12%)

Smallest domain but easy points — don't skip it.

---

## 4.1 AWS Pricing Models

### EC2 Pricing Options

| Option | Description | Best For | Discount |
|--------|------------|----------|----------|
| **On-Demand** | Pay per hour/second, no commitment | Short-term, unpredictable workloads | None |
| **Reserved Instances** | 1 or 3-year commitment | Steady-state, predictable usage | Up to **72%** off |
| **Savings Plans** | Flexible commitment ($/hr) for 1-3 years | Flexible workloads across instance types | Up to **72%** off |
| **Spot Instances** | Bid on unused capacity | Fault-tolerant, flexible, batch jobs | Up to **90%** off |
| **Dedicated Hosts** | Physical server dedicated to you | Licensing, compliance requirements | Varies |

> **Memory Trick: "On-Demand = taxi. Reserved = annual metro pass. Spot = standby flight (cheap but can be bumped). Dedicated = private chauffeur."**

### Free Tier
Three types:
- **Always Free:** Lambda (1M requests/month), DynamoDB (25 GB), etc.
- **12 Months Free:** EC2 (t2.micro/t3.micro), S3 (5 GB), RDS (db.t2.micro).
- **Trials:** Short-term free trials for specific services.

### What's Always Free in AWS
- IAM — always free
- VPC — the basic VPC is free
- Auto Scaling — the service itself is free (you pay for resources)
- CloudFormation — free (pay for resources deployed)
- Elastic Beanstalk — free (pay for underlying resources)

---

## 4.2 Cost Management Tools

| Tool | Purpose |
|------|---------|
| **AWS Cost Explorer** | **Visualize** and analyze spending over time with graphs |
| **AWS Budgets** | Set **custom budgets** and get alerts when thresholds are crossed |
| **AWS Cost and Usage Report (CUR)** | Most **detailed** billing data (downloadable CSV) |
| **AWS Pricing Calculator** | **Estimate** costs for architecture before building |
| **AWS Billing Dashboard** | High-level view of current month charges |
| **Cost Allocation Tags** | Tag resources to track costs by project/team/environment |

> **Memory Trick:** "**Explorer = look at past spending. Budgets = set future limits. Calculator = plan before you build. CUR = the detailed receipt.**"

---

## 4.3 AWS Support Plans

| Feature | Basic (Free) | Developer | Business | Enterprise On-Ramp | Enterprise |
|---------|-------------|-----------|----------|-------------------|------------|
| **Price** | Free | $29+/mo | $100+/mo | $5,500/mo | $15,000+/mo |
| **Trusted Advisor** | 7 core checks | 7 core checks | **Full checks** | **Full checks** | **Full checks** |
| **Technical Support** | None | Business hours email | 24/7 phone, email, chat | 24/7 phone, email, chat | 24/7 phone, email, chat |
| **Response Time (Critical)** | — | — | **1 hour** | **30 minutes** | **15 minutes** |
| **TAM** | No | No | No | Pool of TAMs | **Dedicated TAM** |
| **Concierge** | No | No | No | No | **Yes** |

> **Memory Trick:** "**B-D-B-E-E** (Basic, Dev, Business, Enterprise On-Ramp, Enterprise). Only Business and above get FULL Trusted Advisor and 24/7 support. Only Enterprise gets a dedicated TAM.**"

> **TAM = Technical Account Manager** — your personal AWS advisor.
> **Concierge = Billing and account expert** (Enterprise only).

---

## 4.4 Consolidated Billing (AWS Organizations)

- Combine billing across **multiple AWS accounts** into one bill.
- **Volume discounts** — aggregated usage across all accounts can hit cheaper pricing tiers.
- One **payer account** manages billing for all linked accounts.

---

## 4.5 Other Billing Concepts

| Concept | Detail |
|---------|--------|
| **Data Transfer IN** | Always **free** |
| **Data Transfer OUT** | Charged (between Regions, to internet) |
| **S3 Pricing** | Storage + requests + data transfer out |
| **EC2 Pricing** | Instance hours + EBS + data transfer out |
| **Pay-as-you-go** | Core AWS pricing philosophy |

> **Memory Trick:** "**Data IN = free invitation. Data OUT = exit fee.**"

---

# EXAM DAY CHEAT SHEET — Quick-Fire Mnemonics

| If the question mentions... | Think... |
|-----------------------------|----------|
| "Serverless" | Lambda, Fargate, DynamoDB, S3, Aurora Serverless |
| "Shared Responsibility" | AWS = OF the cloud. Customer = IN the cloud |
| "Audit / who did what" | CloudTrail |
| "Monitor / alarms / metrics" | CloudWatch |
| "DDoS protection" | Shield (Standard = free, Advanced = paid) |
| "Web app firewall" | WAF |
| "Threat detection" | GuardDuty |
| "Sensitive data in S3" | Macie |
| "Compliance documents" | Artifact |
| "Encryption keys" | KMS (managed) or CloudHSM (dedicated hardware) |
| "Infrastructure as Code" | CloudFormation |
| "Cost recommendations" | Trusted Advisor or Cost Explorer |
| "Domain names / DNS" | Route 53 |
| "Content delivery / CDN" | CloudFront |
| "Dedicated private connection" | Direct Connect |
| "Migrate databases" | DMS |
| "Multi-account management" | AWS Organizations |
| "Kubernetes" | EKS |
| "Docker containers" | ECS or Fargate |
| "Message queue" | SQS |
| "Notifications / pub-sub" | SNS |
| "Data warehouse" | Redshift |
| "Real-time streaming" | Kinesis |
| "Query S3 with SQL" | Athena |
| "Chatbot" | Lex |
| "Text to speech" | Polly |
| "Speech to text" | Transcribe |
| "Image/video recognition" | Rekognition |
| "Hybrid / on-premises AWS" | Outposts |
| "Physical data transfer" | Snow Family |
| "BI dashboards" | QuickSight |

---

# EXAM STRATEGY TIPS

1. **Eliminate obviously wrong answers first.** Most questions have 1-2 clearly incorrect options.
2. **Watch for absolute words** like "always," "never," "only" — these are often traps.
3. **"Most cost-effective"** = usually Spot Instances, S3 Glacier, or reserved pricing.
4. **"Least operational overhead"** = usually a managed/serverless service.
5. **"Best practice"** = usually involves Multi-AZ, least privilege, encryption, or automation.
6. **When unsure between two AWS services**, pick the one that is more managed/serverless.
7. **Flag difficult questions** and come back — don't waste time. You have ~83 seconds per question.
8. **Read the question twice** — AWS loves to test whether you read "customer" vs "AWS" responsibility.
9. **Never leave a blank** — there's no penalty for guessing.
10. **If a question mentions a framework**, think Well-Architected or CAF.

---

*Good luck on your CLF-C02 exam! Remember: understand the concepts, don't just memorize services. The exam tests your judgment, not your ability to recite documentation.*
