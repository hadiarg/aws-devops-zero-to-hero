# Lesson 12: AWS CodeCommit

## What You Will Learn
- Set up and manage Git repositories using **AWS CodeCommit**, a fully managed source control service.
- Configure secure access to repositories using **IAM policies** and **authentication methods** (HTTPS with Git credentials or SSH).
- Collaborate with team members using standard Git workflows (clone, commit, push, pull, branches, pull requests).
- Implement **access control** and **collaboration workflows** for team-based development projects.

---

## Overview
AWS CodeCommit is a secure, scalable, and fully managed **Git-based source control service** hosted by AWS. It eliminates the need to operate your own source control system and integrates natively with AWS Identity and Access Management (IAM) for fine-grained permissions. CodeCommit supports:
- Unlimited private repositories
- Full Git functionality (branches, tags, merges, pull requests)
- Integration with AWS CodeBuild, CodePipeline, and third-party tools
- Encryption at rest (AWS KMS) and in transit (HTTPS/SSH)

This service is ideal for DevOps teams building secure, auditable, and compliant CI/CD pipelines on AWS.

---

## Key Features

### Fully Managed Git
- No servers to manage or patch
- Automatically scales with your team and codebase size
- Supports standard Git commands and tools (Git CLI, VS Code, SourceTree, etc.)

### Security & Compliance
- Repositories are **private by default**
- Access controlled via **IAM policies** (not repository-level users)
- Supports **HTTPS (Git credentials)** and **SSH authentication**
- All data encrypted at rest using **AWS KMS** and in transit via TLS

### Collaboration Tools
- Create and manage **branches** and **tags**
- Use **pull requests** for code review and approval workflows
- Track commit history, diffs, and authorship

---

## Setting Up a CodeCommit Repository

### 1. Create a Repository
- Open the [AWS CodeCommit console](https://console.aws.amazon.com/codesuite/codecommit/home)
- Choose **Create repository**
- Enter a **repository name** and (optional) description
- Choose **Create**

### 2. Configure IAM Access
Attach an IAM policy to users/roles that grants CodeCommit permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codecommit:GitPull",
        "codecommit:GitPush"
      ],
      "Resource": "arn:aws:codecommit:region:account-id:repo-name"
    }
  ]
}