---
title: "AWS DevOps & Modern Operations"
date: 2025-11-17
weight: 4
chapter: false
pre: " <b> 4.4. </b> "

---

# Summary Report: “AWS DevOps & Modern Operations”

### Event Objectives

- Build a DevOps mindset and master key efficiency metrics (DORA metrics)
- Establish a complete CI/CD process using AWS Developer Tools
- Modernize infrastructure management with Infrastructure as Code (IaC) using CloudFormation and CDK
- Deploy containerized applications (Docker) on ECS, EKS, and App Runner
- Set up a comprehensive Observability system for distributed applications

### Speakers

- **AWS Experts Team**
- _(Specialists in DevOps, Containers, and Observability)_

### Key Highlights

#### DevOps Mindset & CI/CD
- **DORA Metrics**: Understanding the importance of Deployment Frequency, Lead Time for Changes, MTTR, and Change Failure Rate.
- **Git Strategies**: Comparison of GitFlow and Trunk-based development strategies.
- **Pipeline Automation**: Demo of a full CI/CD pipeline from CodeCommit (source), CodeBuild (build/test) to CodeDeploy (deployment), orchestrated by CodePipeline.
- **Deployment Strategies**: Safe deployment techniques: Blue/Green, Canary, and Rolling updates.

#### Infrastructure as Code (IaC)
- **CloudFormation**: Managing infrastructure via templates, concept of Stacks, and Drift detection.
- **AWS CDK**: Using familiar programming languages to define infrastructure, leveraging "Constructs" and reusable patterns.
- **IaC Choice**: Discussion on criteria for choosing between CloudFormation and CDK depending on the project.

#### Container Services
- **Spectrum of Compute**: From image management (ECR) to orchestration options: ECS (simplified), EKS (standard Kubernetes), and App Runner (maximum simplification).
- **Microservices Deployment**: Comparison and demo of microservices deployment on different platforms.

#### Monitoring & Observability
- **Full-stack Observability**: Combining CloudWatch (Metrics, Logs, Alarms) and X-Ray (Distributed Tracing) for a comprehensive view of system health.
- **Best Practices**: Setting up Monitoring Dashboards and effective On-call processes.

### Key Takeaways

#### Automation First
- **CI/CD** is not just a toolset but a culture that minimizes human error and accelerates release velocity.
- **IaC** is a mandatory standard for modern infrastructure, ensuring consistency across Dev/Test/Prod environments.

#### Operational Excellence
- **Observability** is more critical than simple Monitoring, especially in Microservices architectures for error tracing.
- Choosing the right deployment strategy (like Blue/Green) helps reduce Downtime to zero.

### Applying to Work

- **Refactor Pipeline**: Transition current manual build processes to AWS CodePipeline with automated testing steps.
- **Adopt CDK**: Start using AWS CDK to define infrastructure for new projects instead of manual Console operations.
- **Containerization**: Dockerize applications and pilot deployment on AWS App Runner for smaller services.
- **Setup Tracing**: Integrate AWS X-Ray into applications to monitor latency between microservices.

### Event Experience

The full-day event was intense but the content was very cohesive, moving logically from Mindset to Tools and Operations.

#### Integrated Workflow
- The **"Full CI/CD pipeline walkthrough"** demo was impressive, showing the complete picture of how code moves from a developer's machine to Production.
- Understanding the clear differences and specific use cases for **ECS vs EKS** gave me confidence in proposing solutions for the company.

#### Practical Focus
- Lessons on **Deployment strategies** (Feature flags, Canary) were very practical for solving the team's "deployment fear."
- The **Career roadmap** at the end provided clear direction for developing DevOps skills.

#### Some event photos

_Add your event photos here_

> Overall, the workshop systematized the entire body of knowledge regarding modern operations, helping me understand the tight coupling between Code, Infrastructure, and Monitoring.