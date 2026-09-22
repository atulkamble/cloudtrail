# AWS CloudTrail Monitoring

## 1. Introduction to AWS CloudTrail

**AWS CloudTrail** is an AWS auditing and governance service that records activity performed in an AWS account.

Think of CloudTrail as:

> **CloudTrail = Who did what, when, from where, and on which AWS resource?**

CloudTrail records activity performed through the:

* AWS Management Console
* AWS CLI
* AWS SDKs
* AWS APIs
* AWS services acting on your behalf

### Example

Suppose an IAM user deletes an EC2 instance.

CloudTrail can help identify:

```text
Who?        Atul
What?       TerminateInstances
When?       22-Sep-2026 10:30 AM
From where? 103.x.x.x
Which?      EC2 instance i-0123456789
Region?     ap-south-1
```

---

# 2. Points to Remember

### CloudTrail Key Points

1. CloudTrail is primarily used for **auditing API and account activity**.
2. It records AWS API calls and related events.
3. CloudTrail Event History provides the most recent **90 days of management events** in an AWS Region.
4. Event History is available without creating a trail.
5. Create a **Trail** when you want ongoing delivery of CloudTrail events.
6. Trail logs can be delivered to an **Amazon S3 bucket**.
7. CloudTrail can integrate with **CloudWatch Logs** for monitoring and alerting.
8. CloudTrail records information such as:

```text
User / Role
Event Name
AWS Service
Event Time
Source IP
AWS Region
Resource
Request Parameters
Response
User Agent
```

9. CloudTrail is useful for:

```text
Security auditing
Compliance
Troubleshooting
Incident investigation
Tracking IAM activity
Tracking resource changes
Detecting suspicious API activity
```

10. CloudTrail answers:

```text
WHO performed the action?
WHAT action was performed?
WHEN was it performed?
WHERE did the request originate?
WHICH AWS resource was affected?
```

---

# 3. CloudTrail Architecture Diagram

```text
                    AWS ACCOUNT
                         |
        +----------------+----------------+
        |                |                |
     IAM User         IAM Role        Root User
        |                |                |
        +----------------+----------------+
                         |
                         v
               AWS API / Console / CLI
                         |
                         v
              +---------------------+
              |     AWS Services    |
              |---------------------|
              | EC2                 |
              | S3                  |
              | IAM                 |
              | RDS                 |
              | Lambda              |
              | VPC                 |
              +----------+----------+
                         |
                    API Activity
                         |
                         v
              +---------------------+
              |   AWS CloudTrail    |
              +----------+----------+
                         |
             +-----------+-----------+
             |                       |
             v                       v
      +-------------+         +---------------+
      | Amazon S3   |         | CloudWatch    |
      | Log Storage |         | Logs          |
      +-------------+         +-------+-------+
                                      |
                                      v
                              Metric Filter/Alarm
                                      |
                                      v
                                  Amazon SNS
                                      |
                                      v
                              Email / Notification
```

### Flow

```text
User / Role
     |
     v
AWS API Call
     |
     v
AWS Service
     |
     v
CloudTrail
     |
     +------> Event History
     |
     +------> S3 Bucket
     |
     +------> CloudWatch Logs
                    |
                    v
                  Alarm
                    |
                    v
                   SNS
```

---

# 4. Types of CloudTrail Events

CloudTrail commonly works with three important event categories.

### Management Events

Operations performed on AWS resources and account configuration.

Examples:

```text
RunInstances
TerminateInstances
CreateUser
DeleteUser
CreateBucket
DeleteBucket
CreateSecurityGroup
StopInstances
```

These are commonly called **control-plane operations**.

### Data Events

Operations performed on or within resources.

Examples include:

```text
S3 GetObject
S3 PutObject
Lambda Invoke
DynamoDB item-level activity
```

Data events can generate a very large number of records, so enable them only where required and consider the associated cost.

### Insights Events

