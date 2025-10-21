# Lesson 26: AWS Systems Manager – Patch Management & Automation (Exam‑Ready)

## What You Will Learn
- Use Systems Manager Patch Manager for automated security patching
- Configure maintenance windows for zero‑downtime updates
- Set up patch baselines and compliance monitoring
- Integrate with CloudWatch for monitoring and alerting

---

## Overview
AWS Systems Manager Patch Manager automates the process of patching managed instances (EC2, on‑premises). It supports patch baselines, maintenance windows, and compliance reporting—critical for DevOps security policies.

Key components:
- **Patch Baselines**: Define which patches to install (security, critical, all)
- **Maintenance Windows**: Schedule patching during low‑impact periods
- **Patch Groups**: Organize instances for different patching strategies
- **Compliance**: Monitor and report patch status across instances

---

## Hands‑On: Automated Security Patching

### 1) Create a Patch Baseline
- Systems Manager → Patch Manager → Patch baselines
- Create baseline → **AWS‑DefaultPatchBaseline** (or custom)
- Configure:
  - **Security updates**: Auto‑approve after 7 days
  - **Critical updates**: Auto‑approve after 1 day
  - **Other updates**: Manual approval

### 2) Create a Patch Group
- Systems Manager → Patch Manager → Patch groups
- Name: `Production-Servers`
- Attach patch baseline to the group

### 3) Tag EC2 Instances
```bash
# Tag instances for patch group
aws ec2 create-tags --resources i-1234567890abcdef0 --tags Key=PatchGroup,Value=Production-Servers
```

### 4) Create Maintenance Window
- Systems Manager → Maintenance Windows
- Schedule: `cron(0 2 ? * SUN *)` (Sundays at 2 AM)
- Duration: 4 hours
- Add task: **AWS‑RunPatchBaseline** with patch group target

### 5) Monitor Compliance
- Systems Manager → Compliance
- View patch compliance status across instances
- Set up CloudWatch alarms for non‑compliant instances

---

## Exam Patterns

### Automated Patching (Question 5)
- **Patch Manager + Maintenance Windows** = automated compliance
- **Patch baselines** define what gets installed
- **Patch groups** organize instances by environment
- **CloudWatch alarms** monitor compliance status

### Integration with CloudWatch
```json
{
  "AlarmName": "PatchCompliance-Alert",
  "MetricName": "PatchCompliance",
  "Namespace": "AWS/SSM-PatchManager",
  "Statistic": "Average",
  "Threshold": 100,
  "ComparisonOperator": "LessThanThreshold"
}
```

---

## Best Practices
- **Separate baselines** for production vs development
- **Maintenance windows** during low‑traffic periods
- **Patch groups** for different update schedules
- **Compliance monitoring** with CloudWatch alarms
- **Rollback plans** for critical systems

---

## Troubleshooting
- **Instances not appearing**: Check SSM agent status and IAM roles
- **Patches not installing**: Verify maintenance window targets and permissions
- **Compliance issues**: Check patch baseline rules and instance tags

---

## Cleanup
- Remove maintenance window tasks
- Delete unused patch baselines and groups
- Remove CloudWatch alarms if created for testing

---

## Exam Tips
- **Patch Manager** is the AWS‑native solution for automated patching
- **Maintenance windows** provide controlled deployment timing
- **Patch baselines** define approval rules (security vs all updates)
- **Compliance reporting** integrates with CloudWatch for monitoring
- **IAM roles** required for SSM agent communication
