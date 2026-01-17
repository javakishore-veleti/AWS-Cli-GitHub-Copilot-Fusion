# AWS S3 Operations Instructions

This instruction file helps GitHub Copilot assist developers with S3 bucket operations, file uploads, searches, cross-platform transfers (Dropbox ↔ S3), and email sharing. When a developer uses `/aws-s3` commands, Copilot should execute operations silently and display only formatted results.

**Important:** Never show raw JSON or verbose AWS CLI output. Always parse and format results automatically.

<Commands>

| Command | Description |
|---------|-------------|
| `/aws-s3 create bucket <name>` | Create a new S3 bucket |
| `/aws-s3 list buckets` | List all S3 buckets |
| `/aws-s3 upload <local-path> <bucket>` | Upload file/folder to S3 bucket |
| `/aws-s3 upload <local-path> <bucket>/<prefix>` | Upload to specific path in bucket |
| `/aws-s3 list files <bucket>` | List all files in a bucket |
| `/aws-s3 list files <bucket>/<prefix>` | List files under specific prefix |
| `/aws-s3 search <pattern>` | Search for files across all buckets |
| `/aws-s3 search <pattern> in <bucket>` | Search for files in specific bucket |
| `/aws-s3 download <bucket>/<key> <local-path>` | Download file from S3 |
| `/aws-s3 delete <bucket>/<key>` | Delete file from S3 |
| `/aws-s3 sync dropbox <dropbox-path> to <bucket>` | Sync Dropbox folder to S3 |
| `/aws-s3 sync s3 <bucket>/<prefix> to dropbox <dropbox-path>` | Sync S3 to Dropbox |
| `/aws-s3 email <bucket>/<key>` | Open email app with file attachment |
| `/aws-s3 email <bucket>/<key> to <recipient>` | Open email with recipient pre-filled |

</Commands>

<Goals>
- Enable developers to manage S3 buckets and files using natural language commands
- Upload files from local laptop to S3 with progress indication
- Search files across buckets or within specific bucket patterns
- Transfer files between Dropbox and S3 seamlessly
- Open default email app with S3 file as attachment and auto-generated description
- Display only clean, formatted output — never raw CLI output
</Goals>

<Limitations>
- Requires AWS CLI v2 installed (see 01-aws-cli-install.instructions.md)
- Requires active AWS profile (see 02-aws-profiles-setup.instructions.md)
- Requires IAM permissions: s3:CreateBucket, s3:PutObject, s3:GetObject, s3:ListBucket, s3:DeleteObject
- Dropbox CLI must be installed for Dropbox sync operations
- Email feature requires default mail app configured on laptop
- Large file uploads may take time — progress shown automatically
</Limitations>

<HighLevelDetails>

