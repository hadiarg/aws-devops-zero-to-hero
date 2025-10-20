# Lesson 12: AWS CodeCommit

## What You Will Learn
- Set up and manage Git repositories using AWS CodeCommit
- Configure secure access via IAM (HTTPS Git credentials or SSH)
- Use branches and pull requests for collaboration
- Integrate with CodeBuild and CodePipeline

---

## Overview
AWS CodeCommit is a fully managed, secure, Git-based source control service. It integrates with IAM for fine‑grained access, encrypts data at rest (KMS) and in transit (TLS), and works with standard Git tools.

Key capabilities:
- Unlimited private repos, standard Git workflows
- Pull requests, code reviews, branch protections
- Native integrations with CodeBuild/CodeDeploy/CodePipeline

---

## Quick Start

### 1) Create a Repository
- Open the CodeCommit console
- Create repository → name it (for example, `demo-repo`)

### 2) Grant Access (IAM)
Attach a minimal policy to a user/role (replace placeholders):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CodeCommitGitAccess",
      "Effect": "Allow",
      "Action": [
        "codecommit:GitPull",
        "codecommit:GitPush",
        "codecommit:GetRepository",
        "codecommit:ListBranches"
      ],
      "Resource": "arn:aws:codecommit:REGION:ACCOUNT_ID:REPO_NAME"
    }
  ]
}
```

### 3) Choose Auth
- HTTPS: Generate “Git credentials for CodeCommit” for your IAM user (Console → IAM → Users → Security credentials)
- SSH: Upload your SSH public key to IAM; use the SSH clone URL

### 4) Clone and Push
```bash
# Configure Windows credential manager (optional but convenient)
git config --global credential.helper manager-core

# Clone (HTTPS example)
git clone https://git-codecommit.REGION.amazonaws.com/v1/repos/REPO_NAME

cd REPO_NAME
echo "# Hello CodeCommit" > README.md
git add .
git commit -m "chore: initial commit"
git push -u origin main
```

---

## Branching & Pull Requests
```bash
# Create a feature branch
git checkout -b feature/add-ci
# Commit and push
git commit -am "feat: add buildspec"
git push -u origin feature/add-ci
```
- Open a Pull Request from `feature/add-ci` → `main`
- Add reviewers and enable approvals in repo settings

---

## Integrations

### CodeBuild
- Create a project using your CodeCommit repo
- Add `buildspec.yml` at repo root
- Optionally enable webhooks to build on push

### CodePipeline
- Source: CodeCommit → Build: CodeBuild → Deploy: (CodeDeploy/CloudFormation/ECS/EKS)

---

## Read‑Only Policy Example
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadOnlyRepoAccess",
      "Effect": "Allow",
      "Action": [
        "codecommit:GitPull",
        "codecommit:GetRepository",
        "codecommit:ListBranches",
        "codecommit:Get*",
        "codecommit:List*"
      ],
      "Resource": "arn:aws:codecommit:REGION:ACCOUNT_ID:REPO_NAME"
    }
  ]
}
```

---

## Troubleshooting
- HTTPS auth fails: Regenerate CodeCommit Git credentials; ensure credential manager stores updated values
- SSH denied: Verify IAM SSH public key, correct SSH URL and IAM-provided username (`APKA...`)
- Push rejected to `main`: Check branch protection and required approvals

---

## Clean Up
- Delete test repos when done
- Remove access keys/SSH keys not needed
- Delete CodeBuild/CodePipeline resources created for demos
