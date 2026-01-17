# AWS QuickSight Instructions

This instruction file helps GitHub Copilot assist developers with creating AWS QuickSight dashboards from Excel files (local or S3), managing QuickSight resources, and sharing dashboard links. When a developer uses `/aws-quicksight` commands, Copilot should execute operations silently and display only formatted results.

**Important:** Never show raw JSON or verbose AWS CLI output. Always parse and format results automatically.

<Commands>

| Command | Description |
|---------|-------------|
| `/aws-quicksight create from local <excel-path>` | Create dashboard from local Excel file |
| `/aws-quicksight create from s3 <bucket>/<key>` | Create dashboard from Excel in S3 |
| `/aws-quicksight create from local <excel-path> --name <dashboard-name>` | Create with custom name |
| `/aws-quicksight list` | List all QuickSight dashboards |
| `/aws-quicksight list datasets` | List all QuickSight datasets |
| `/aws-quicksight list analyses` | List all QuickSight analyses |
| `/aws-quicksight link <dashboard-name>` | Get shareable link for dashboard |
| `/aws-quicksight open <dashboard-name>` | Open dashboard in browser |
| `/aws-quicksight delete <dashboard-name>` | Delete specific dashboard |
| `/aws-quicksight delete multiple <name1> <name2>` | Delete multiple dashboards |
| `/aws-quicksight delete all` | Delete all dashboards (with confirmation) |
| `/aws-quicksight status` | Show QuickSight account status and usage |

</Commands>

<Goals>
- Enable developers to create QuickSight dashboards from Excel files using natural language commands
- Support both local Excel files and Excel files stored in S3
- Automatically handle data upload, dataset creation, and dashboard generation
- Provide shareable links for created dashboards
- List, manage, and delete QuickSight resources easily
- Display only clean, formatted output — never raw CLI output
</Goals>

<Limitations>
- Requires AWS CLI v2 installed (see 01-aws-cli-install.instructions.md)
- Requires active AWS profile (see 02-aws-profiles-setup.instructions.md)
- Requires QuickSight Enterprise or Standard edition enabled in AWS account
- Requires IAM permissions for QuickSight and S3
- Excel files must be .xlsx or .xls format
- Maximum file size: 1GB for SPICE import
- QuickSight must be enabled in the AWS region
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
                "quicksight:CreateDataSet",
                "quicksight:CreateDataSource",
                "quicksight:CreateAnalysis",
                "quicksight:CreateDashboard",
                "quicksight:DeleteDataSet",
                "quicksight:DeleteDataSource",
                "quicksight:DeleteAnalysis",
                "quicksight:DeleteDashboard",
                "quicksight:DescribeDashboard",
                "quicksight:DescribeDataSet",
                "quicksight:ListDashboards",
                "quicksight:ListDataSets",
                "quicksight:ListAnalyses",
                "quicksight:GetDashboardEmbedUrl",
                "quicksight:UpdateDashboardPermissions"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::ghcopilot-*",
                "arn:aws:s3:::ghcopilot-*/*"
            ]
        }
    ]
}
```

## Naming Convention
- Datasets: `ghcopilot-dataset-<timestamp>-<filename>`
- Analyses: `ghcopilot-analysis-<timestamp>-<filename>`
- Dashboards: `ghcopilot-dashboard-<timestamp>-<filename>` or custom name

</HighLevelDetails>

<BuildInstructions>

## Execution Flow

For ALL `/aws-quicksight` commands, Copilot MUST:
1. Validate AWS profile is set (if not, prompt user to run `/aws-profile switch <env>`)
2. Verify QuickSight is enabled in the region
3. Execute AWS CLI commands silently in the background
4. Parse and display ONLY formatted output
5. NEVER show raw JSON or verbose CLI output

---

## /aws-quicksight create from local <excel-path>

**Execution flow:**
1. Validate Excel file exists and is valid format (.xlsx, .xls)
2. Upload Excel to S3 staging bucket (ghcopilot-<env>-quicksight-staging)
3. Create QuickSight data source pointing to S3
4. Create QuickSight dataset from data source
5. Create QuickSight analysis from dataset
6. Create QuickSight dashboard from analysis
7. Generate shareable link
8. Display results

**Internal commands (do not show to user):**
```bash
# Upload to S3
aws s3 cp <excel-path> s3://ghcopilot-<env>-quicksight-staging/<filename>

