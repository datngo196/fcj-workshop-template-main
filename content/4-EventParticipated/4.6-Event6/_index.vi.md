---
title: "Workshop: Data Science on AWS"
date: 2025-10-16
weight: 6
chapter: false
pre: " <b> 4.6. </b> "

---

# Báo cáo tổng hợp: “Workshop: Data Science on AWS”

### Mục tiêu sự kiện

-   Khám phá toàn bộ hành trình xây dựng một hệ thống Data Science hiện đại từ lý thuyết đến thực hành.
-   Nắm vững quy trình Data Science Pipeline trên AWS (từ S3, Glue đến SageMaker).
-   Thực hành xử lý dữ liệu thực tế (dataset IMDb) và triển khai mô hình học máy (Sentiment Analysis).
-   Phân tích bài toán chi phí và hiệu năng giữa nền tảng Cloud và hạ tầng On-premise.

### Diễn giả

-   **Anh Văn Hoàng Kha** – Cloud Solutions Architect, AWS Community Builder
-   **Anh Bạch Doãn Vương** – Cloud DevOps Engineer, AWS Community Builder

### Nội dung nổi bật

#### 1. Tầm quan trọng của Cloud và Tổng quan Pipeline
-   **Vai trò của Cloud**: Thảo luận về lý do tại sao Data Science hiện đại cần dựa vào Cloud để đạt được khả năng mở rộng và tích hợp, thay vì bị giới hạn bởi phần cứng on-premise.
-   **AWS Data Science Pipeline**:
    -   **Lưu trữ**: Sử dụng **Amazon S3** làm nền tảng Data Lake.
    -   **ETL/Xử lý**: Sử dụng **AWS Glue** để tích hợp và làm sạch dữ liệu serverless.
    -   [cite_start]**Mô hình hóa**: Tận dụng **Amazon SageMaker** làm trung tâm để xây dựng, huấn luyện và triển khai mô hình[cite: 132].
    -   [cite_start]Tổng quan về bộ công cụ AWS AI/ML stack bao gồm các AI Services, ML Services và hạ tầng cơ sở[cite: 121].

#### 2. Demo thực hành (Hands-on)
-   **Demo 1: Xử lý dữ liệu với AWS Glue**:
    -   **Kịch bản**: Xử lý và làm sạch dữ liệu thô từ **dataset IMDb**.
    -   **Kỹ thuật**: Minh họa cách thực hiện Feature Engineering. [cite_start]Workshop cũng giới thiệu các tùy chọn khác nhau từ Low-code (SageMaker Canvas) [cite: 129] [cite_start]đến Code-first (Numpy/Pandas)[cite: 130].
-   **Demo 2: Phân tích cảm xúc (Sentiment Analysis) với SageMaker**:
    -   **Kịch bản**: Huấn luyện và triển khai mô hình AI để phân tích cảm xúc văn bản.
    -   **Quy trình**: Thực hiện vòng đời "Train, Tune, Deploy" trên SageMaker Studio. [cite_start]Buổi demo cũng đề cập đến khái niệm "Bring Your Own Model" (BYOM), cho phép chạy các framework như TensorFlow và PyTorch trên hạ tầng AWS[cite: 133].

#### 3. Thảo luận chuyên sâu
-   **Cloud vs. On-premise**: Phân tích sâu về tối ưu hóa chi phí và các chỉ số hiệu năng. Sự linh hoạt của Cloud cho phép thử nghiệm các tác vụ nặng mà không cần đầu tư vốn lớn (CAPEX) ban đầu.
-   **Dự án củng cố**: Hướng dẫn thực hiện dự án nhỏ sau workshop để người tham dự tự thực hành lại quy trình.

### Bài học chính (Key Takeaways)

#### Quy trình kỹ thuật
-   **Pipeline thống nhất**: Một quy trình Data Science hiệu quả đòi hỏi sự tích hợp mượt mà giữa lưu trữ (S3), làm sạch (Glue) và mô hình hóa (SageMaker).
-   [cite_start]**Lựa chọn công cụ**: Hiểu rõ khi nào nên dùng các dịch vụ có sẵn (Managed Services như Rekognition, Polly) [cite: 124, 126] và khi nào cần xây dựng mô hình tùy chỉnh trên SageMaker.

#### Ứng dụng thực tế
-   **Kết nối đào tạo - doanh nghiệp**: Sự chuyển dịch từ lý thuyết hàn lâm sang ứng dụng thực tế nằm ở khả năng tự động hóa và mở rộng của hệ thống.
-   **Tư duy chi phí**: Các dự án dữ liệu thành công phải cân bằng được giữa độ chính xác của mô hình và chi phí tính toán.

### Ứng dụng vào công việc

-   **Áp dụng AWS Glue**: Chuyển đổi các script ETL cục bộ sang AWS Glue để tự động hóa việc làm sạch dữ liệu quy mô lớn.
-   **Triển khai SageMaker**: Di chuyển các mô hình thử nghiệm từ Jupyter notebook cá nhân lên SageMaker Studio để chuẩn hóa quy trình huấn luyện.
-   **Thực hiện dự án**: Hoàn thành dự án nhỏ được hướng dẫn để nắm chắc quy trình xử lý dataset IMDb.

### Trải nghiệm sự kiện

Workshop **“Data Science on AWS”** thực sự là cầu nối giữa kiến thức sinh viên và thực tế doanh nghiệp.
-   **Kết nối trực tiếp**: Sự kiện giúp tôi thấy được cách các doanh nghiệp hàng đầu vận hành hệ thống phân tích dữ liệu.
-   **Thực tế hóa lý thuyết**: Việc tận mắt xem demo xử lý **dataset IMDb** và triển khai mô hình **Sentiment Analysis** giúp giải mã sự phức tạp của AI trên nền tảng đám mây.
-   **Chia sẻ từ chuyên gia**: Phần thảo luận với các AWS Community Builder mang lại góc nhìn chiến lược về bài toán "Cloud vs On-premise", giúp tôi hiểu giá trị của Cloud không chỉ nằm ở công nghệ mà còn ở hiệu quả kinh tế.

#### Một số hình ảnh sự kiện

_Thêm hình ảnh sự kiện của bạn tại đây_