# AWS CLI Installation Instructions

This instruction file helps GitHub Copilot assist developers with installing AWS CLI v2 on their system. When a developer uses the `/aws-cli install` command, Copilot should detect the operating system and guide them through the correct installation process.

This is a one-time setup task per machine. Doing this correctly ensures all subsequent AWS operations work smoothly.

<Commands>

| Command | Description |
|---------|-------------|
| `/aws-cli install` | Detect OS and install AWS CLI v2 |
| `/aws-cli check` | Check if AWS CLI is installed |
| `/aws-cli version` | Show installed AWS CLI version |
| `/aws-cli upgrade` | Upgrade AWS CLI to latest version |
| `/aws-cli uninstall` | Remove AWS CLI from system |

</Commands>

<Goals>
- Enable developers to install AWS CLI v2 using natural language commands listed above
- Automatically detect operating system (Windows, macOS, Linux)
- Provide correct installation commands for detected OS
- Validate successful installation
- Handle common errors gracefully
</Goals>

<Limitations>
- AWS CLI v2 only (v1 is deprecated)
- Requires internet connection for download
- Windows requires Administrator privileges
- macOS PKG installer requires sudo access
</Limitations>

<HighLevelDetails>
- Supported OS: Windows 10/11, macOS (Intel & Apple Silicon), Linux (Ubuntu/Debian, RHEL/CentOS)
- Tool: AWS CLI v2
- Source: https://aws.amazon.com/cli/
</HighLevelDetails>

<BuildInstructions>

## OS Detection

Windows (PowerShell):
$env:OS -eq "Windows_NT"

macOS (Bash):
[[ "$OSTYPE" == "darwin"* ]]

Linux (Bash):
[[ "$OSTYPE" == "linux-gnu"* ]]

## Installation Commands

### Windows (Run PowerShell as Administrator)
Invoke-WebRequest -Uri "https://awscli.amazonaws.com/AWSCLIV2.msi" -OutFile "$env:TEMP\AWSCLIV2.msi"
Start-Process msiexec.exe -ArgumentList "/i $env:TEMP\AWSCLIV2.msi /qn" -Wait
Remove-Item "$env:TEMP\AWSCLIV2.msi"

### macOS (Homebrew)
brew install awscli

### macOS (PKG - if Homebrew unavailable)
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
rm AWSCLIV2.pkg

### Linux (Ubuntu/Debian)
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm -rf awscliv2.zip aws/

## Upgrade Commands

### Windows (Run PowerShell as Administrator)
Invoke-WebRequest -Uri "https://awscli.amazonaws.com/AWSCLIV2.msi" -OutFile "$env:TEMP\AWSCLIV2.msi"
Start-Process msiexec.exe -ArgumentList "/i $env:TEMP\AWSCLIV2.msi /qn" -Wait
Remove-Item "$env:TEMP\AWSCLIV2.msi"

### macOS (Homebrew)
brew upgrade awscli

### Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update
rm -rf awscliv2.zip aws/

## Uninstall Commands

### Windows
Start-Process msiexec.exe -ArgumentList "/x $(Get-WmiObject -Class Win32_Product | Where-Object { $_.Name -like '*AWS CLI*' } | Select-Object -ExpandProperty IdentifyingNumber) /qn" -Wait

### macOS (Homebrew)
brew uninstall awscli

### macOS (PKG)
sudo rm -rf /usr/local/aws-cli
sudo rm /usr/local/bin/aws
sudo rm /usr/local/bin/aws_completer

### Linux
sudo rm -rf /usr/local/aws-cli
sudo rm /usr/local/bin/aws
sudo rm /usr/local/bin/aws_completer

## Validation
Always run after installation:
aws --version

Expected output:
aws-cli/2.x.x Python/3.x.x OS/arch

</BuildInstructions>

<CommonErrors>

| Error | Cause | Fix |
|-------|-------|-----|
| aws: command not found | PATH not updated | Restart terminal |
| Permission denied (Windows) | Not Admin | Run PowerShell as Administrator |
| unzip: command not found | Missing unzip | sudo apt install unzip |
| brew: command not found | Homebrew missing | Install Homebrew first |

</CommonErrors>

<AlwaysNever>

Always:
- Restart terminal after installation
- Verify with aws --version
- Use AWS CLI v2

Never:
- Install AWS CLI v1
- Skip validation step
- Run Windows installer without Admin rights

</AlwaysNever>