# Create manifest file for QuickSight
cat > /tmp/manifest.json << EOF
{
    "fileLocations": [
        {"URIs": ["s3://ghcopilot-<env>-quicksight-staging/<filename>"]}
    ],
    "globalUploadSettings": {
        "format": "XLSX"
    }
}
EOF

# Create data source
aws quicksight create-data-source --aws-account-id <account-id> --data-source-id <id> --name <name> --type S3 --data-source-parameters '{"S3Parameters":{"ManifestFileLocation":{"Bucket":"...","Key":"..."}}}'

# Create dataset
aws quicksight create-data-set --aws-account-id <account-id> --data-set-id <id> --name <name> --physical-table-map '...' --import-mode SPICE

# Create analysis
aws quicksight create-analysis --aws-account-id <account-id> --analysis-id <id> --name <name> --source-entity '...'

# Create dashboard
aws quicksight create-dashboard --aws-account-id <account-id> --dashboard-id <id> --name <name> --source-entity '...'

# Get dashboard URL
aws quicksight get-dashboard-embed-url --aws-account-id <account-id> --dashboard-id <id> --identity-type IAM
```

**Display this output (with progress):**
```
╔══════════════════════════════════════════════════════════════╗
║           📊 CREATING QUICKSIGHT DASHBOARD                   ║
╠══════════════════════════════════════════════════════════════╣
║  Source:        /Users/dev/Documents/sales-data.xlsx         ║
║  File Size:     2.4 MB                                       ║
║  Sheets Found:  3 (Sales, Products, Regions)                 ║
╠══════════════════════════════════════════════════════════════╣
║  Progress:                                                   ║
║  ├── ✅ Excel file validated                                 ║
║  ├── ✅ Uploaded to S3 staging                               ║
║  ├── ✅ Data source created                                  ║
║  ├── ✅ Dataset created (SPICE import)                       ║
║  ├── ⏳ Creating analysis...                                 ║
║  ├── ⏳ Creating dashboard...                                ║
║  └── ⏳ Generating link...                                   ║
╚══════════════════════════════════════════════════════════════╝
```

**On completion:**
```
╔══════════════════════════════════════════════════════════════╗
║           ✅ QUICKSIGHT DASHBOARD CREATED                    ║
╠══════════════════════════════════════════════════════════════╣
║  Dashboard:     ghcopilot-dashboard-20250116-sales-data      ║
║  Dataset:       ghcopilot-dataset-20250116-sales-data        ║
║  Source File:   sales-data.xlsx                              ║
║  Rows Imported: 15,432                                       ║
║  Created:       Jan 16, 2025 10:30:45 AM                     ║
╠══════════════════════════════════════════════════════════════╣
║  🔗 DASHBOARD LINK                                           ║
║  ────────────────────────────────────────────────────────────║
║  https://us-east-1.quicksight.aws.amazon.com/sn/dashboards/  ║
║  ghcopilot-dashboard-20250116-sales-data                     ║
║                                                              ║
║  📋 Link copied to clipboard!                                ║
╠══════════════════════════════════════════════════════════════╣
║  💡 Tip: Run `/aws-quicksight open sales-data` to open in    ║
║     browser                                                  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight create from s3 <bucket>/<key>

