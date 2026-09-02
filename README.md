# AWS Security Incident Investigation with CloudTrail

## Project Overview

This project simulates a real-world security incident on an AWS-hosted web server and walks through the full incident response lifecycle: detection, investigation, remediation, and hardening.

A simple web application running on an EC2 instance was compromised — an attacker opened SSH access to the world, gained access to the instance, and defaced the website. Using AWS CloudTrail, I investigated the incident through three different methods (raw log analysis, the AWS CLI, and Amazon Athena SQL queries) to identify the responsible IAM identity, the source IP address, and the exact sequence of actions taken. I then remediated the incident and hardened the environment to prevent recurrence.

The main goal of this project was to gain hands-on experience with AWS-native incident response and to understand how CloudTrail logging supports real security investigations.

## Architecture

```
Attacker
   ↓ (opens SSH via unauthorized Security Group change)
EC2 Instance (Web Server)
   ↓
Website defaced
   ↓
CloudTrail (logs all API activity to S3)
   ↓
Investigation (grep / AWS CLI / Athena SQL)
   ↓
Remediation & Hardening
```

## AWS Services Used

* Amazon EC2
* AWS CloudTrail
* Amazon S3
* Amazon Athena
* AWS IAM

## How It Works

1. An EC2 instance is set up as a public-facing web server.
2. AWS CloudTrail is enabled to log all management (API) activity to an encrypted S3 bucket.
3. A simulated attacker modifies the instance's Security Group to open SSH (port 22) to the world, logs in, and defaces the website.
4. The incident is detected when the defacement and the unexpected Security Group rule are noticed.
5. CloudTrail logs are investigated using three different methods to identify the attacker's IAM identity, source IP, and actions.
6. The incident is remediated: unauthorized sessions and users are removed, SSH is re-hardened, the website is restored, and the Security Group rule is reverted.

## Implementation

### Step 1 – Resource Setup

An EC2 instance running a basic web server was launched to act as the target application. AWS CloudTrail was enabled with a dedicated, SSE-KMS-encrypted S3 bucket to capture management events (read and write API calls) across the account, providing the audit trail needed for the investigation.

### Step 2 – Configuration

The EC2 instance's Security Group was configured to restrict SSH access appropriately as a baseline. To simulate the incident, this rule was later modified to allow SSH from `0.0.0.0/0`, and password authentication was enabled on the instance — both common attacker techniques for establishing persistent access.

### Step 3 – Service Integration

CloudTrail was connected to an S3 bucket for durable log storage, and Amazon Athena was set up with an external table over the CloudTrail logs in S3, enabling SQL-based querying of API activity instead of manually parsing raw JSON logs.

### Step 4 – Testing

The investigation itself served as the test: I confirmed I could reliably identify the unauthorized `AuthorizeSecurityGroupIngress` event and trace it back to a specific IAM identity and source IP using three independent methods — `grep` against raw log files, the `aws cloudtrail lookup-events` CLI command, and Athena SQL queries. Cross-checking all three confirmed the findings were consistent.

## Screenshots

<img width="1910" height="1382" alt="image" src="https://github.com/user-attachments/assets/f327da4a-ed1f-44f4-9f7b-94af3224d9ea" />
<img width="1908" height="748" alt="image" src="https://github.com/user-attachments/assets/6c4b0fed-653a-4a8d-91c3-13a35190f88a" />

* AWS resources — EC2 instance and Security Group configuration



* CloudTrail trail configuration and S3 bucket
* The defaced website (before remediation)
* Raw log `grep` output showing the suspicious event
* AWS CLI `lookup-events` output
* Athena query and results identifying the attacker's identity and IP
* The IAM user cleanup and Security Group rule reversal
* The restored website (after remediation)

## Challenges and Troubleshooting

*Describe any problems you experienced while completing the project. For example:*

* What went wrong (e.g., difficulty parsing raw CloudTrail JSON, Athena table not returning results)
* How you investigated the problem
* How you fixed it
* What you learned from the issue

## Security Considerations

* **IAM roles and permissions** – A separate IAM user was used to simulate the attacker, keeping the "incident" isolated from the primary account credentials.
* **Security groups** – The investigation centered on identifying and reverting an over-permissive Security Group rule (SSH open to `0.0.0.0/0`).
* **Encryption** – The CloudTrail S3 bucket was encrypted using SSE-KMS to protect log integrity and confidentiality.
* **Least privilege access** – Remediation included disabling and deleting the compromised IAM user's access keys and re-hardening SSH to disable password authentication.

## What I Learned

Through this project, I developed practical experience with:

* Enabling and configuring AWS CloudTrail for security auditing
* Investigating security incidents using multiple methods: raw log analysis, the AWS CLI, and Amazon Athena SQL
* Identifying unauthorized IAM activity and tracing it to a source IP and identity
* Remediating a compromised EC2 instance and IAM user
* Applying AWS security hardening practices, including least-privilege IAM cleanup and SSH hardening

## Outcome

The project was successfully completed and tested. I was able to detect a simulated security incident, investigate it using three independent CloudTrail-based methods, identify the responsible identity and actions, and fully remediate and harden the environment against recurrence.

## Future Improvements

Possible future improvements could include:

* Adding Infrastructure as Code with Terraform or AWS CloudFormation to make the lab environment reproducible
* Setting up automated alerting (e.g., via Amazon SNS or CloudWatch Alarms) to detect similar Security Group changes in real time
* Adding AWS GuardDuty for automated threat detection alongside CloudTrail
* Using AWS Config to continuously monitor and auto-remediate non-compliant Security Group rules
