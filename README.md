# AWS Security Incident Investigation with CloudTrail & Athena

**LAB-187**

![AWS](https://img.shields.io/badge/AWS-CloudTrail%20%7C%20Athena%20%7C%20IAM%20%7C%20EC2-orange)
![Category](https://img.shields.io/badge/Category-Incident%20Response-blue)

## Overview

A hands-on lab simulating and investigating a security breach on a production-style web server hosted on EC2. AWS CloudTrail auditing was enabled, the attacker's actions were traced through raw log analysis, AWS CLI queries, and SQL-based querying in Amazon Athena, the compromised IAM identity was identified, and the attack vector was fully remediated — restoring both the affected service and account-level security posture.

## Scenario

A café's public-facing web server was defaced shortly after an unauthorized inbound SSH rule (port 22 open to `0.0.0.0/0`) was added to its security group. With CloudTrail freshly enabled, the incident became traceable. The objective: identify **who** made the change, **when**, **from where**, and **how** — then fully remediate the compromise.

## Objectives

- Enable and configure a CloudTrail trail to capture account activity
- Analyze CloudTrail logs via raw JSON inspection, Linux `grep`, AWS CLI `lookup-events`, and Amazon Athena SQL
- Import CloudTrail log data into Athena for structured querying
- Identify the compromised IAM identity, timestamp, source IP, and access method behind the breach
- Remediate the compromise at both the OS and AWS account level

## Tech Stack / Skills Demonstrated

| Area | Tools / Concepts |
|---|---|
| Cloud Auditing | AWS CloudTrail — trail configuration, log structure, S3-backed log storage |
| Log Analysis | Amazon Athena — external table creation over JSON logs, SQL (`SELECT`, `WHERE`, `LIKE`, `DISTINCT`, timestamp filtering) |
| CLI Tooling | AWS CLI — `cloudtrail lookup-events`, `ec2 describe-instances`, attribute-based filtering |
| Linux Administration | SSH, `grep`, `aureport`, `who`, process termination, `sshd_config` hardening, VI editor |
| Identity Security | IAM — identifying and removing compromised identities, deactivating access keys |
| Methodology | Incident response: detection → investigation → root cause → containment → remediation → hardening |

## Investigation Workflow

1. **Baseline observation** — confirmed normal application behavior and reviewed existing security group rules
2. **Enabled auditing** — created a CloudTrail trail with a dedicated, KMS-encrypted S3 bucket
3. **Detected the breach** — found the defaced site and an unauthorized `0.0.0.0/0` inbound SSH rule
4. **Log analysis (grep)** — downloaded and decompressed raw `.json.gz` logs, filtered by `sourceIPAddress` and `eventName`
5. **Log analysis (AWS CLI)** — used `lookup-events` with attribute filters to narrow results programmatically
6. **Log analysis (Athena)** — created an external table over the CloudTrail S3 logs and ran SQL to isolate the actor, event, and timestamp
7. **Root cause identification** — confirmed the attacker's IAM identity, exact timestamp, source IP, and access method (programmatic API call)
8. **OS-level remediation** — used `aureport` and `who` to detect and terminate an unauthorized session, then removed the account
9. **SSH hardening** — disabled password authentication in `sshd_config`, restricted access to key-pair auth only
10. **Application recovery** — restored the original site content from a backup
11. **IAM cleanup** — deleted the compromised IAM user and deactivated its access keys
12. **Network hardening** — removed the unauthorized `0.0.0.0/0` SSH inbound rule

## Outcome

Identified the compromised IAM identity and the exact attack mechanism (an over-permissive inbound rule combined with SSH password authentication left enabled), removed the attacker's access at both the OS and AWS account level, restored the affected application, and closed the misconfigurations that enabled the breach.

## Key Takeaway

A single audit trail (CloudTrail) enabled three independent investigation paths — raw log `grep`, CLI filtering, and SQL querying via Athena — each useful at a different stage of triage. The root cause wasn't just a compromised credential; it was a chain of misconfigurations (overly permissive security group + password auth left enabled on SSH) that turned one compromised action into full instance access. Defense-in-depth at both the network and OS layer would have limited the blast radius even after the initial compromise.

## Repository Contents

```
.
├── README.md                                        # This file
└── AWS_CloudTrail_Investigation_Lab_Guide.md         # Full step-by-step reproduction guide
```

## Reproducing This Lab

See [`AWS_CloudTrail_Investigation_Lab_Guide.md`](./AWS_CloudTrail_Investigation_Lab_Guide.md) for a complete phase-by-phase walkthrough, including AWS CLI commands and Athena SQL queries, to set up and run this lab in a sandbox AWS account.

> **Note:** Run this only in a disposable/sandbox AWS account — never in production. Remember to tear down resources (EC2 instance, S3 bucket, CloudTrail trail) afterward to avoid ongoing charges.

## Disclaimer

This project was performed in a controlled, non-production AWS environment for educational and portfolio purposes. No real systems, data, or third parties were involved.