**Same flow as above but skips the upload step.**

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║           📊 CREATING QUICKSIGHT DASHBOARD                   ║
╠══════════════════════════════════════════════════════════════╣
║  Source:        s3://ghcopilot-dev-uploads/reports/data.xlsx ║
║  File Size:     5.1 MB                                       ║
║  Sheets Found:  2 (Transactions, Summary)                    ║
╠══════════════════════════════════════════════════════════════╣
║  Progress:                                                   ║
║  ├── ✅ S3 file validated                                    ║
║  ├── ✅ Data source created                                  ║
║  ├── ✅ Dataset created (SPICE import)                       ║
║  ├── ✅ Analysis created                                     ║
║  ├── ✅ Dashboard created                                    ║
║  └── ✅ Link generated                                       ║
╠══════════════════════════════════════════════════════════════╣
║  🔗 DASHBOARD LINK                                           ║
║  ────────────────────────────────────────────────────────────║
║  https://us-east-1.quicksight.aws.amazon.com/sn/dashboards/  ║
║  ghcopilot-dashboard-20250116-data                           ║
║                                                              ║
║  📋 Link copied to clipboard!                                ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight list

**Internal command (do not show to user):**
aws quicksight list-dashboards --aws-account-id <account-id> --output json

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║                 YOUR QUICKSIGHT DASHBOARDS                   ║
╠══════════════════════════════════════════════════════════════╣
║  #   Dashboard Name                     Created      Status  ║
╠══════════════════════════════════════════════════════════════╣
║  1   ghcopilot-dashboard-sales-q1       Jan 10, 2025   ✅    ║
║  2   ghcopilot-dashboard-inventory      Jan 12, 2025   ✅    ║
║  3   ghcopilot-dashboard-revenue        Jan 14, 2025   ✅    ║
║  4   ghcopilot-dashboard-sales-data     Jan 16, 2025   ✅    ║
╠══════════════════════════════════════════════════════════════╣
║  Total: 4 dashboards                                         ║
║                                                              ║
║  💡 Run `/aws-quicksight link <name>` to get shareable link  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight list datasets

**Internal command (do not show to user):**
aws quicksight list-data-sets --aws-account-id <account-id> --output json

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║                 YOUR QUICKSIGHT DATASETS                     ║
╠══════════════════════════════════════════════════════════════╣
║  #   Dataset Name                   Rows       Import Mode   ║
╠══════════════════════════════════════════════════════════════╣
║  1   ghcopilot-dataset-sales-q1     15,432     SPICE         ║
║  2   ghcopilot-dataset-inventory    8,921      SPICE         ║
║  3   ghcopilot-dataset-revenue      42,156     SPICE         ║
║  4   ghcopilot-dataset-sales-data   15,432     SPICE         ║
╠══════════════════════════════════════════════════════════════╣
║  Total: 4 datasets | SPICE Usage: 2.3 GB of 10 GB            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight list analyses

**Internal command (do not show to user):**
aws quicksight list-analyses --aws-account-id <account-id> --output json

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║                 YOUR QUICKSIGHT ANALYSES                     ║
╠══════════════════════════════════════════════════════════════╣
║  #   Analysis Name                  Dataset          Status  ║
╠══════════════════════════════════════════════════════════════╣
║  1   ghcopilot-analysis-sales-q1    sales-q1           ✅    ║
║  2   ghcopilot-analysis-inventory   inventory          ✅    ║
║  3   ghcopilot-analysis-revenue     revenue            ✅    ║
║  4   ghcopilot-analysis-sales-data  sales-data         ✅    ║
╠══════════════════════════════════════════════════════════════╣
║  Total: 4 analyses                                           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight link <dashboard-name>

**Internal command (do not show to user):**
aws quicksight get-dashboard-embed-url --aws-account-id <account-id> --dashboard-id <dashboard-id> --identity-type IAM

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║           🔗 QUICKSIGHT DASHBOARD LINK                       ║
╠══════════════════════════════════════════════════════════════╣
║  Dashboard:     ghcopilot-dashboard-sales-data               ║
║  Created:       Jan 16, 2025                                 ║
╠══════════════════════════════════════════════════════════════╣
║  📎 SHAREABLE LINK                                           ║
║  ────────────────────────────────────────────────────────────║
║  https://us-east-1.quicksight.aws.amazon.com/sn/dashboards/  ║
║  abc123-def456-ghi789                                        ║
║                                                              ║
║  📋 Link copied to clipboard!                                ║
╠══════════════════════════════════════════════════════════════╣
║  ⚠️  Note: Viewer must have QuickSight access in this AWS    ║
║     account to view the dashboard.                           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight open <dashboard-name>

