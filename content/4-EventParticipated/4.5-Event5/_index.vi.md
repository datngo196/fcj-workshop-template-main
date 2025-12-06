---
title: "AWS Security Specialty Workshop"
date: 2025-11-29
weight: 5
chapter: false
pre: " <b> 4.5. </b> "

---

# Báo cáo tổng hợp: “AWS Security Specialty Workshop”

### Event Objectives

- Hiểu sâu về vai trò của Security Pillar trong khung kiến trúc Well-Architected
- Nắm vững 5 trụ cột bảo mật: IAM, Detection, Infrastructure, Data Protection, Incident Response
- Cập nhật các mối đe dọa an ninh hàng đầu tại thị trường Cloud Việt Nam
- Thực hành rà soát quyền hạn (IAM) và xây dựng quy trình phản ứng sự cố (IR Playbook)

### Speakers

- **Đội ngũ chuyên gia bảo mật AWS (AWS Security Experts)**
- _(Chuyên sâu về kiến trúc bảo mật và tuân thủ)_

### Key Highlights

#### Foundation & Identity (Pillar 1)
- **Core Principles**: Áp dụng triệt để nguyên tắc đặc quyền tối thiểu (Least Privilege), Zero Trust và bảo mật nhiều lớp (Defense in Depth).
- **Modern IAM**: Chuyển dịch từ việc dùng IAM Users (long-term credentials) sang IAM Roles và AWS Identity Center (SSO) để quản lý tập trung.
- **Access Control**: Sử dụng Service Control Policies (SCP) và Permission Boundaries để giới hạn phạm vi quyền hạn trong môi trường multi-account.
- **Mini Demo**: Thực hành xác thực (Validate) IAM Policy và mô phỏng quyền truy cập để phát hiện lỗi bảo mật.

#### Detection & Infrastructure (Pillar 2 & 3)
- **Continuous Monitoring**: Kích hoạt CloudTrail (toàn tổ chức), GuardDuty và Security Hub để giám sát liên tục.
- **Logging Strategy**: Ghi nhật ký tại mọi tầng: VPC Flow Logs (mạng), ALB logs (ứng dụng), S3 logs (lưu trữ).
- **Network Security**: Phân đoạn mạng (Segmentation) với VPC, kết hợp Security Groups và NACLs. Bảo vệ biên với WAF, Shield và Network Firewall.

#### Data Protection & Incident Response (Pillar 4 & 5)
- **Encryption**: Mã hóa dữ liệu đường truyền (in-transit) và dữ liệu lưu trữ (at-rest) trên S3, EBS, RDS sử dụng KMS.
- **Secrets Management**: Loại bỏ hard-code credentials bằng cách sử dụng Secrets Manager và Parameter Store với cơ chế xoay vòng (rotation).
- **IR Automation**: Xây dựng kịch bản phản ứng (Playbook) cho các sự cố phổ biến (lộ key, malware) và tự động hóa quy trình cô lập bằng Lambda/Step Functions.

### Key Takeaways

#### Zero Trust Mindset
- **Identity is the new perimeter**: Trong môi trường Cloud, định danh (Identity) là hàng rào bảo vệ quan trọng nhất, không phải địa chỉ IP.
- Không bao giờ tin tưởng mặc định, luôn xác thực và cấp quyền tối thiểu.

#### Automation is Key
- Bảo mật thủ công không thể theo kịp tốc độ của Cloud. Cần áp dụng **Detection-as-Code** và **Auto-remediation** (tự động khắc phục) để giảm thiểu rủi ro con người.

### Applying to Work

- **Review IAM**: Rà soát lại toàn bộ IAM User, xóa các key cũ và chuyển sang dùng IAM Role cho các ứng dụng.
- **Enable GuardDuty**: Bật GuardDuty trên tất cả các region/account để phát hiện các hành vi bất thường.
- **Implement Secrets Manager**: Thay thế các config file chứa mật khẩu DB bằng việc gọi API tới Secrets Manager.
- **Draft IR Playbook**: Viết quy trình xử lý sự cố cho tình huống "IAM Key bị lộ" và diễn tập với team.

### Event Experience

Buổi workshop đi sâu vào chi tiết kỹ thuật, bao phủ toàn diện các khía cạnh bảo mật mà một kỹ sư Cloud cần nắm vững.

#### Comprehensive Framework
- Việc chia nội dung theo **5 Pillars** giúp tôi hệ thống hóa lại kiến thức bảo mật rời rạc trước đây thành một khung chuẩn chỉnh.
- Phần nói về **Top threats tại Việt Nam** rất thực tế, giúp nhận diện các rủi ro cụ thể với bối cảnh trong nước.

#### Practical Demos
- Demo về **Access Analyzer** và **Validate IAM Policy** rất hữu ích, giải quyết được nỗi đau đầu khi debug quyền hạn (permissions) hàng ngày.
- Phần **Incident Response** giúp tôi hiểu rằng "phát hiện" (detection) chỉ là một nửa câu chuyện, "phản ứng" (response) tự động mới là đích đến.

#### Some event photos

_Add your event photos here_

> Overall, sự kiện khẳng định rằng bảo mật không phải là nút chặn (blocker) mà là yếu tố cho phép doanh nghiệp vận hành nhanh và an toàn hơn (enabler).