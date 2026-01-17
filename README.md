# AWS-Cli-GitHub-Copilot-Fusion

> **"What was that S3 bucket I created last month?"**
> **"How do I check my AWS bill again?"**
> **"Which profile am I using right now?"**

Sound familiar? 

If you're a developer running AWS from your laptop, you've probably Googled the same CLI commands dozens of times. You copy-paste, tweak, forget, and repeat.

**What if you could just *ask*?**

This repo flips the script. Define your setup once, then talk to AWS naturally — forever. No memorizing commands. No digging through history. Just ask questions like you would a colleague, and GitHub Copilot handles the rest.

*"Upload this file to my dev bucket."* Done.
*"How much did I spend this month?"* Here's a dashboard.
*"Create a QuickSight report from this Excel."* Link copied.

Everything you do is guided, formatted, and remembered. Your AWS workflow becomes a conversation.

---

A hands-on guide for integrating AWS CLI with GitHub Copilot using custom instructions. Enables developers to manage AWS profiles using natural language commands like `/aws-profile check dev` or `/aws-profile create all`. Features resilient profile creation, interactive prompts, and cross-environment support for dev, qa, staging, and prod workflows.

---

## 🤔 Why This Exists

**The Problem:**
```bash
aws ce get-cost-and-usage --time-period Start=2025-01-01,End=2025-01-16 \
  --granularity MONTHLY --metrics "UnblendedCost" --output json | jq '.ResultsByTime[].Total'
```
☝️ Nobody wants to remember this.

**The Solution:**
```
You: /aws-billing cost
```
```
╔══════════════════════════════════════════════════════════════╗
║                 AWS COST SUMMARY - January 2025              ║
╠══════════════════════════════════════════════════════════════╣
║  Period:        Jan 01 - Jan 16, 2025                        ║
║  Current Cost:  $1,234.56                                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 🎬 See It In Action

### Installing AWS CLI

**What you ask:**
> "I need to install AWS CLI on my machine"

**What Copilot does:**
- Detects your OS (Windows/macOS/Linux)
- Downloads the correct installer
- Runs installation silently
- Verifies with `aws --version`

**What you see:**
```
✅ AWS CLI v2.15.0 installed successfully!
```

---

### Setting Up Profiles

**What you ask:**
> "Set up my AWS profiles for all environments"

**What Copilot does:**
- Checks which profiles already exist (skips them)
- Prompts you before creating each one
- Handles if you cancel midway
- Resumes from where you left off next time

**What you see:**
```
╔══════════════════════════════════════════════════════════════╗
║  awscli-ghcopilot-dev       ✅ Already configured            ║
║  awscli-ghcopilot-qa        ⏳ Creating... Enter credentials ║
║  awscli-ghcopilot-staging   ⏸️  Pending                       ║
║  awscli-ghcopilot-prod      ⏸️  Pending                       ║
╚══════════════════════════════════════════════════════════════╝
```

---

### Checking Your AWS Bill

**What you ask:**
> "How much have I spent this month on AWS?"

**What Copilot does:**
- Fetches cost data from Cost Explorer
- Parses the JSON internally
- Formats it beautifully

**What you see:**
```
╔══════════════════════════════════════════════════════════════╗
║  💰 CURRENT MONTH: $1,234.56                                 ║
║  📊 TOP SERVICES                                             ║
║     1. EC2        $542.30  (44%)                             ║
║     2. RDS        $312.15  (25%)                             ║
║     3. S3         $156.78  (13%)                             ║
║  ⚠️  ALERT: Projected to exceed budget by $456               ║
╚══════════════════════════════════════════════════════════════╝
```

---

### Working with S3

**What you ask:**
> "Upload my quarterly report to the dev bucket"

**What you see:**
```
╔══════════════════════════════════════════════════════════════╗
║  📤 UPLOADING TO S3                                          ║
║  Source:      /Users/you/Documents/Q1-Report.pdf             ║
║  Destination: s3://ghcopilot-dev-uploads/Q1-Report.pdf       ║
║  Progress:    ████████████████████░░░░ 85%                   ║
╚══════════════════════════════════════════════════════════════╝
```

**What you ask:**
> "Find all reports across my S3 buckets"

**What you see:**
```
╔══════════════════════════════════════════════════════════════╗
║  🔍 SEARCH RESULTS: "report"                                 ║
╠══════════════════════════════════════════════════════════════╣
║  ghcopilot-dev-uploads      Q1-Report.pdf           2.4 MB   ║
║  ghcopilot-dev-uploads      Q2-Report.pdf           1.8 MB   ║
║  ghcopilot-prod-backups     Annual-Report.zip       45 MB    ║
╠══════════════════════════════════════════════════════════════╣
║  Found: 3 files across 2 buckets                             ║
╚══════════════════════════════════════════════════════════════╝
```

**What you ask:**
> "Email the Q1 report to john@company.com"

**What happens:**
- Downloads file from S3
- Opens your default email app
- Attaches the file
- Auto-fills a description

**What you see:**
```
╔══════════════════════════════════════════════════════════════╗
║  📧 EMAIL READY                                              ║
║  To:       john@company.com                                  ║
║  Attached: Q1-Report.pdf (2.4 MB)                            ║
║  ✅ Description auto-filled                                  ║
║  📝 Review and click Send!                                   ║
╚══════════════════════════════════════════════════════════════╝
```

---

### Creating Dashboards from Excel

**What you ask:**
> "Create a QuickSight dashboard from my sales spreadsheet"

**What Copilot does:**
- Validates your Excel file
- Uploads to S3 staging
- Creates dataset, analysis, and dashboard
- Generates shareable link

**What you see:**
```
╔══════════════════════════════════════════════════════════════╗
║  📊 CREATING QUICKSIGHT DASHBOARD                            ║
╠══════════════════════════════════════════════════════════════╣
║  ├── ✅ Excel validated (3 sheets, 15,432 rows)              ║
║  ├── ✅ Uploaded to S3                                       ║
║  ├── ✅ Dataset created                                      ║
║  ├── ✅ Dashboard created                                    ║
║  └── ✅ Link generated                                       ║
╠══════════════════════════════════════════════════════════════╣
║  🔗 https://quicksight.aws.amazon.com/sn/dashboards/...      ║
║  📋 Link copied to clipboard!                                ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 🌍 Supported Environments