CloudTrail Insights can identify unusual patterns in supported API activity, such as unexpected changes in API call or error rates.

---

# 5. Basic CloudTrail Practical — Console

## Step 1 — Open CloudTrail

```text
AWS Management Console
        ↓
Search "CloudTrail"
        ↓
Open CloudTrail
```

## Step 2 — Check Event History

Navigate to:

```text
CloudTrail
    ↓
Event history
```

You can filter events using fields such as:

```text
Event name
Event source
Resource name
Resource type
Username
Access key
Event ID
```

---

# 6. Generate Activity for Testing

For example, create an S3 bucket or EC2 instance.

CLI example:

```bash
aws s3 mb s3://my-cloudtrail-demo-123456
```

Check:

```bash
aws s3 ls
```

Delete it later:

```bash
aws s3 rb s3://my-cloudtrail-demo-123456
```

Now return to:

```text
CloudTrail
    ↓
Event history
```

Search for:

```text
CreateBucket
```

or:

```text
DeleteBucket
```

---

# 7. Monitor User Activity

Suppose an IAM user runs:

```bash
aws ec2 describe-instances
```

Or stops an instance:

```bash
aws ec2 stop-instances \
--instance-ids i-0123456789abcdef0
```

CloudTrail can record information about that API request.

Typical event:

```json
{
  "eventSource": "ec2.amazonaws.com",
  "eventName": "StopInstances",
  "awsRegion": "ap-south-1",
  "sourceIPAddress": "103.x.x.x"
}
```

The complete event contains additional identity, request, resource, and response information.

---

# 8. Create a CloudTrail Trail

Go to:

```text
CloudTrail
    ↓
Trails
    ↓
Create trail
```

Enter:

```text
Trail name:
my-cloudtrail

Storage location:
Create new S3 bucket

Bucket:
aws-cloudtrail-logs-example
```

Select the required event types.

For a basic lab:

```text
Management events
    Read  → Enable
    Write → Enable
```

Then:

```text
Create trail
```

Architecture:

```text
AWS API Activity
       |
       v
   CloudTrail
       |
       v
    Trail
       |
       v
   S3 Bucket
       |
       v
Long-term Audit Logs
```

---

# 9. CloudTrail AWS CLI Commands

Check configured identity first:

```bash
aws sts get-caller-identity
```

List trails:

```bash
aws cloudtrail describe-trails
```

Check trail status:

```bash
aws cloudtrail get-trail-status \
--name my-cloudtrail
```

Start logging:

```bash
aws cloudtrail start-logging \
--name my-cloudtrail
```

Stop logging:

```bash
aws cloudtrail stop-logging \
--name my-cloudtrail
```

---

# 10. Search CloudTrail Events Using CLI

Show recent events:

```bash
aws cloudtrail lookup-events
```

Search by username:

```bash
aws cloudtrail lookup-events \
--lookup-attributes AttributeKey=Username,AttributeValue=Atul
```

Search by event name:

```bash
aws cloudtrail lookup-events \
--lookup-attributes AttributeKey=EventName,AttributeValue=CreateBucket
```

Search for EC2 termination:

```bash
aws cloudtrail lookup-events \
--lookup-attributes AttributeKey=EventName,AttributeValue=TerminateInstances
```

---

# 11. Example Security Investigation

Suppose somebody terminates an EC2 instance.

```text
EC2 Instance
     |
     X  Terminated
     |
     v
Administrator investigates
     |
     v
CloudTrail
     |
     v
Search:
TerminateInstances
     |
     v
+----------------------------+
| User Identity              |
| Event Time                 |
| Source IP                  |
| Instance ID                |
| Region                     |
| Request Parameters         |
+----------------------------+
```

Command:

```bash
aws cloudtrail lookup-events \
--lookup-attributes AttributeKey=EventName,AttributeValue=TerminateInstances
```

CloudTrail helps answer:

