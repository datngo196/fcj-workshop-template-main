---
title: "AWS DevOps & Modern Operations"
date: 2025-11-17
weight: 4
chapter: false
pre: " <b> 4.4. </b> "

---

# Báo cáo tổng hợp: “AWS DevOps & Modern Operations”

### Event Objectives

- Xây dựng tư duy DevOps và nắm vững các chỉ số đo lường hiệu quả (DORA metrics)
- Thiết lập quy trình CI/CD hoàn chỉnh sử dụng bộ công cụ AWS Developer Tools
- Hiện đại hóa quản lý hạ tầng bằng mã (Infrastructure as Code) với CloudFormation và CDK
- Triển khai ứng dụng container hóa (Docker) trên ECS, EKS và App Runner
- Thiết lập hệ thống giám sát toàn diện (Observability) cho ứng dụng phân tán

### Speakers

- **Đội ngũ chuyên gia AWS (AWS Experts)**
- _(Các chuyên gia về DevOps, Container và Observability)_

### Key Highlights

#### DevOps Mindset & CI/CD
- **DORA Metrics**: Hiểu tầm quan trọng của các chỉ số: Tần suất triển khai, Thời gian thay đổi (Lead time), MTTR.
- **Git Strategies**: So sánh chiến lược GitFlow và Trunk-based development.
- **Pipeline Automation**: Demo quy trình CI/CD đầy đủ từ CodeCommit (lưu trữ), CodeBuild (build/test) đến CodeDeploy (triển khai) được điều phối bởi CodePipeline.
- **Deployment Strategies**: Các kỹ thuật triển khai an toàn: Blue/Green, Canary, và Rolling updates.

#### Infrastructure as Code (IaC)
- **CloudFormation**: Quản lý hạ tầng bằng template, khái niệm Stacks và phát hiện sai lệch (Drift detection).
- **AWS CDK**: Sử dụng ngôn ngữ lập trình quen thuộc để định nghĩa hạ tầng, tận dụng các "Constructs" và mẫu tái sử dụng.
- **IaC Choice**: Thảo luận về tiêu chí lựa chọn giữa CloudFormation và CDK tùy theo dự án.

#### Container Services
- **Spectrum of Compute**: Từ quản lý image (ECR) đến các lựa chọn điều phối: ECS (đơn giản), EKS (Kubernetes chuẩn) và App Runner (đơn giản hóa tối đa).
- **Microservices Deployment**: So sánh và demo triển khai microservices trên các nền tảng khác nhau.

#### Monitoring & Observability
- **Full-stack Observability**: Kết hợp CloudWatch (Metrics, Logs, Alarms) và X-Ray (Distributed Tracing) để có cái nhìn toàn diện về sức khỏe hệ thống.
- **Best Practices**: Thiết lập Dashboard giám sát và quy trình trực chiến (On-call) hiệu quả.

### Key Takeaways

#### Automation First
- **CI/CD** không chỉ là công cụ mà là văn hóa giúp giảm thiểu lỗi con người và tăng tốc độ phát hành.
- **IaC** là tiêu chuẩn bắt buộc cho hạ tầng hiện đại, giúp môi trường Dev/Test/Prod đồng nhất.

#### Operational Excellence
- **Observability** (Khả năng quan sát) quan trọng hơn Monitoring (Giám sát) đơn thuần, đặc biệt trong kiến trúc Microservices để truy vết lỗi (tracing).
- Việc chọn đúng chiến lược triển khai (như Blue/Green) giúp giảm thiểu Downtime xuống bằng 0.

### Applying to Work

- **Refactor Pipeline**: Chuyển đổi các quy trình build thủ công hiện tại sang AWS CodePipeline với các bước test tự động.
- **Adopt CDK**: Bắt đầu sử dụng AWS CDK để định nghĩa hạ tầng cho các dự án mới thay vì thao tác trên Console.
- **Containerization**: Đóng gói ứng dụng vào Docker và thử nghiệm triển khai lên AWS App Runner cho các service nhỏ.
- **Setup Tracing**: Tích hợp AWS X-Ray vào ứng dụng để theo dõi độ trễ (latency) giữa các microservices.

### Event Experience

Sự kiện kéo dài cả ngày nhưng nội dung rất liền mạch, đi từ tư duy (Mindset) đến công cụ (Tools) và vận hành (Operations).

#### Integrated Workflow
- Phần Demo **"Full CI/CD pipeline walkthrough"** rất ấn tượng, cho thấy bức tranh toàn cảnh code di chuyển từ máy lập trình viên lên môi trường Production như thế nào.
- Hiểu rõ sự khác biệt và trường hợp sử dụng cụ thể của **ECS vs EKS**, giúp tôi tự tin hơn khi đề xuất giải pháp cho công ty.

#### Practical Focus
- Các bài học về **Deployment strategies** (Feature flags, Canary) rất thực tế để giải quyết bài toán "sợ deploy" của team.
- Phần **Career roadmap** cuối giờ cung cấp định hướng rõ ràng cho lộ trình phát triển kỹ năng DevOps.

#### Some event photos

_Add your event photos here_

> Overall, buổi workshop đã hệ thống hóa lại toàn bộ kiến thức về vận hành hiện đại, giúp tôi hiểu rõ mối liên kết chặt chẽ giữa Code, Hạ tầng và Giám sát.