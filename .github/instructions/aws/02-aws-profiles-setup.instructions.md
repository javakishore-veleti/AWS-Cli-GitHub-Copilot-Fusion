# AWS CLI Profiles Setup Instructions

This instruction file helps GitHub Copilot assist developers with creating and managing AWS CLI profiles for different environments (dev, qa, staging, prod). When a developer uses `/aws-profile` commands, Copilot should check existing profiles, prompt before creating, and handle interruptions gracefully.

This is a one-time setup task per machine. The process is resilient — if interrupted, future runs will skip already-created profiles and resume where left off.

<Commands>

| Command | Description |
|---------|-------------|
| `/aws-profile check dev` | Check if dev profile exists |
| `/aws-profile check qa` | Check if qa profile exists |
| `/aws-profile check staging` | Check if staging profile exists |
| `/aws-profile check prod` | Check if prod profile exists |
| `/aws-profile check all` | Check status of all profiles |
| `/aws-profile create dev` | Create dev profile (with prompt) |
| `/aws-profile create qa` | Create qa profile (with prompt) |
| `/aws-profile create staging` | Create staging profile (with prompt) |
| `/aws-profile create prod` | Create prod profile (with prompt) |
| `/aws-profile create all` | Create all missing profiles (resilient) |
| `/aws-profile switch dev` | Switch to dev environment |
| `/aws-profile switch prod-dr` | Switch to prod DR (us-west-2) |
| `/aws-profile whoami` | Show current AWS identity & environment |
| `/aws-profile list` | List all configured profiles |

</Commands>

<Goals>
- Enable developers to manage AWS profiles using natural language commands listed above
- Check if profiles already exist before creating
- Prompt user before creating each profile
- Handle abrupt cancellation gracefully
- Resume from where left off on future attempts
- Support multiple environments: dev, qa, staging, prod
</Goals>

<Limitations>
- Requires AWS CLI v2 installed (see 01-aws-cli-install.instructions.md)
- Requires valid AWS Access Key ID and Secret Access Key for each environment
- One profile per environment per machine
</Limitations>

<HighLevelDetails>

## Profile Naming Convention
| Profile Name | Environment | Primary Region | DR Region |
|--------------|-------------|----------------|-----------|
| `awscli-ghcopilot-dev` | Development | us-east-1 | - |
| `awscli-ghcopilot-qa` | QA | us-east-1 | - |
| `awscli-ghcopilot-staging` | Staging | us-east-1 | us-west-2 |
| `awscli-ghcopilot-prod` | Production | us-east-1 | us-west-2 |

## Environment Variables Set
- `AWS_PROFILE` — Active profile name
- `AWS_REGION` — Active AWS region
- `GHCOPILOT_ENV` — Environment identifier (dev/qa/staging/prod)

</HighLevelDetails>

<BuildInstructions>

## Check if Profile Exists

### Single Profile
aws configure get aws_access_key_id --profile awscli-ghcopilot-dev

If returns value → profile exists
If returns empty/error → profile does not exist

### All Profiles
aws configure list-profiles | grep awscli-ghcopilot

## Create Profile Commands

### Development
aws configure --profile awscli-ghcopilot-dev
# Prompts for: Access Key ID, Secret Access Key
# Set region: us-east-1
# Set output: json

### QA
aws configure --profile awscli-ghcopilot-qa

### Staging
aws configure --profile awscli-ghcopilot-staging

### Production
aws configure --profile awscli-ghcopilot-prod

## Resilient Creation Flow

For each profile in [dev, qa, staging, prod]:
1. Check if profile exists
2. If exists → Skip, inform user "Profile already configured"
3. If not exists → Ask "Create [env] profile? (y/n)"
4. If user says yes → Run aws configure --profile awscli-ghcopilot-[env]
5. If user says no or cancels → Stop, inform "Run /aws-profile create all to resume later"

## Switch Environment Commands

### Bash/Zsh
export AWS_PROFILE=awscli-ghcopilot-dev AWS_REGION=us-east-1 GHCOPILOT_ENV=dev

### PowerShell
$env:AWS_PROFILE="awscli-ghcopilot-dev"; $env:AWS_REGION="us-east-1"; $env:GHCOPILOT_ENV="dev"

## Validation
aws sts get-caller-identity

Expected output:
{
    "UserId": "AIDAXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-username"
}

</BuildInstructions>

<ShellAliases>

Add to ~/.bashrc or ~/.zshrc:

# Switch environments
alias ghcp-dev='export AWS_PROFILE=awscli-ghcopilot-dev AWS_REGION=us-east-1 GHCOPILOT_ENV=dev && echo "✅ Switched to Dev (us-east-1)"'
alias ghcp-qa='export AWS_PROFILE=awscli-ghcopilot-qa AWS_REGION=us-east-1 GHCOPILOT_ENV=qa && echo "✅ Switched to QA (us-east-1)"'
alias ghcp-staging='export AWS_PROFILE=awscli-ghcopilot-staging AWS_REGION=us-east-1 GHCOPILOT_ENV=staging && echo "✅ Switched to Staging (us-east-1)"'
alias ghcp-prod='export AWS_PROFILE=awscli-ghcopilot-prod AWS_REGION=us-east-1 GHCOPILOT_ENV=prod && echo "✅ Switched to Prod (us-east-1)"'
alias ghcp-prod-dr='export AWS_PROFILE=awscli-ghcopilot-prod AWS_REGION=us-west-2 GHCOPILOT_ENV=prod && echo "✅ Switched to Prod DR (us-west-2)"'

# Utility
alias ghcp-whoami='aws sts get-caller-identity && echo "Region: $AWS_REGION | Env: $GHCOPILOT_ENV"'
alias ghcp-clear='unset AWS_PROFILE AWS_REGION GHCOPILOT_ENV && echo "🧹 Cleared environment"'

Reload: source ~/.bashrc or source ~/.zshrc

</ShellAliases>

<CommonErrors>

| Error | Cause | Fix |
|-------|-------|-----|
| Unable to locate credentials | Profile not configured | Run `/aws-profile create [env]` |
| ExpiredToken | Session expired | Re-run credentials setup |
| Access Denied | Wrong profile or permissions | Verify with `/aws-profile whoami` |
| Region not set | Missing region in export | Include AWS_REGION in switch command |
| Profile not found | Typo in profile name | Use exact name: awscli-ghcopilot-[env] |

</CommonErrors>

<AlwaysNever>

Always:
- Check if profile exists before creating
- Prompt user before each profile creation
- Set all three variables: AWS_PROFILE, AWS_REGION, GHCOPILOT_ENV
- Verify with `aws sts get-caller-identity` after setup
- Use `us-east-1` as primary region
- Use `us-west-2` for DR (staging/prod only)

Never:
- Overwrite existing profile without user confirmation
- Skip validation step
- Hardcode credentials in scripts or files
- Share credentials across environments

</AlwaysNever>