## Required IAM Permissions
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket",
                "s3:ListAllMyBuckets",
                "s3:ListBucket",
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:GetBucketLocation"
            ],
            "Resource": "*"
        }
    ]
}
```

## Bucket Naming Convention
- Prefix: `ghcopilot-<env>-<purpose>`
- Examples: `ghcopilot-dev-uploads`, `ghcopilot-prod-assets`

</HighLevelDetails>

<BuildInstructions>

## Execution Flow

For ALL `/aws-s3` commands, Copilot MUST:
1. Validate AWS profile is set (if not, prompt user to run `/aws-profile switch <env>`)
2. Execute AWS CLI commands silently in the background
3. Show progress for uploads/downloads
4. Parse and display ONLY formatted output
5. NEVER show raw JSON or verbose CLI output

---

## /aws-s3 create bucket <name>

**Internal commands (do not show to user):**
aws s3api create-bucket --bucket <name> --region $AWS_REGION --create-bucket-configuration LocationConstraint=$AWS_REGION

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║                 ✅ S3 BUCKET CREATED                         ║
╠══════════════════════════════════════════════════════════════╣
║  Bucket Name:   ghcopilot-dev-uploads                        ║
║  Region:        us-east-1                                    ║
║  Created:       Jan 16, 2025 10:30:45 AM                     ║
║  ARN:           arn:aws:s3:::ghcopilot-dev-uploads           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 list buckets

**Internal command (do not show to user):**
aws s3api list-buckets --output json

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║                    YOUR S3 BUCKETS                           ║
╠══════════════════════════════════════════════════════════════╣
║  #   Bucket Name                    Region      Created      ║
╠══════════════════════════════════════════════════════════════╣
║  1   ghcopilot-dev-uploads          us-east-1   Jan 10, 2025 ║
║  2   ghcopilot-dev-assets           us-east-1   Jan 12, 2025 ║
║  3   ghcopilot-prod-backups         us-east-1   Jan 05, 2025 ║
║  4   ghcopilot-staging-logs         us-west-2   Jan 08, 2025 ║
╠══════════════════════════════════════════════════════════════╣
║  Total: 4 buckets                                            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 upload <local-path> <bucket>

**Internal command (do not show to user):**
aws s3 cp <local-path> s3://<bucket>/ --recursive (if folder)
aws s3 cp <local-path> s3://<bucket>/ (if file)

**Display this output (with live progress):**
```
╔══════════════════════════════════════════════════════════════╗
║                    📤 UPLOADING TO S3                        ║
╠══════════════════════════════════════════════════════════════╣
║  Source:        /Users/dev/Documents/report.pdf              ║
║  Destination:   s3://ghcopilot-dev-uploads/report.pdf        ║
║  Size:          2.4 MB                                       ║
║  Progress:      ████████████████████░░░░ 85%                 ║
╚══════════════════════════════════════════════════════════════╝
```

**On completion:**
```
╔══════════════════════════════════════════════════════════════╗
║                 ✅ UPLOAD COMPLETE                           ║
╠══════════════════════════════════════════════════════════════╣
║  File:          report.pdf                                   ║
║  Bucket:        ghcopilot-dev-uploads                        ║
║  S3 URI:        s3://ghcopilot-dev-uploads/report.pdf        ║
║  Size:          2.4 MB                                       ║
║  Time:          3.2 seconds                                  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 list files <bucket>

**Internal command (do not show to user):**
aws s3api list-objects-v2 --bucket <bucket> --output json

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║          FILES IN: ghcopilot-dev-uploads                     ║
╠══════════════════════════════════════════════════════════════╣
║  Name                          Size       Last Modified      ║
╠══════════════════════════════════════════════════════════════╣
║  📄 report.pdf                 2.4 MB     Jan 16, 2025 10:30 ║
║  📄 data.csv                   156 KB     Jan 15, 2025 14:22 ║
║  📁 images/                    —          —                  ║
║     ├── logo.png               45 KB      Jan 14, 2025 09:15 ║
║     ├── banner.jpg             1.2 MB     Jan 14, 2025 09:16 ║
║     └── icon.svg               12 KB      Jan 14, 2025 09:17 ║
║  📄 config.json                8 KB       Jan 12, 2025 11:00 ║
╠══════════════════════════════════════════════════════════════╣
║  Total: 6 files | 3.8 MB                                     ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 search <pattern>

**Internal commands (do not show to user):**
1. aws s3api list-buckets (get all buckets)
2. For each bucket: aws s3api list-objects-v2 --bucket <bucket> --query "Contents[?contains(Key, '<pattern>')]"

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║              🔍 SEARCH RESULTS: "report"                     ║
╠══════════════════════════════════════════════════════════════╣
║  Bucket                        File                    Size  ║
╠══════════════════════════════════════════════════════════════╣
║  ghcopilot-dev-uploads         report.pdf              2.4MB ║
║  ghcopilot-dev-uploads         reports/q1-report.xlsx  1.1MB ║
║  ghcopilot-prod-backups        report-2024.zip         45MB  ║
║  ghcopilot-staging-logs        error-report.log        230KB ║
╠══════════════════════════════════════════════════════════════╣
║  Found: 4 files matching "report" across 3 buckets           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 search <pattern> in <bucket>

**Internal command (do not show to user):**
aws s3api list-objects-v2 --bucket <bucket> --query "Contents[?contains(Key, '<pattern>')]" --output json

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║     🔍 SEARCH: "config" IN ghcopilot-dev-uploads             ║
╠══════════════════════════════════════════════════════════════╣
║  File                              Size       Modified       ║
╠══════════════════════════════════════════════════════════════╣
║  config.json                       8 KB       Jan 12, 2025   ║
║  settings/app-config.yaml          4 KB       Jan 10, 2025   ║
║  backups/config-old.json           6 KB       Jan 05, 2025   ║
╠══════════════════════════════════════════════════════════════╣
║  Found: 3 files matching "config"                            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 download <bucket>/<key> <local-path>

