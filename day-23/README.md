# Lesson 23: AWS CodePipeline – CI/CD Orchestration (Exam‑Ready)

## What You Will Learn
- Build an end‑to‑end CI/CD pipeline using CodePipeline
- Use CodeCommit (source) and CodeBuild (build/test)
- Deploy to EC2 (via CodeDeploy) or ECS/EKS (high level)
- Apply exam‑relevant security, permissions, and troubleshooting

---

## Overview
AWS CodePipeline automates build, test, and deploy stages on every code change. It integrates with CodeCommit, CodeBuild, CodeDeploy, CloudFormation, ECS, and third‑party tools. Pipelines are event‑driven (webhooks/CloudWatch Events), versioned, and support manual approvals.

Key concepts:
- Stages → Actions (Source/Build/Test/Deploy/Approval)
- Artifacts passed between actions via S3 artifact store
- IAM roles per service: pipeline role, CodeBuild service role, CodeDeploy role

---

## Lab: Minimal CI/CD to EC2 via CodeDeploy

### Prerequisites
- A CodeCommit repo (see Lesson 12)
- An EC2 Auto Scaling group with CodeDeploy agent installed
- An IAM instance profile for EC2 that permits CodeDeploy and S3 artifact access

### 1) AppSpec + BuildSpec in Repo
Create these files at repo root:

`appspec.yml` (CodeDeploy EC2/Linux sample)
```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  AfterInstall:
    - location: scripts/restart.sh
      timeout: 180
      runas: root
```

`buildspec.yml` (CodeBuild)
```yaml
version: 0.2
phases:
  install:
    commands:
      - echo Installing dependencies
  build:
    commands:
      - echo Building application
artifacts:
  files:
    - '**/*'
  discard-paths: no
```

Add `scripts/restart.sh` and make it executable.

### 2) Create a CodeBuild Project
- Source: CodeCommit repo
- Environment: Managed image (Amazon Linux), runtime as needed
- Service role: Allow read from CodeCommit, write to S3 artifact bucket, CloudWatch Logs

### 3) Create a CodeDeploy Application & Deployment Group
- Compute platform: EC2/On‑Premises
- Deployment group: target ASG/instances with tag
- Service role: Allow access to Auto Scaling, EC2, and S3 artifacts

### 4) Create a CodePipeline
- Source: CodeCommit (branch `main`)
- Build: CodeBuild project from step 2
- Deploy: CodeDeploy application + deployment group
- Artifact store: S3 bucket (auto‑created or existing)

### 5) Test the Pipeline
- Commit and push; verify Source → Build → Deploy succeed
- Check CodeDeploy events and lifecycle hooks

---

## Exam Tips
- Use separate IAM roles: pipeline role vs CodeBuild vs CodeDeploy
- Manual approvals before production (actions of type Approval)
- Cross‑account deployments use artifact encryption (KMS) and role assumptions
- Cache builds in CodeBuild to speed up (S3 or local cache modes)
- Use parameterized templates and CloudFormation change sets for IaC stages

## Multi‑Region Deployment Patterns
- **StackSets**: Deploy CloudFormation templates across multiple regions
- **S3 Cross‑Region Replication**: Automatically sync artifacts between regions
- **Route 53 + CloudFront**: Single endpoint with global distribution and failover

## Blue/Green Deployment with CodePipeline
- Create separate Auto Scaling groups behind different ALBs
- Use Route 53 weighted routing to gradually shift traffic (10% → 50% → 100%)
- Both environments share the same RDS Multi‑AZ database during transition
- **Exam Pattern**: ALB + ASG + Route 53 weighted routing = gradual traffic shift

---

## Troubleshooting
- Pipeline stuck in Source: verify webhook and CodeCommit permissions
- Build fails: open CodeBuild logs in CloudWatch; verify buildspec path and runtime
- Deploy fails: check CodeDeploy agent status on instances and `appspec.yml` hooks
- Artifact not found: ensure build artifacts include deployable files and correct paths

---

## Cleanup
- Disable or delete the pipeline
- Delete CodeBuild project and its CloudWatch log groups if not needed
- Remove CodeDeploy application/deployment group
- Delete S3 artifact objects to reduce cost
