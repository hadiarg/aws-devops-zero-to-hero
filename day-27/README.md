# Lesson 27: CloudWatch Monitoring & SNS Integration (Exam‑Ready)

## What You Will Learn
- Set up CloudWatch alarms for system metrics
- Configure SNS topics for notifications
- Use EventBridge for event‑driven automation
- Implement automated remediation with Lambda

---

## Overview
CloudWatch provides monitoring, logging, and alerting for AWS resources. Combined with SNS and EventBridge, it enables automated responses to system events—essential for DevOps automation and incident response.

Key components:
- **CloudWatch Metrics**: System and custom metrics from AWS services
- **CloudWatch Alarms**: Threshold‑based alerting
- **SNS Topics**: Notification delivery to multiple endpoints
- **EventBridge**: Event‑driven automation and integration

---

## Hands‑On: Automated Memory Monitoring & Remediation

### 1) Create CloudWatch Alarm
- CloudWatch → Alarms → Create alarm
- Metric: `AWS/EC2` → `MemoryUtilization`
- Threshold: `> 80%` for 2 consecutive periods
- Actions: Send notification to SNS topic

### 2) Create SNS Topic
- SNS → Topics → Create topic
- Name: `Memory-Alerts`
- Add subscribers: Email, SMS, or Lambda function

### 3) Automated Remediation with Lambda
```python
import boto3
import json

def lambda_handler(event, context):
    # Parse SNS message
    message = json.loads(event['Records'][0]['Sns']['Message'])
    
    # Get instance ID from alarm
    instance_id = message['Trigger']['Dimensions'][0]['value']
    
    # Restart application using Systems Manager
    ssm = boto3.client('ssm')
    response = ssm.send_command(
        InstanceIds=[instance_id],
        DocumentName='AWS-RunShellScript',
        Parameters={
            'commands': ['sudo systemctl restart your-app']
        }
    )
    
    return {'statusCode': 200}
```

### 4) EventBridge Integration
- EventBridge → Rules → Create rule
- Event pattern: CloudWatch alarm state change
- Target: Lambda function for automated response

---

## Exam Patterns

### Memory Leak Remediation (Question 10)
- **CloudWatch alarm** on memory utilization metric
- **SNS topic** for notification delivery
- **Lambda function** for automated remediation
- **Systems Manager** for safe command execution

### Multi‑Consumer Notifications (Question 6)
- **SNS fanout** pattern for multiple consumers
- **S3 event notifications** → SNS topic → Lambda + SQS
- **EventBridge** for complex event routing

### Security Monitoring (Question 7)
- **GuardDuty** findings → EventBridge → Lambda
- **Custom user lookup** via API integration
- **External notification** platform integration

---

## Advanced Patterns

### SNS Fanout Pattern
```json
{
  "S3Event": {
    "Records": [{
      "s3": {
        "bucket": {"name": "my-bucket"},
        "object": {"key": "path/to/file"}
      }
    }]
  }
}
```

### EventBridge Rules
```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "type": ["Recon:EC2/PortProbeUnprotectedPort"]
  }
}
```

---

## Best Practices
- **Separate SNS topics** for different alert types
- **Lambda functions** for automated remediation
- **EventBridge** for complex event routing
- **CloudWatch dashboards** for visualization
- **Log groups** for centralized logging

---

## Troubleshooting
- **Alarms not triggering**: Check metric data and threshold settings
- **SNS delivery failures**: Verify endpoint permissions and format
- **Lambda timeouts**: Optimize function execution time
- **EventBridge rules**: Validate event patterns and targets

---

## Cleanup
- Delete CloudWatch alarms
- Remove SNS topics and subscriptions
- Delete Lambda functions and EventBridge rules
- Clean up CloudWatch log groups

---

## Exam Tips
- **CloudWatch alarms** trigger SNS notifications
- **SNS fanout** enables multiple consumers from single event
- **EventBridge** provides event‑driven automation
- **Lambda + Systems Manager** = automated remediation
- **GuardDuty** integrates with EventBridge for security automation
