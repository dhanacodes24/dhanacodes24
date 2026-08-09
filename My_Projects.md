<div align="center">

# ☁️ AWS Automation Projects Portfolio

### Serverless automation, cost optimization & security compliance using AWS Lambda, Boto3 & EventBridge

![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS%20Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![Boto3](https://img.shields.io/badge/Boto3-569A31?style=for-the-badge&logo=amazon-aws&logoColor=white)
![EventBridge](https://img.shields.io/badge/EventBridge-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)

![Profile Views](https://komarev.com/ghpvc/?username=your-username&color=blue&style=flat-square)
![GitHub last commit](https://img.shields.io/github/last-commit/your-username/your-repo?style=flat-square)
![GitHub repo size](https://img.shields.io/github/repo-size/your-username/your-repo?style=flat-square)

</div>

---

## 📌 About This Repository

> 👋 Hi! I'm a **Cloud/DevOps enthusiast** building real-world AWS automation projects to solve common infrastructure problems — cost control, security auditing, and operational efficiency — using **serverless architecture**.

Each project below is self-contained, documented, and includes architecture notes, IAM policies, and a discussion of when to use Lambda vs. native AWS services. Click into any project folder for full code, setup steps, and screenshots.

📫 **Let's connect:** [LinkedIn](#) • [Portfolio](#) • [Email](#)

---

## 🧰 Tech Stack & Tools

| Category | Tools / Services |
|---|---|
| ☁️ **Cloud Provider** | Amazon Web Services (AWS) |
| ⚙️ **Compute** | AWS Lambda (Python 3.12) |
| 📦 **Storage** | Amazon S3 |
| 💾 **Compute Storage** | Amazon EBS (Elastic Block Store) |
| 🖥️ **Virtual Servers** | Amazon EC2 |
| ⏰ **Scheduling / Events** | Amazon EventBridge |
| 📣 **Notifications** | Amazon SNS |
| 🔐 **Security & IAM** | AWS IAM (least-privilege inline policies) |
| 🐍 **SDK / Language** | Python 3.12+, Boto3 |
| 🕵️ **Auditing** | CloudTrail (bonus scenario) |

---

## 🗂️ Projects Overview

| # | Project | Core AWS Services | Trigger Type | Status |
|---|---|---|---|---|
| 1 | [S3 Bucket Auto-Cleanup](#-1-automated-s3-bucket-cleanup) | S3, Lambda | Manual / Scheduled | ✅ Complete |
| 2 | [EBS Snapshot Backup & Cleanup](#-2-automated-ebs-snapshot-creation--cleanup) | EC2 (EBS), Lambda, EventBridge | Weekly Schedule | ✅ Complete |
| 3 | [Auto-Tagging EC2 on Launch](#-3-auto-tagging-ec2-instances-on-launch) | EC2, Lambda, EventBridge, CloudTrail | Event-Driven | ✅ Complete |
| 4 | [S3 Public Access Auditor](#-4-audit-s3-buckets-for-public-access--notify) | S3, Lambda, SNS, EventBridge | Daily Schedule | ✅ Complete |

---

<br>

## 🪣 1. Automated S3 Bucket Cleanup
**`Objects Older Than 30 Days`**

🔗 **Repo folder:** [`/s3-bucket-cleanup`](#)

### 🎯 Objective
Automatically delete stale objects from an S3 bucket to reduce storage costs — no manual housekeeping required.

### 🛠️ How It Works
- 📋 Lists all objects in the target bucket using a **paginator** (handles buckets with 1000+ objects safely)
- 🕒 Compares each object's `LastModified` (timezone-aware) against current UTC time
- 🗑️ Deletes any object older than **30 days**
- 🖨️ Logs the names of every deleted object for auditability

### 🔐 IAM Permissions Used
```
s3:ListBucket
s3:DeleteObject   (scoped to the specific bucket ARN)
```

### 🧪 Testing Approach
Lowered the age threshold to *minutes* for quick validation, then reset to the production value (30 days) before final deployment. Verified via manual Lambda invocation that only newer files remained.

### 💡 Discussion: Lambda vs. S3 Lifecycle Rules
> In production, **S3 Lifecycle Rules** handle this natively with **zero code**. I chose Lambda here to demonstrate custom logic — this approach is better suited when you need **conditional deletion** (e.g., filename patterns, tags, or metadata-based rules) or need to **trigger cross-service actions** (like notifications or logging) alongside the delete.

**Tech used:** ![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white) ![S3](https://img.shields.io/badge/-Amazon%20S3-569A31?style=flat-square&logo=amazons3&logoColor=white) ![Lambda](https://img.shields.io/badge/-Lambda-FF9900?style=flat-square&logo=awslambda&logoColor=white)

---

<br>

## 💾 2. Automated EBS Snapshot Creation & Cleanup

🔗 **Repo folder:** [`/ebs-snapshot-automation`](#)

### 🎯 Objective
Automate EBS volume backups and enforce a retention policy by deleting snapshots older than 30 days.

### 🛠️ How It Works
- 📸 Creates a snapshot of a specified EBS volume and tags it (`CreatedBy=Lambda-Backup`)
- 🔍 Lists all snapshots owned by the account with that tag
- 🗑️ Deletes snapshots older than the retention period
- 🖨️ Prints created & deleted snapshot IDs for tracking
- ⏰ Runs automatically every week via **EventBridge**

### 🔐 IAM Permissions Used
```
ec2:CreateSnapshot
ec2:DescribeSnapshots
ec2:DeleteSnapshot
ec2:CreateTags
```

### 🧪 Testing Approach
Manually triggered the function and confirmed snapshot creation and cleanup directly in the **EC2 console**.

### 💡 Discussion: Lambda vs. Data Lifecycle Manager (DLM)
> **AWS Data Lifecycle Manager (DLM)** natively automates this exact workflow. Lambda is the better choice when you need **custom retention logic**, **cross-account snapshot copies**, or need to trigger **notifications/alerts** as part of the backup workflow.

**Tech used:** ![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white) ![EC2](https://img.shields.io/badge/-Amazon%20EC2-FF9900?style=flat-square&logo=amazonec2&logoColor=white) ![EventBridge](https://img.shields.io/badge/-EventBridge-FF4F8B?style=flat-square)

---

<br>

## 🏷️ 3. Auto-Tagging EC2 Instances on Launch

🔗 **Repo folder:** [`/ec2-auto-tagging`](#)

### 🎯 Objective
Automatically tag newly launched EC2 instances for resource tracking, ownership, and cost allocation — no manual tagging needed.

### 🛠️ How It Works
- 📥 Extracts the instance ID from the EventBridge event payload (`detail.instance-id`)
- 🏷️ Applies tags: `LaunchDate=<current date>` + custom `Owner`/`Environment` tag
- ✅ Prints a confirmation message on success
- ⚡ Triggered instantly via an **EventBridge rule** matching:
  - `source: aws.ec2`
  - `detail-type: EC2 Instance State-change Notification`
  - `state: running`

### 🔐 IAM Permissions Used
```
ec2:CreateTags
ec2:DescribeInstances
```

### 🧪 Testing Approach
Launched a new EC2 instance and confirmed the tags appeared automatically within seconds of it entering the `running` state.

### 🌟 Bonus Enhancement
> Extracted the **launching IAM user/role** from **CloudTrail** events and automatically applied it as an `Owner` tag — a scenario frequently asked about in cloud engineering interviews, since it ties together CloudTrail, EventBridge, and Lambda for real accountability tracking.

**Tech used:** ![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white) ![EC2](https://img.shields.io/badge/-Amazon%20EC2-FF9900?style=flat-square&logo=amazonec2&logoColor=white) ![CloudTrail](https://img.shields.io/badge/-CloudTrail-8C4FFF?style=flat-square)

---

<br>

## 🛡️ 4. Audit S3 Buckets for Public Access & Notify

🔗 **Repo folder:** [`/s3-public-access-auditor`](#)

### 🎯 Objective
Continuously detect any S3 bucket that is publicly accessible and send an instant alert — a critical **security compliance** guardrail.

> ⚠️ **Note:** Since April 2023, new S3 buckets have *Block Public Access* enabled and ACLs disabled by default — so this audit checks **Block Public Access config + bucket policy status**, not just ACLs, for a complete picture.

### 🛠️ How It Works
- 📋 Lists all buckets in the account
- 🔎 For each bucket, checks:
  - `get_public_access_block` (Block Public Access settings)
  - `get_bucket_policy_status` (`IsPublic` flag)
  - `get_bucket_acl` (public grants)
- 🚨 Publishes an **SNS alert** naming any bucket found public or with Block Public Access disabled
- ⏰ Runs automatically **once a day** via EventBridge

### 🔐 IAM Permissions Used
```
s3:ListAllMyBuckets
s3:GetBucketPublicAccessBlock
s3:GetBucketPolicyStatus
s3:GetBucketAcl
sns:Publish
```

### 🧪 Testing Approach
Deliberately disabled Block Public Access on a test bucket and attached a public-read bucket policy → confirmed the SNS email alert fired correctly → immediately re-secured the bucket after validation.

**Tech used:** ![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white) ![S3](https://img.shields.io/badge/-Amazon%20S3-569A31?style=flat-square&logo=amazons3&logoColor=white) ![SNS](https://img.shields.io/badge/-Amazon%20SNS-DD344C?style=flat-square)

---

## 📁 Suggested Repo Structure

```
aws-automation-projects/
│
├── s3-bucket-cleanup/
│   ├── lambda_function.py
│   ├── iam_policy.json
│   └── README.md
│
├── ebs-snapshot-automation/
│   ├── lambda_function.py
│   ├── iam_policy.json
│   └── README.md
│
├── ec2-auto-tagging/
│   ├── lambda_function.py
│   ├── iam_policy.json
│   └── README.md
│
├── s3-public-access-auditor/
│   ├── lambda_function.py
│   ├── iam_policy.json
│   └── README.md
│
└── README.md   ← (this file)
```

---

## 🎓 Key Skills Demonstrated

✔️ Serverless architecture design with AWS Lambda
✔️ Least-privilege IAM policy design
✔️ Event-driven automation with EventBridge
✔️ Boto3 scripting & pagination handling
✔️ Cost optimization & security compliance automation
✔️ Understanding trade-offs between custom code vs. native AWS features

---

<div align="center">

### ⭐ If you found this useful, consider starring the repo!

**Made with ☁️ + 🐍 + a lot of `boto3.client()` calls**

</div>