**Internal command (do not show to user):**
aws s3 cp s3://<bucket>/<key> <local-path>

**Display this output (with live progress):**
```
╔══════════════════════════════════════════════════════════════╗
║                    📥 DOWNLOADING FROM S3                    ║
╠══════════════════════════════════════════════════════════════╣
║  Source:        s3://ghcopilot-dev-uploads/report.pdf        ║
║  Destination:   /Users/dev/Downloads/report.pdf              ║
║  Size:          2.4 MB                                       ║
║  Progress:      ██████████████████░░░░░░ 75%                 ║
╚══════════════════════════════════════════════════════════════╝
```

**On completion:**
```
╔══════════════════════════════════════════════════════════════╗
║                 ✅ DOWNLOAD COMPLETE                         ║
╠══════════════════════════════════════════════════════════════╣
║  File:          report.pdf                                   ║
║  Saved to:      /Users/dev/Downloads/report.pdf              ║
║  Size:          2.4 MB                                       ║
║  Time:          2.8 seconds                                  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 delete <bucket>/<key>

**Internal command (do not show to user):**
aws s3 rm s3://<bucket>/<key>

**First, prompt for confirmation:**
```
╔══════════════════════════════════════════════════════════════╗
║              ⚠️  CONFIRM DELETE                              ║
╠══════════════════════════════════════════════════════════════╣
║  File:          report.pdf                                   ║
║  Bucket:        ghcopilot-dev-uploads                        ║
║  Size:          2.4 MB                                       ║
║                                                              ║
║  Are you sure you want to delete this file? (y/n)            ║
╚══════════════════════════════════════════════════════════════╝
```

**On confirmation:**
```
╔══════════════════════════════════════════════════════════════╗
║                 ✅ FILE DELETED                              ║
╠══════════════════════════════════════════════════════════════╣
║  File:          report.pdf                                   ║
║  Bucket:        ghcopilot-dev-uploads                        ║
║  Deleted at:    Jan 16, 2025 10:45:00 AM                     ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 sync dropbox <dropbox-path> to <bucket>

**Prerequisites:**
- Dropbox CLI installed: https://www.dropbox.com/install-linux (Linux) or Dropbox desktop app
- Dropbox folder synced locally (default: ~/Dropbox)

**Internal commands (do not show to user):**
aws s3 sync ~/Dropbox/<dropbox-path> s3://<bucket>/<dropbox-path>/

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║              📦 SYNCING: DROPBOX → S3                        ║
╠══════════════════════════════════════════════════════════════╣
║  Source:        Dropbox/Projects/2025                        ║
║  Destination:   s3://ghcopilot-dev-uploads/Projects/2025     ║
╠══════════════════════════════════════════════════════════════╣
║  Scanning files...                                           ║
║  ├── 📄 proposal.docx          ✅ Uploaded (1.2 MB)          ║
║  ├── 📄 budget.xlsx            ✅ Uploaded (340 KB)          ║
║  ├── 📄 notes.txt              ⏭️  Skipped (unchanged)       ║
║  └── 📁 assets/                ✅ 5 files uploaded           ║
╠══════════════════════════════════════════════════════════════╣
║  ✅ Sync complete: 7 uploaded | 1 skipped | 0 failed         ║
║  Total transferred: 4.8 MB in 12.3 seconds                   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 sync s3 <bucket>/<prefix> to dropbox <dropbox-path>

**Internal commands (do not show to user):**
aws s3 sync s3://<bucket>/<prefix> ~/Dropbox/<dropbox-path>/

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║              📦 SYNCING: S3 → DROPBOX                        ║
╠══════════════════════════════════════════════════════════════╣
║  Source:        s3://ghcopilot-dev-uploads/reports           ║
║  Destination:   Dropbox/AWS-Reports                          ║
╠══════════════════════════════════════════════════════════════╣
║  Downloading...                                              ║
║  ├── 📄 q1-report.pdf          ✅ Downloaded (2.1 MB)        ║
║  ├── 📄 q2-report.pdf          ✅ Downloaded (1.8 MB)        ║
║  └── 📄 summary.xlsx           ✅ Downloaded (540 KB)        ║
╠══════════════════════════════════════════════════════════════╣
║  ✅ Sync complete: 3 downloaded | 0 skipped | 0 failed       ║
║  Total transferred: 4.4 MB in 8.7 seconds                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 email <bucket>/<key>