| Environment | Profile Name | Primary Region | DR Region |
|-------------|--------------|----------------|-----------|
| Development | `awscli-ghcopilot-dev` | us-east-1 | — |
| QA | `awscli-ghcopilot-qa` | us-east-1 | — |
| Staging | `awscli-ghcopilot-staging` | us-east-1 | us-west-2 |
| Production | `awscli-ghcopilot-prod` | us-east-1 | us-west-2 |

---

## ⚡ Quick Reference

| I want to... | I type... |
|--------------|-----------|
| Install AWS CLI | `/aws-cli install` |
| Check my current identity | `/aws-profile whoami` |
| Switch to production | `/aws-profile switch prod` |
| See this month's AWS bill | `/aws-billing summary` |
| Upload a file to S3 | `/aws-s3 upload ./file.pdf bucket-name` |
| Search for files in S3 | `/aws-s3 search "report"` |
| Create a dashboard from Excel | `/aws-quicksight create from local ./data.xlsx` |
| Delete all dashboards | `/aws-quicksight delete all` |

---

## 🛡️ Features

- **Resilient Profile Creation** — Interrupted? Just run again. It picks up where you left off.
- **No Raw JSON** — Every output is formatted beautifully. Never scroll through JSON again.
- **OS Detection** — Works on Windows, macOS, and Linux automatically.
- **Cross-Platform Sync** — Move files between S3 and Dropbox seamlessly.
- **One-Click Email** — Share S3 files via email with auto-generated descriptions.
- **Progress Indicators** — See upload/download progress in real-time.

---

## 📺 YouTube Tutorial

🎬 **Coming Soon:** *AWS CLI and GitHub Copilot Integration Use Cases*

---

## 🤝 Contributing

Found a bug? Have an idea? Open an issue or submit a PR!