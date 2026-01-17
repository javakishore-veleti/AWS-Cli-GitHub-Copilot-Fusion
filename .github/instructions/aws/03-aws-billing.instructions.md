# AWS Billing & Cost Management Instructions

This instruction file helps GitHub Copilot assist developers with checking AWS billing information, current month costs, budgets, and alerts. When a developer uses `/aws-billing` commands, Copilot should automatically execute the necessary AWS CLI commands, parse the JSON response internally, and display ONLY the well-formatted console output.

**Important:** Never show raw JSON or AWS CLI commands to the user. Always parse and format the output automatically.

Note: Billing commands require appropriate IAM permissions (Billing, Cost Explorer, Budgets access).

<Commands>

| Command | Description |
|---------|-------------|
| `/aws-billing cost` | Show current month's cost summary |
| `/aws-billing cost daily` | Show daily cost breakdown for current month |
| `/aws-billing cost service` | Show cost breakdown by AWS service |
| `/aws-billing budget` | Show all budgets and status |
| `/aws-billing alerts` | Show active billing alerts |
| `/aws-billing forecast` | Show end-of-month cost forecast |
| `/aws-billing summary` | Show full billing dashboard (cost + budget + alerts) |

</Commands>

<Goals>
- Enable developers to check AWS costs using natural language commands
- Automatically execute AWS CLI commands and parse JSON responses
- Display ONLY clean, formatted console output — never raw JSON
- Show budget status with visual progress indicators
- Highlight alerts and threshold warnings clearly
</Goals>

<Limitations>
- Requires AWS CLI v2 installed (see 01-aws-cli-install.instructions.md)
- Requires active AWS profile (see 02-aws-profiles-setup.instructions.md)
- Requires IAM permissions: ce:GetCostAndUsage, budgets:ViewBudget, budgets:DescribeBudgets
- Cost Explorer data has ~24 hour delay
- Billing data only available to accounts with Cost Explorer enabled
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
                "ce:GetCostAndUsage",
                "ce:GetCostForecast",
                "budgets:ViewBudget",
                "budgets:DescribeBudgets"
            ],
            "Resource": "*"
        }
    ]
}
```

## AWS Services Used
- AWS Cost Explorer (`aws ce`)
- AWS Budgets (`aws budgets`)

</HighLevelDetails>

<BuildInstructions>

## Execution Flow

For ALL `/aws-billing` commands, Copilot MUST:
1. Execute the AWS CLI command silently in the background
2. Capture and parse the JSON response internally
3. Extract relevant data fields
4. Generate and display ONLY the formatted output
5. NEVER show raw JSON or the underlying AWS CLI command to the user

---

## /aws-billing cost

**Internal command (do not show to user):**
aws ce get-cost-and-usage --time-period Start=$(date -u +%Y-%m-01),End=$(date -u +%Y-%m-%d) --granularity MONTHLY --metrics "UnblendedCost" --output json

**Parse and display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║                 AWS COST SUMMARY - January 2025              ║
╠══════════════════════════════════════════════════════════════╣
║  Period:        Jan 01 - Jan 16, 2025                        ║
║  Current Cost:  $1,234.56                                    ║
║  Currency:      USD                                          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-billing cost service

**Internal command (do not show to user):**
aws ce get-cost-and-usage --time-period Start=$(date -u +%Y-%m-01),End=$(date -u +%Y-%m-%d) --granularity MONTHLY --metrics "UnblendedCost" --group-by Type=DIMENSION,Key=SERVICE --output json

**Parse and display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║              COST BY SERVICE - January 2025                  ║
╠══════════════════════════════════════════════════════════════╣
║  Service                              Cost         % of Total║
╠══════════════════════════════════════════════════════════════╣
║  Amazon EC2                           $542.30          44%   ║
║  Amazon RDS                           $312.15          25%   ║
║  Amazon S3                            $156.78          13%   ║
║  AWS Lambda                           $89.45            7%   ║
║  Others                               $133.88          11%   ║
╠══════════════════════════════════════════════════════════════╣
║  TOTAL                              $1,234.56         100%   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-billing cost daily

**Internal command (do not show to user):**
aws ce get-cost-and-usage --time-period Start=$(date -u +%Y-%m-01),End=$(date -u +%Y-%m-%d) --granularity DAILY --metrics "UnblendedCost" --output json

**Parse and display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║              DAILY COSTS - January 2025                      ║
╠══════════════════════════════════════════════════════════════╣
║  Date          Cost        Trend                             ║
╠══════════════════════════════════════════════════════════════╣
║  Jan 01        $78.50      ████████████                      ║
║  Jan 02        $82.30      █████████████                     ║
║  Jan 03        $75.20      ███████████                       ║
║  Jan 04        $91.45      ██████████████ ▲                  ║
║  Jan 05        $68.90      ██████████ ▼                      ║
║  ...                                                         ║
╠══════════════════════════════════════════════════════════════╣
║  Average/Day:  $77.16                                        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-billing budget

**Internal command (do not show to user):**
aws budgets describe-budgets --account-id $(aws sts get-caller-identity --query Account --output text) --output json

**Parse and display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║                    AWS BUDGETS STATUS                        ║
╠══════════════════════════════════════════════════════════════╣
║  Budget Name        Limit      Actual     Status             ║
╠══════════════════════════════════════════════════════════════╣
║  Monthly-Total      $2,000     $1,234     ██████░░░░ 62%  ✅ ║
║  EC2-Budget         $800       $542       ██████░░░░ 68%  ✅ ║
║  RDS-Budget         $400       $312       ███████░░░ 78%  ⚠️ ║
║  Lambda-Budget      $100       $89        ████████░░ 89%  🚨 ║
╠══════════════════════════════════════════════════════════════╣
║  ✅ On Track   ⚠️ Warning (>75%)   🚨 Critical (>85%)        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-billing forecast

**Internal command (do not show to user):**
aws ce get-cost-forecast --time-period Start=$(date -u +%Y-%m-%d),End=$(date -u -d "+1 month" +%Y-%m-01) --metric UNBLENDED_COST --granularity MONTHLY --output json

**Parse and display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║                 END OF MONTH FORECAST                        ║
╠══════════════════════════════════════════════════════════════╣
║  Current Spend:     $1,234.56                                ║
║  Forecasted Total:  $2,456.78                                ║
║  Monthly Budget:    $2,000.00                                ║
║  Status:            ⚠️  PROJECTED TO EXCEED BUDGET BY $456.78║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-billing alerts

**Internal command (do not show to user):**
aws budgets describe-budget-notifications-for-account --account-id $(aws sts get-caller-identity --query Account --output text) --output json

**Parse and display this output:**
```
╔══════════════════════════════════════════════════════════════╗
║                    ACTIVE BILLING ALERTS                     ║
╠══════════════════════════════════════════════════════════════╣
║  🚨 CRITICAL                                                 ║
║  ────────────────────────────────────────────────────────────║
║  Lambda-Budget at 89% - threshold 85% exceeded               ║
║                                                              ║
║  ⚠️  WARNING                                                 ║
║  ────────────────────────────────────────────────────────────║
║  RDS-Budget at 78% - approaching 80% threshold               ║
║                                                              ║
║  ✅ NO ISSUES                                                ║
║  ────────────────────────────────────────────────────────────║
║  Monthly-Total, EC2-Budget - within normal limits            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## /aws-billing summary