**Execution flow:**
1. Download file from S3 to temp location
2. Generate file description automatically
3. Open default email app with attachment and pre-filled body

**Internal commands (do not show to user):**

### macOS
```bash
aws s3 cp s3://<bucket>/<key> /tmp/<filename>
open -a Mail
# AppleScript to compose email with attachment
```

### Windows
```powershell
aws s3 cp s3://<bucket>/<key> $env:TEMP\<filename>
Start-Process "mailto:?subject=File: <filename>&body=<auto-description>&attachment=$env:TEMP\<filename>"
```

### Linux
```bash
aws s3 cp s3://<bucket>/<key> /tmp/<filename>
xdg-email --attach /tmp/<filename> --subject "File: <filename>" --body "<auto-description>"
```

**Auto-generated description template:**
```
Hi,

Please find attached the file "<filename>" from our AWS S3 storage.

File Details:
- Name: <filename>
- Size: <size>
- Source: s3://<bucket>/<key>
- Downloaded: <timestamp>

Let me know if you have any questions.

Best regards
```

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║              📧 EMAIL READY                                  ║
╠══════════════════════════════════════════════════════════════╣
║  File:          report.pdf                                   ║
║  Size:          2.4 MB                                       ║
║  Source:        s3://ghcopilot-dev-uploads/report.pdf        ║
╠══════════════════════════════════════════════════════════════╣
║  ✅ Default email app opened                                 ║
║  ✅ File attached                                            ║
║  ✅ Description auto-filled                                  ║
║                                                              ║
║  📝 Just add recipient and click Send!                       ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-s3 email <bucket>/<key> to <recipient>

Same as above but with recipient pre-filled:

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║              📧 EMAIL READY                                  ║
╠══════════════════════════════════════════════════════════════╣
║  To:            john.doe@company.com                         ║
║  File:          report.pdf                                   ║
║  Size:          2.4 MB                                       ║
║  Source:        s3://ghcopilot-dev-uploads/report.pdf        ║
╠══════════════════════════════════════════════════════════════╣
║  ✅ Default email app opened                                 ║
║  ✅ Recipient added                                          ║
║  ✅ File attached                                            ║
║  ✅ Description auto-filled                                  ║
║                                                              ║
║  📝 Review and click Send!                                   ║
╚══════════════════════════════════════════════════════════════╝
```

</BuildInstructions>

<CommonErrors>

When errors occur, display in formatted style:

```
╔══════════════════════════════════════════════════════════════╗
║                    ❌ ERROR                                  ║
╠══════════════════════════════════════════════════════════════╣
║  Error:   BucketAlreadyExists                                ║
║  Cause:   Bucket name is already taken globally              ║
║  Fix:     Choose a unique bucket name with your prefix       ║
╚══════════════════════════════════════════════════════════════╝
```

| Error | Cause | Fix |
|-------|-------|-----|
| BucketAlreadyExists | Bucket name taken globally | Use unique name: ghcopilot-<env>-<unique> |
| NoSuchBucket | Bucket doesn't exist | Check bucket name or create it first |
| AccessDenied | Missing S3 permissions | Add s3: permissions to IAM user |
| NoSuchKey | File not found in bucket | Verify file path with `/aws-s3 list files` |
| Dropbox folder not found | Dropbox not synced locally | Install Dropbox and sync folder |
| Email app not configured | No default mail app | Set default mail app in OS settings |

</CommonErrors>

<AlwaysNever>

Always:
- Validate AWS profile is set before any S3 operation
- Show progress for uploads/downloads larger than 1MB
- Use formatted output — never raw JSON or CLI output
- Include file size and timestamp in listings
- Auto-generate meaningful email descriptions
- Clean up temp files after email operation
- Use folder icons (📁) and file icons (📄) in listings
- Prompt for confirmation before deleting files

Never:
- Show raw AWS CLI output
- Upload without confirming destination
- Delete files without confirmation prompt
- Leave temp files after email operation
- Send email automatically — always let user review first
- Expose full S3 ARN in casual listings (use S3 URI instead)

</AlwaysNever>