```text
Who terminated it?
When?
Which credentials/role were used?
Which instance?
Which region?
What was the source IP?
```

---

# 12. CloudTrail + CloudWatch Monitoring Architecture

CloudTrail and CloudWatch can work together.

```text
              AWS USER
                  |
                  v
              AWS API
                  |
                  v
            AWS SERVICE
                  |
                  v
            CLOUDTRAIL
             /      \
            /        \
           v          v
      S3 Bucket   CloudWatch Logs
                       |
                       v
                  Metric Filter
                       |
                       v
                 CloudWatch Alarm
                       |
                       v
                    SNS
                       |
                       v
                 Administrator
```

Example objective:

```text
Someone changes/deletes a resource
             ↓
CloudTrail records API activity
             ↓
CloudWatch Logs receives logs
             ↓
Monitoring rule detects activity
             ↓
Alarm / notification
             ↓
Administrator investigates
```

---

# 13. CloudWatch vs CloudTrail

| Feature                 | CloudWatch                             | CloudTrail                          |
| ----------------------- | -------------------------------------- | ----------------------------------- |
| Main purpose            | Monitoring & observability             | Auditing & governance               |
| Focus                   | Performance/health/logs                | AWS API/account activity            |
| Records API activity    | Not its primary purpose                | Yes                                 |
| Metrics                 | Yes                                    | Not primary function                |
| Application logs        | Yes                                    | No                                  |
| Alarms                  | Yes                                    | Not directly like CloudWatch alarms |
| User activity auditing  | Limited/not primary                    | Yes                                 |
| Source IP investigation | Not primary                            | Yes                                 |
| Security investigation  | Useful                                 | Very important                      |
| S3 log delivery         | Possible depending on service/workflow | Trails can deliver logs to S3       |
| Typical question        | "What is happening?"                   | "Who did it?"                       |

## Easy Way to Remember

```text
CloudWatch
    =
WATCH the infrastructure/application

CPU
Memory*
Network
Application Logs
Errors
Metrics
Alarms
Dashboards
```

`*` EC2 memory utilization requires an agent or custom metric; it is not a default EC2 metric.

```text
CloudTrail
    =
TRAIL of AWS activity

Who?
What?
When?
Where?
Which resource?
```

---

# 14. Practical Example

### Problem

```text
EC2 instance was terminated.
```

### CloudWatch tells you:

```text
Instance metrics
Performance information
Monitoring/alarm history
Relevant logs if configured
```

### CloudTrail helps tell you:

```text
TerminateInstances API was called
User/Role = ...
Time = ...
Source IP = ...
Instance ID = ...
Region = ...
```

Therefore:

```text
CloudWatch → Monitoring

CloudTrail → Auditing
```

---

# 15. Important Commands for Students

```bash
# Check current AWS identity
aws sts get-caller-identity

# List CloudTrail trails
aws cloudtrail describe-trails

# Check trail status
aws cloudtrail get-trail-status \
--name my-cloudtrail

# Start logging
aws cloudtrail start-logging \
--name my-cloudtrail

# Stop logging
aws cloudtrail stop-logging \
--name my-cloudtrail

# Search recent events
aws cloudtrail lookup-events

# Search CreateBucket activity
aws cloudtrail lookup-events \
--lookup-attributes AttributeKey=EventName,AttributeValue=CreateBucket

# Search TerminateInstances activity
aws cloudtrail lookup-events \
--lookup-attributes AttributeKey=EventName,AttributeValue=TerminateInstances
```

# 16. Final Points to Remember

```text
CloudTrail = Audit AWS activity

CloudWatch = Monitor AWS resources/applications

CloudTrail records:
WHO
WHAT
WHEN
WHERE
WHICH RESOURCE

Event History:
Recent 90 days of management events

Trail:
Used for ongoing event delivery

S3:
Long-term CloudTrail log storage

CloudWatch Logs:
Monitoring and alerting integration

Management Events:
Control-plane operations

Data Events:
Resource-level/data-plane operations

CloudTrail Insights:
Unusual API activity patterns
```