**Internal command (do not show to user):**

### macOS
```bash
open "https://us-east-1.quicksight.aws.amazon.com/sn/dashboards/<dashboard-id>"
```

### Windows
```powershell
Start-Process "https://us-east-1.quicksight.aws.amazon.com/sn/dashboards/<dashboard-id>"
```

### Linux
```bash
xdg-open "https://us-east-1.quicksight.aws.amazon.com/sn/dashboards/<dashboard-id>"
```

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║           🌐 OPENING DASHBOARD IN BROWSER                    ║
╠══════════════════════════════════════════════════════════════╣
║  Dashboard:     ghcopilot-dashboard-sales-data               ║
║  Opening in:    Default browser                              ║
║                                                              ║
║  ✅ Browser launched!                                        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight delete <dashboard-name>

**Internal commands (do not show to user):**
aws quicksight delete-dashboard --aws-account-id <account-id> --dashboard-id <dashboard-id>

**First, prompt for confirmation:**
```
╔══════════════════════════════════════════════════════════════╗
║              ⚠️  CONFIRM DELETE                              ║
╠══════════════════════════════════════════════════════════════╣
║  Dashboard:     ghcopilot-dashboard-sales-data               ║
║  Created:       Jan 16, 2025                                 ║
║  Dataset:       ghcopilot-dataset-sales-data                 ║
║                                                              ║
║  ⚠️  This will delete the dashboard only.                    ║
║     Dataset and analysis will remain.                        ║
║                                                              ║
║  Delete this dashboard? (y/n)                                ║
╚══════════════════════════════════════════════════════════════╝
```

**On confirmation:**
```
╔══════════════════════════════════════════════════════════════╗
║           ✅ DASHBOARD DELETED                               ║
╠══════════════════════════════════════════════════════════════╣
║  Dashboard:     ghcopilot-dashboard-sales-data               ║
║  Deleted at:    Jan 16, 2025 11:30:00 AM                     ║
║                                                              ║
║  💡 Dataset and analysis still exist.                        ║
║     Run `/aws-quicksight list datasets` to view.             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight delete multiple <name1> <name2>

**First, show selection and confirm:**
```
╔══════════════════════════════════════════════════════════════╗
║              ⚠️  CONFIRM MULTIPLE DELETE                     ║
╠══════════════════════════════════════════════════════════════╣
║  Dashboards to delete:                                       ║
║  ├── 1. ghcopilot-dashboard-sales-q1                         ║
║  ├── 2. ghcopilot-dashboard-inventory                        ║
║  └── 3. ghcopilot-dashboard-revenue                          ║
║                                                              ║
║  Total: 3 dashboards                                         ║
║                                                              ║
║  Delete all selected dashboards? (y/n)                       ║
╚══════════════════════════════════════════════════════════════╝
```

**On confirmation:**
```
╔══════════════════════════════════════════════════════════════╗
║           🗑️  DELETING DASHBOARDS                            ║
╠══════════════════════════════════════════════════════════════╣
║  ├── ghcopilot-dashboard-sales-q1      ✅ Deleted            ║
║  ├── ghcopilot-dashboard-inventory     ✅ Deleted            ║
║  └── ghcopilot-dashboard-revenue       ✅ Deleted            ║
╠══════════════════════════════════════════════════════════════╣
║  ✅ 3 dashboards deleted successfully                        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight delete all

**First, show warning and confirm:**
```
╔══════════════════════════════════════════════════════════════╗
║              🚨 CONFIRM DELETE ALL                           ║
╠══════════════════════════════════════════════════════════════╣
║  ⚠️  WARNING: This will delete ALL QuickSight dashboards!    ║
║                                                              ║
║  Dashboards to delete:                                       ║
║  ├── 1. ghcopilot-dashboard-sales-q1                         ║
║  ├── 2. ghcopilot-dashboard-inventory                        ║
║  ├── 3. ghcopilot-dashboard-revenue                          ║
║  └── 4. ghcopilot-dashboard-sales-data                       ║
║                                                              ║
║  Total: 4 dashboards                                         ║
║                                                              ║
║  Type "DELETE ALL" to confirm:                               ║
╚══════════════════════════════════════════════════════════════╝
```