**Internal commands (execute all silently, do not show to user):**
1. aws ce get-cost-and-usage (for current cost)
2. aws ce get-cost-and-usage with group-by SERVICE (for top services)
3. aws ce get-cost-forecast (for forecast)
4. aws budgets describe-budgets (for budget status)

**Combine all data and display this single dashboard:**
```
╔══════════════════════════════════════════════════════════════╗
║         AWS BILLING DASHBOARD - awscli-ghcopilot-dev         ║
║                     January 16, 2025                         ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  💰 CURRENT MONTH COST                                       ║
║  ────────────────────                                        ║
║  Spent:      $1,234.56 of $2,000.00 budget                   ║
║  Progress:   ██████████████░░░░░░░░ 62%                      ║
║  Days Left:  15                                              ║
║                                                              ║
║  📊 TOP SERVICES                                             ║
║  ────────────────────                                        ║
║  1. EC2        $542.30  (44%)                                ║
║  2. RDS        $312.15  (25%)                                ║
║  3. S3         $156.78  (13%)                                ║
║                                                              ║
║  📈 FORECAST                                                 ║
║  ────────────────────                                        ║
║  Projected:   $2,456.78                                      ║
║  Status:      ⚠️  Over budget by $456.78                     ║
║                                                              ║
║  🚨 ALERTS                                                   ║
║  ────────────────────                                        ║
║  ⚠️  RDS-Budget at 78% - approaching threshold               ║
║  🚨 Lambda-Budget at 89% - critical threshold reached        ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</BuildInstructions>

<CommonErrors>

When errors occur, display them in the same formatted style:

```
╔══════════════════════════════════════════════════════════════╗
║                    ❌ ERROR                                  ║
╠══════════════════════════════════════════════════════════════╣
║  Error:   AccessDeniedException                              ║
║  Cause:   Missing IAM permissions                            ║
║  Fix:     Add ce: and budgets: permissions to your IAM user  ║
╚══════════════════════════════════════════════════════════════╝
```

| Error | Cause | Fix |
|-------|-------|-----|
| AccessDeniedException | Missing IAM permissions | Add ce: and budgets: permissions |
| Cost Explorer not enabled | Account not enrolled | Enable Cost Explorer in AWS Console |
| No budgets found | No budgets created | Create budget in AWS Budgets console |
| Data not available | <24 hours since charge | Wait for Cost Explorer sync |
| No profile set | AWS_PROFILE not configured | Run `/aws-profile switch dev` first |

</CommonErrors>

<AlwaysNever>

Always:
- Execute AWS CLI commands silently in the background
- Parse JSON responses internally
- Display ONLY formatted output — never raw JSON
- Check AWS profile is set before running billing commands
- Format currency with 2 decimal places and $ symbol
- Show visual progress bars for budget status
- Include date range in output headers
- Use emoji indicators for status (✅ ⚠️ 🚨)
- Format errors in the same box style as results

Never:
- Show raw JSON output to users
- Display the underlying AWS CLI commands
- Ask user to parse or scroll through JSON
- Assume permissions exist (handle errors gracefully)
- Skip currency indicator (USD)
- Hide alerts or warnings
- Show partial/incomplete data without explanation

</AlwaysNever>