### One-Line Exam Memory

> **CloudWatch watches performance and operational data; CloudTrail tracks AWS API and account activity.**

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

## Get Bucket Policy Details 
```
aws s3api get-bucket-policy \
  --bucket atul-cloudtrail-logs
```

## Apply policy to bucket
```
aws s3api put-bucket-policy \
  --bucket atul-cloudtrail-logs \
  --policy file://cloudtrail-policy.json
```

## Check Bucket Region 
```
aws s3api get-bucket-location \
  --bucket atul-cloudtrail-logs
```

## Check your CLI region:
```
aws configure get region
```

## Check Bucket Ownership Controls

```
aws s3api get-bucket-ownership-controls \
  --bucket atul-cloudtrail-logs
```  

## check Public Access Block settings
```
aws s3api get-public-access-block \
  --bucket atul-cloudtrail-logs
```

## Get your account ID
```
aws sts get-caller-identity \
  --query Account \
  --output text
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
## Perform AWS actions
```
aws ec2 describe-instances
aws s3 ls
aws iam list-users
```

## Verify Events 
```
aws cloudtrail lookup-events \
    --max-results 5
```
## Verify log files 
```
aws s3 ls s3://atul-cloudtrail-logs --recursive
```

## Verify Trail

```bash
aws cloudtrail describe-trails
```

## Create an S3 Event
```
echo "CloudTrail Test" > test.txt

aws s3 cp test.txt s3://atul-cloudtrail-logs/
```

## Verify: S3 Event 
```
 aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventSource,AttributeValue=s3.amazonaws.com \
  --max-results 10
```

## Enable S3 Data Events 
```
aws cloudtrail put-event-selectors \
  --trail-name mytrail \
  --event-selectors '[
    {
      "ReadWriteType":"All",
      "IncludeManagementEvents":true,
      "DataResources":[
        {
          "Type":"AWS::S3::Object",
          "Values":["arn:aws:s3:::atul-cloudtrail-logs/"]
        }
      ]
    }
  ]'
  ```

## Verify 
```
aws cloudtrail get-event-selectors \
  --trail-name mytrail
```  

## Upload another file 
```
echo "Hello" > file2.txt

aws s3 cp file2.txt s3://atul-cloudtrail-logs/
```

## Search Again
```
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventSource,AttributeValue=s3.amazonaws.com \
  --max-results 20
```

## List Newest Logs 
```
aws s3 ls s3://atul-cloudtrail-logs/AWSLogs/535002879962/CloudTrail/us-east-1/ --recursive
```

## List Latest Files 
```
aws s3 ls s3://atul-cloudtrail-logs/AWSLogs/535002879962/CloudTrail/us-east-1/ --recursive | tail -1
```
## Download Latest File 
```
aws s3 cp \
s3://atul-cloudtrail-logs/AWSLogs/535002879962/CloudTrail/us-east-1/2026/06/17/535002879962_CloudTrail_us-east-1_20260617T2150Z_BzyOhgsC6hiTSJjJ.json.gz \
.

aws s3 cp \
s3://atul-cloudtrail-logs/AWSLogs/535002879962/CloudTrail/us-east-1/2026/06/17/535002879962_CloudTrail_us-east-1_20260617T2150Z_xZjX2dDsOUoPTlZV.json.gz \
.
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

## Delete 
```
aws cloudtrail stop-logging \\n  --name mytrail
aws cloudtrail delete-trail \\n  --name mytrail
aws cloudtrail list-trails
aws s3 ls
aws s3 rb s3://atul-cloudtrail-logs
aws s3 ls s3://atul-cloudtrail-logs --recursive
aws s3 rm s3://atul-cloudtrail-logs --recursive
aws s3 rb s3://atul-cloudtrail-logs
aws s3 ls
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