**On confirmation:**
```
╔══════════════════════════════════════════════════════════════╗
║           🗑️  DELETING ALL DASHBOARDS                        ║
╠══════════════════════════════════════════════════════════════╣
║  ├── ghcopilot-dashboard-sales-q1      ✅ Deleted            ║
║  ├── ghcopilot-dashboard-inventory     ✅ Deleted            ║
║  ├── ghcopilot-dashboard-revenue       ✅ Deleted            ║
║  └── ghcopilot-dashboard-sales-data    ✅ Deleted            ║
╠══════════════════════════════════════════════════════════════╣
║  ✅ All 4 dashboards deleted successfully                    ║
║                                                              ║
║  💡 Datasets and analyses still exist.                       ║
║     Run `/aws-quicksight list datasets` to manage.           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-quicksight status

**Internal commands (do not show to user):**
```bash
aws quicksight describe-account-settings --aws-account-id <account-id>
aws quicksight list-dashboards --aws-account-id <account-id>
aws quicksight list-data-sets --aws-account-id <account-id>
```

**Display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║           📊 QUICKSIGHT ACCOUNT STATUS                       ║
╠══════════════════════════════════════════════════════════════╣
║  Account:       123456789012                                 ║
║  Edition:       Enterprise                                   ║
║  Region:        us-east-1                                    ║
╠══════════════════════════════════════════════════════════════╣
║  📈 USAGE                                                    ║
║  ────────────────────────────────────────────────────────────║
║  Dashboards:    4                                            ║
║  Datasets:      4                                            ║
║  Analyses:      4                                            ║
║  SPICE Used:    2.3 GB of 10 GB  ███████░░░░░░░░░░░░░ 23%    ║
╠══════════════════════════════════════════════════════════════╣
║  👥 AUTHORS                                                  ║
║  ────────────────────────────────────────────────────────────║
║  Active:        3 users                                      ║
║  Monthly Cost:  ~$54.00 (3 authors × $18/month)              ║
╚══════════════════════════════════════════════════════════════╝
```

</BuildInstructions>

<CommonErrors>

When errors occur, display in formatted style:

```
╔══════════════════════════════════════════════════════════════╗
║                    ❌ ERROR                                  ║
╠══════════════════════════════════════════════════════════════╣
║  Error:   QuickSight not enabled                             ║
║  Cause:   QuickSight is not set up in this AWS account       ║
║  Fix:     Enable QuickSight at console.aws.amazon.com/       ║
║           quicksight                                         ║
╚══════════════════════════════════════════════════════════════╝
```

| Error | Cause | Fix |
|-------|-------|-----|
| QuickSight not enabled | Not set up in account | Enable at AWS QuickSight console |
| AccessDeniedException | Missing permissions | Add quicksight: permissions to IAM |
| InvalidParameterValueException | Invalid Excel format | Ensure file is .xlsx or .xls |
| ResourceExistsException | Dashboard name exists | Use unique name or delete existing |
| SPICE capacity exceeded | No storage left | Delete unused datasets or upgrade |
| UnsupportedUserEditionException | Feature not in edition | Upgrade to Enterprise edition |

</CommonErrors>

<AlwaysNever>

Always:
- Validate AWS profile is set before any QuickSight operation
- Validate Excel file format before upload
- Show progress during dashboard creation
- Copy dashboard link to clipboard automatically
- Prompt for confirmation before any delete operation
- Require typing "DELETE ALL" for bulk delete
- Clean up temporary manifest files after creation

Never:
- Show raw AWS CLI output or JSON
- Delete dashboards without confirmation
- Delete all without explicit "DELETE ALL" confirmation
- Leave orphaned datasets/analyses without informing user
- Create dashboards with duplicate names without warning
- Expose internal AWS account IDs in casual output

</AlwaysNever>