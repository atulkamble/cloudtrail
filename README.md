# CloudTrail

## What is AWS CloudTrail?

AWS CloudTrail is a service that records and monitors all API activity and user actions performed in your AWS account.

It helps answer questions like:

* Who created an EC2 instance?
* Who deleted an S3 bucket?
* When was an IAM user modified?
* Which IP address accessed AWS resources?
* What API calls were made?

### Real-World Example

Suppose someone accidentally deletes an EC2 instance.

Without CloudTrail:

* No idea who deleted it.

With CloudTrail:

* User name
* Time of deletion
* Source IP
* API call used
* AWS Region

All information is available in CloudTrail logs.

---

## CloudTrail Architecture

![Image](https://images.openai.com/static-rsc-4/JheXk-ZCuP7JU8R14Z1WGbz4RRfA9g48oFtPPFotFOgju4WDtVvaJTwnR-ege9Xo5xKE62AncAcBLS7m2Z4j8UR-ehjJKIxHSYrYhW3nwBAYWmH89qlytHoYWkcyyI9PVFRQLFblhV94jrIQo7ZSV8sSuxHPLamWTCr1YAMwaPtbGo96JC3JztV61kJ8rw4e?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/1EfqVt2A6JjB16azwh9MXnrT3e0X-ijBSSVdN3PTsK8L-nuMpgig6SQF8ccR85pBtQF-3E5gpFmDe0dkeBocaPzR9JwLQsL0yY0B4_m64VvWG3VIsysDvcYefRf0N3Nej4VgGWbKwa3RYlTrmenkySOAu2ogm8u2r8wvd8J6tpyCI3lsjgpilg-i_BfDD9ih?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/7wpdmCl1UzCOiigco7s8fA1d0lQtoK0T6DRuZMqop7e7hfhOyDbK4LkR4ppZg8TIqOYX-bqsvJqY6X3Z4_qAV3SkAAPW5Z4ocQZAmMhTmBJU4wH0BjU6sJl-9I9vYY0wn6rLvIn8eFmFy1uV5RUQXhraJIxLpPDqkXDoN4JpvlOURrMVrteSEsFkFg04QKd2?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/CP4NL2l4ZAwaTFwBA-8gAssM_N3UhDgmaG8pPBxKjzq5VlgnUZQsRivfHO4c0ewG9grOgcH07cv5lPvOOJNuRwPFQZxlknvmObPdXZRYwQYu-sZcN8BLTLDndTVCpKFy4FNFvlj5-PThta3e1nandxCcb0lA28QspGmDirEPb9dtT5BXdLn3dfUzcD6QrFiL?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/t8XNDaFBgfRzyw-ld2sW5YqDpMRA7mqrXr6ZkfgKJbCpo4Gr-ZS-acnqgUCWEKn0w9R6BdBN6TjY6F4mTK3MAwCsmtzG_HEmQXMiXt2URq-MTa7kvEiqBDwYSgzfkCW5oTARWtoddwUbTHAvg9soUyu9KfKUjdYTf2n-iyNs1d_AP5ohvg2BfZDL6CaII4s-?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/iWYjS_2VCH59fTFrQkmPAhnqD3C4yuT2kYr-TZMSe38gjrOCLkUahrNC6ulDX--NXvJ_WKyePgokngHsC5xRlDYEYJINC3t5D2b6ZS5np0xehND6K8cbMpGFhE0TNOVbO4sJcWlqQeA1aBLw_xTiJuNQuUYnB5dMOdPpU8xYzXHxohcbv1py-WBOj5pMgj6f?purpose=fullsize)

```text
User/API Call
      |
      v
AWS CloudTrail
      |
      +---- Event History
      |
      +---- S3 Bucket
      |
      +---- CloudWatch Logs
      |
      +---- EventBridge
```

---

# Types of Events

## 1. Management Events

Tracks AWS resource management operations.

Examples:

* RunInstances
* StopInstances
* CreateBucket
* DeleteBucket
* CreateUser

Example:

```bash
aws ec2 run-instances
```

CloudTrail records:

```text
Event Name: RunInstances
User: atul
Service: EC2
```

---

## 2. Data Events

Tracks resource-level operations.

Examples:

### S3

```text
GetObject
PutObject
DeleteObject
```

### Lambda

```text
Invoke Function
```

---

## 3. Insights Events

Detects unusual activities.

Examples:

* Sudden spike in API calls
* Multiple failed logins
* Unusual resource creation

---

# CloudTrail Event History

View last 90 days of management events.

AWS Console:

```text
CloudTrail
 → Event History
```

No setup required.

---

# Create CloudTrail Using AWS CLI

## Create S3 Bucket

```bash
aws s3 mb s3://atul-cloudtrail-logs
```

---

## Create Trail

```bash
aws cloudtrail create-trail \
  --name mytrail \
  --s3-bucket-name atul-cloudtrail-logs
```

---

## Start Logging

```bash
aws cloudtrail start-logging \
  --name mytrail
```

---

## Verify Trail

```bash
aws cloudtrail describe-trails
```

---

## Check Logging Status

```bash
aws cloudtrail get-trail-status \
  --name mytrail
```

---

## Stop Logging

```bash
aws cloudtrail stop-logging \
  --name mytrail
```

---

## Delete Trail

```bash
aws cloudtrail delete-trail \
  --name mytrail
```

---

# Useful CloudTrail Commands

## List Trails

```bash
aws cloudtrail list-trails
```

---

## Show Trail Details

```bash
aws cloudtrail get-trail \
  --name mytrail
```

---

## Lookup Events

### Last 10 Events

```bash
aws cloudtrail lookup-events \
  --max-results 10
```

---

### Find EC2 Events

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventSource,AttributeValue=ec2.amazonaws.com
```

---

### Find S3 Events

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventSource,AttributeValue=s3.amazonaws.com
```

---

### Find IAM Events

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventSource,AttributeValue=iam.amazonaws.com
```

---

### Search by Username

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=admin
```

---

## Enable Log File Validation

```bash
aws cloudtrail update-trail \
  --name mytrail \
  --enable-log-file-validation
```

---

## Enable Multi-Region Trail

```bash
aws cloudtrail update-trail \
  --name mytrail \
  --is-multi-region-trail
```

---

# Sample Event Record

```json
{
  "eventTime": "2026-06-17T10:00:00Z",
  "eventName": "RunInstances",
  "eventSource": "ec2.amazonaws.com",
  "sourceIPAddress": "1.2.3.4",
  "userIdentity": {
    "userName": "atul"
  }
}
```

---

# CloudTrail + CloudWatch

Send CloudTrail logs to CloudWatch.

Benefits:

* Real-time monitoring
* Metric filters
* Alerts
* Dashboards

Example:

```text
Unauthorized API Call
       |
       v
CloudWatch Alarm
       |
       v
SNS Email Notification
```

---

# CloudTrail + EventBridge

Trigger automation when events occur.

Example:

```text
Create EC2 Instance
       |
       v
CloudTrail
       |
       v
EventBridge
       |
       v
Lambda Function
```

Use Cases:

* Auto-tag resources
* Security alerts
* Compliance checks
* Auto-remediation

---

# CloudTrail Significance

## Security

Tracks all user actions.

Example:

```text
Who deleted EC2?
Who modified Security Group?
Who created IAM User?
```

---

## Compliance

Required for:

* PCI DSS
* HIPAA
* ISO 27001
* SOC 2

---

## Auditing

Maintains audit logs.

Example:

```text
Developer created bucket
Admin deleted bucket
```

---

## Troubleshooting

Find root cause of failures.

Example:

```text
EC2 terminated accidentally
CloudTrail identifies culprit
```

---

# Important Points to Remember

| Point           | Description                    |
| --------------- | ------------------------------ |
| Default History | Last 90 days Management Events |
| Global Service  | Available in all AWS Regions   |
| Storage         | Logs stored in S3              |
| Encryption      | Supports SSE-KMS               |
| Monitoring      | Integrates with CloudWatch     |
| Automation      | Integrates with EventBridge    |
| Insights        | Detects unusual API activity   |
| Multi-Region    | Recommended for production     |
| Validation      | Detects log tampering          |
| Security        | Essential for auditing         |

---

# Interview Questions

### What does CloudTrail do?

Records API calls and user activities in AWS.

---

### Difference Between CloudTrail and CloudWatch?

| CloudTrail        | CloudWatch              |
| ----------------- | ----------------------- |
| Records API Calls | Monitors Metrics & Logs |
| Auditing          | Monitoring              |
| Security          | Performance             |
| Who did what      | Resource Health         |

---

### Where are CloudTrail logs stored?

Amazon S3.

---

### How long does Event History remain?

90 days.

---

### Can CloudTrail track S3 object uploads?

Yes, by enabling Data Events.

---

# Hands-On Project 1: Security Monitoring

## Objective

Detect unauthorized IAM activities.

### Architecture

```text
IAM Activity
      |
      v
CloudTrail
      |
      v
CloudWatch Logs
      |
      v
Metric Filter
      |
      v
SNS Email
```

### Tasks

1. Create CloudTrail
2. Enable CloudWatch Logs
3. Create SNS Topic
4. Create Metric Filter

```text
ConsoleLoginFailure
UnauthorizedOperation
AccessDenied
```

5. Create Alarm
6. Receive Email Alert

### Outcome

Real-time security monitoring.

---

# Hands-On Project 2: EC2 Monitoring & Audit

## Scenario

Track all EC2 operations.

### Resources

* CloudTrail
* S3 Bucket
* EC2 Instance
* CloudWatch

### Perform

```bash
aws ec2 run-instances
aws ec2 stop-instances
aws ec2 terminate-instances
```

### Verify

```bash
aws cloudtrail lookup-events \
--lookup-attributes \
AttributeKey=EventSource,AttributeValue=ec2.amazonaws.com
```

### Expected Result

View:

* User
* Action
* Timestamp
* IP Address

---

# Hands-On Project 3: Compliance Logging

## Objective

Store immutable audit logs.

### Resources

* CloudTrail
* S3
* KMS
* Lifecycle Policies

### Features

* Multi-region Trail
* KMS Encryption
* Log Validation
* Long-term retention

### Use Cases

* Banking
* Healthcare
* Government
* Enterprise Audits

---

# Most Common Real-World Use Cases

| Industry         | Use Case                 |
| ---------------- | ------------------------ |
| Banking          | Compliance & Auditing    |
| Healthcare       | HIPAA Audits             |
| E-Commerce       | Security Monitoring      |
| DevOps           | Change Tracking          |
| Cloud Operations | Troubleshooting          |
| SOC Teams        | Threat Detection         |
| Enterprises      | Governance               |
| Startups         | User Activity Monitoring |

---

# Exam Tips (AWS SAA / SysOps / DevOps)

✅ CloudTrail = API Activity Tracking

✅ CloudWatch = Monitoring & Metrics

✅ Logs stored in S3

✅ Event History = 90 Days

✅ Enable Multi-Region Trail in Production

✅ Enable Log File Validation

✅ Use KMS Encryption

✅ CloudTrail + EventBridge = Automation

✅ CloudTrail + CloudWatch + SNS = Security Alerts

✅ Data Events required for S3 Object-level tracking

✅ One of the most important services for Security, Compliance, Auditing, and Troubleshooting in AWS.
