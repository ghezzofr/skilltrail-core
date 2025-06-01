# SkillTrail Core Infrastructure

This is the AWS CDK infrastructure project for SkillTrail Core, built with Go.

The `cdk.json` file tells the CDK toolkit how to execute your app.

## 🚀 Quick Start

### Prerequisites
- AWS CLI configured with appropriate credentials
- AWS CDK installed globally (`npm install -g aws-cdk`)
- Go 1.23+ installed

### Environment Variables
Set these environment variables before running CDK commands:
```bash
export CDK_DEFAULT_ACCOUNT="your-aws-account-id"
export CDK_DEFAULT_REGION="eu-south-1"  # or your preferred region
```

### First Time Setup
1. Bootstrap CDK in your target region (only needed once per account/region):
   ```bash
   cdk bootstrap --region eu-south-1
   ```

## 📝 Command Line Usage

### CDK Commands
```bash
# Build Go application
go build -o infrastructure

# Run tests
go test -v

# Synthesize CloudFormation template
cdk synth

# Show differences between deployed and current state
cdk diff

# Deploy stack to AWS
cdk deploy --require-approval never

# Destroy stack
cdk destroy --force

# Bootstrap CDK (first time only)
cdk bootstrap --region eu-south-1
```

## 🎯 VS Code Tasks

This project includes pre-configured VS Code tasks for easier development. Access them via:
- **Keyboard shortcut**: `Cmd+Shift+B` (runs default build task)
- **Command Palette**: `Cmd+Shift+P` → "Tasks: Run Task"

### Available Tasks

| Task | Description | Command |
|------|-------------|---------|
| 🔧 **CDK: Synth** | Generate CloudFormation template | `cdk synth` |
| 🚀 **CDK: Deploy** | Deploy stack to AWS | `cdk deploy --require-approval never` |
| ⚡ **CDK: Bootstrap** | Initialize CDK in region | `cdk bootstrap --region eu-south-1` |
| 🔍 **CDK: Diff** | Show deployment differences | `cdk diff` |
| 💥 **CDK: Destroy** | Remove stack from AWS | `cdk destroy --force` |
| 🔨 **Go: Build CDK** | Compile Go application | `go build -o infrastructure` |
| 🧪 **Go: Test CDK** | Run unit tests | `go test -v` |
| 🎯 **CDK: Full Deploy** | Complete deploy workflow | Build → Synth → Deploy |

### VS Code Tasks Configuration

To set up the tasks in your own workspace, create `.vscode/tasks.json`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "🔧 CDK: Synth",
      "type": "shell",
      "command": "cdk",
      "args": ["synth"],
      "group": "build",
      "options": {
        "cwd": "${workspaceFolder}/infrastructure",
        "env": {
          "CDK_DEFAULT_ACCOUNT": "${env:CDK_DEFAULT_ACCOUNT}",
          "CDK_DEFAULT_REGION": "${env:CDK_DEFAULT_REGION}"
        }
      }
    },
    {
      "label": "🚀 CDK: Deploy",
      "type": "shell",
      "command": "cdk",
      "args": ["deploy", "--require-approval", "never"],
      "group": "build",
      "options": {
        "cwd": "${workspaceFolder}/infrastructure",
        "env": {
          "CDK_DEFAULT_ACCOUNT": "${env:CDK_DEFAULT_ACCOUNT}",
          "CDK_DEFAULT_REGION": "${env:CDK_DEFAULT_REGION}"
        }
      },
      "dependsOn": "🔧 CDK: Synth"
    },
    {
      "label": "⚡ CDK: Bootstrap",
      "type": "shell",
      "command": "cdk",
      "args": ["bootstrap", "--region", "${env:CDK_DEFAULT_REGION}"],
      "group": "build",
      "options": {
        "cwd": "${workspaceFolder}/infrastructure",
        "env": {
          "CDK_DEFAULT_ACCOUNT": "${env:CDK_DEFAULT_ACCOUNT}",
          "CDK_DEFAULT_REGION": "${env:CDK_DEFAULT_REGION}"
        }
      }
    },
    {
      "label": "🔍 CDK: Diff",
      "type": "shell",
      "command": "cdk",
      "args": ["diff"],
      "group": "test",
      "options": {
        "cwd": "${workspaceFolder}/infrastructure",
        "env": {
          "CDK_DEFAULT_ACCOUNT": "${env:CDK_DEFAULT_ACCOUNT}",
          "CDK_DEFAULT_REGION": "${env:CDK_DEFAULT_REGION}"
        }
      }
    },
    {
      "label": "💥 CDK: Destroy",
      "type": "shell",
      "command": "cdk",
      "args": ["destroy", "--force"],
      "group": "build",
      "options": {
        "cwd": "${workspaceFolder}/infrastructure",
        "env": {
          "CDK_DEFAULT_ACCOUNT": "${env:CDK_DEFAULT_ACCOUNT}",
          "CDK_DEFAULT_REGION": "${env:CDK_DEFAULT_REGION}"
        }
      }
    },
    {
      "label": "🔨 Go: Build CDK",
      "type": "shell",
      "command": "go",
      "args": ["build", "-o", "infrastructure"],
      "group": "build",
      "options": {
        "cwd": "${workspaceFolder}/infrastructure"
      },
      "problemMatcher": ["$go"]
    },
    {
      "label": "🧪 Go: Test CDK",
      "type": "shell",
      "command": "go",
      "args": ["test", "-v"],
      "group": "test",
      "options": {
        "cwd": "${workspaceFolder}/infrastructure"
      },
      "problemMatcher": ["$go"]
    },
    {
      "label": "🎯 CDK: Full Deploy",
      "dependsOrder": "sequence",
      "dependsOn": [
        "🔨 Go: Build CDK",
        "🔧 CDK: Synth",
        "🚀 CDK: Deploy"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ]
}
```

## 🏗️ Project Structure

```
infrastructure/
├── cdk.json                 # CDK configuration
├── go.mod                   # Go module definition
├── go.sum                   # Go dependencies lock
├── infrastructure.go        # Main CDK stack definition
├── infrastructure_test.go   # Unit tests
└── README.md               # This file
```

## 🌍 Environment Configuration

The stack is configured for:
- **Account**: Specified via `CDK_DEFAULT_ACCOUNT` environment variable
- **Region**: `eu-south-1` (Milan) - can be changed via `CDK_DEFAULT_REGION`

## 📚 Useful Resources

- [AWS CDK Go Developer Guide](https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-go.html)
- [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/go/)
- [CDK Patterns](https://cdkpatterns.com/)
