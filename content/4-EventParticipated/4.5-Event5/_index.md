---
title: "AWS Security Specialty Workshop"
date: 2025-11-29
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

# Summary Report: “AWS Security Specialty Workshop”

### Event Objectives

- Deeply understand the role of the Security Pillar within the Well-Architected Framework
- Master the 5 security pillars: IAM, Detection, Infrastructure, Data Protection, Incident Response
- Update on top security threats in the Vietnam cloud market
- Practice reviewing permissions (IAM) and building Incident Response (IR) Playbooks

### Speakers

- **AWS Security Experts Team**
- _(Specialized in security architecture and compliance)_

### Key Highlights

#### Foundation & Identity (Pillar 1)
- **Core Principles**: Strictly applying Least Privilege, Zero Trust, and Defense in Depth principles.
- **Modern IAM**: Shifting from IAM Users (long-term credentials) to IAM Roles and AWS Identity Center (SSO) for centralized management.
- **Access Control**: Using Service Control Policies (SCPs) and Permission Boundaries to limit permission scopes in multi-account environments.
- **Mini Demo**: Practice Validating IAM Policies and simulating access to detect security flaws.

#### Detection & Infrastructure (Pillar 2 & 3)
- **Continuous Monitoring**: Enabling CloudTrail (org-level), GuardDuty, and Security Hub for continuous monitoring.
- **Logging Strategy**: Logging at every layer: VPC Flow Logs (network), ALB logs (application), S3 logs (storage).
- **Network Security**: Network segmentation with VPC, combining Security Groups and NACLs. Edge protection with WAF, Shield, and Network Firewall.

#### Data Protection & Incident Response (Pillar 4 & 5)
- **Encryption**: Encrypting data in-transit and at-rest on S3, EBS, RDS using KMS.
- **Secrets Management**: Eliminating hard-coded credentials by using Secrets Manager and Parameter Store with rotation mechanisms.
- **IR Automation**: Building Playbooks for common incidents (compromised keys, malware) and automating isolation processes using Lambda/Step Functions.

### Key Takeaways

#### Zero Trust Mindset
- **Identity is the new perimeter**: In the Cloud environment, Identity is the most critical defense barrier, not IP addresses.
- Never trust by default; always verify and grant least privilege.

#### Automation is Key
- Manual security cannot keep up with the speed of the Cloud. **Detection-as-Code** and **Auto-remediation** must be applied to minimize human risk.

### Applying to Work

- **Review IAM**: Audit all IAM Users, delete old keys, and switch to IAM Roles for applications.
- **Enable GuardDuty**: Activate GuardDuty across all regions/accounts to detect anomalous behaviors.
- **Implement Secrets Manager**: Replace config files containing DB passwords with API calls to Secrets Manager.
- **Draft IR Playbook**: Write an incident response procedure for "Compromised IAM Key" scenarios and rehearse with the team.

### Event Experience

The workshop dove deep into technical details, comprehensively covering the security aspects that a Cloud Engineer needs to master.

#### Comprehensive Framework
- Structuring content by the **5 Pillars** helped me systematize previously scattered security knowledge into a standard framework.
- The section on **Top threats in Vietnam** was very practical, helping identify specific risks within the local context.

#### Practical Demos
- The demos on **Access Analyzer** and **Validate IAM Policy** were extremely useful, solving the daily headache of debugging permissions.
- The **Incident Response** section helped me understand that "detection" is only half the story; automated "response" is the goal.

#### Some event photos

_Add your event photos here_

> Overall, the event confirmed that security is not a blocker but an enabler that allows businesses to operate faster and more securely.