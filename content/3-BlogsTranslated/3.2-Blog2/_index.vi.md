---
title: "Blog 2"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

## Tự động xoay vòng OIDC client secret với Application Load Balancer 

*Bởi Kani Murugan, đăng ngày 16 tháng 9 năm 2025, thuộc các chuyên mục: **[Advanced 300](https://aws.amazon.com/blogs/security/category/learning-levels/advanced-300/)**, **[Security, Identity, & Compliance](https://aws.amazon.com/blogs/security/category/security-identity-compliance/)** ,**[Technical How-to](https://aws.amazon.com/blogs/security/category/post-types/technical-how-to/)**.

**[Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/)** đơn giản hóa việc xác thực bằng cách chuyển giao công việc này cho các nhà cung cấp danh tính (IdP) tương thích với OpenID Connect (OIDC). Điều này cho phép các nhà phát triển tập trung vào logic ứng dụng trong khi vẫn sử dụng cơ chế quản lý danh tính mạnh mẽ.

OIDC client secret là các thông tin đăng nhập bí mật được sử dụng trong các giao thức OAuth 2.0 và OIDC để xác thực client (ứng dụng). Tuy nhiên, việc quản lý thủ công OIDC client secret có thể tạo rủi ro bảo mật và gia tăng gánh nặng vận hành.

Như hình 1 minh họa, quản lý thủ công OIDC client secret bắt đầu bằng xác thực thông qua một IdP bên thứ ba.

![](/images/3-BlogsTranslated/blog_2/fig_1_blog_2.png)

**Hình 1: Quản lý thủ công OIDC client secret**

Các rủi ro khi quản lý thủ công OIDC client secret bao gồm:

- Lộ thông tin đăng nhập dạng plaintext

- Cần can thiệp thủ công để điều chỉnh cấu hình [ Application Load Balancer (ALB) ](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/)

- Thiếu giám sát chủ động đối với thay đổi của thông tin đăng nhập

- Thiếu xác minh liên tục các thông tin xác thực

- Không khả thi khi mở rộng cấu hình ALB với nhiều listener rules \


Trong bài viết này, tôi sẽ hướng dẫn cách tự động xoay vòng OIDC client secret bằng [AWS Secrets Manager](https://aws.amazon.com/secrets-manager), [AWS Lambda](https://aws.amazon.com/lambda) và [Amazon](https://aws.amazon.com/eventbridge)[EventBridge](https://aws.amazon.com/eventbridge), giúp nâng cao bảo mật và tối ưu hóa vận hành. Tự động xoay vòng secret là một thực hành bảo mật quan trọng, giúp giảm thiểu rủi ro lộ thông tin đăng nhập và hỗ trợ tuân thủ liên tục.

Đối với cấu hình ALB-OIDC authentication, xem hướng dẫn [Authenticate](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-authenticate-users.html), [Users using an Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-authenticate-users.html).

### **Tổng quan giải pháp**

Giải pháp này cung cấp khung linh hoạt để quản lý thông tin đăng nhập tự động trên nhiều nhà cung cấp OIDC (Auth0 làm ví dụ), với một triển khai cụ thể tích hợp với các dịch vụ AWS. Kiến trúc cốt lõi hỗ trợ: xoay vòng thông tin đăng nhập tự động, lưu trữ bí mật an toàn, thiết kế không phụ thuộc vào nhà cung cấp (provider-agnostic), triển khai mở rộng cho các workflow xác thực khác nhau. Các thành phần chính bao gồm:

- Secrets Manager: Lưu trữ và quản lý an toàn thông tin đăng nhập OIDC (Auth0).

- Lambda: Thực thi logic xoay vòng secret theo lịch định sẵn.

- Elastic Load Balancing: Chuyển giao việc xác thực bằng các OIDC listener rules.

- EventBridge (scheduled): Kích hoạt Lambda theo lịch đã định.

- Custom  [AWS CloudFormation](https://aws-security-blog-content.s3.us-east-1.amazonaws.com/public/sample/2919-automate-oidc-client-secrete-rotation/template-automate-oidc.yaml) resource: Tự động hóa toàn bộ stack và kiến trúc được sử dụng trong bài viết này.

![](/images/3-BlogsTranslated/blog_2/fig_2_blog_2.png)

**Hình 2: Tự động xoay vòng OIDC client secret**

Workflow xác thực, như minh họa trong Hình 2, bao gồm các bước:

1. EventBridge kích hoạt Lambda handler** **`Auth0CredentialHandler`** **mỗi 15 phút.

2. Lambda handler` ``uth0CredentialHandler`** **kết nối đến domain quản lý Auth0 và lấy thông tin client hiện tại — `auth0_current`.

3. Lambda handler` ``Auth0CredentialHandler`** **truy xuất thông tin đăng nhập hiện có** **`auth0/credentials/${Auth0-dev-domain}` từ Secrets Manager và so sánh với thông tin` ``auth0_current` đã lấy ở bước trước. 

- Nếu secret không tìm thấy, handler sẽ thử lại 3 lần trong vòng 30 phút và sau đó ghi log cảnh báo vào [AWS CloudWatch](https://aws.amazon.com/cloudwatch).

- Giả định rằng ARN của secret đã tồn tại trong Secrets Manager

- Nếu thông tin đăng nhập khác nhau, `Auth0CredentialHandler` **cập nhật **`auth0/credentials/${Auth0-dev-domain}`** **với giá trị mới. Nếu thông tin đăng nhập giống nhau, không thực hiện hành động nào. CloudWatch alarms được cấu hình để kích hoạt khi cập nhật secret thành công hoặc thất bại.

- ALB listener rule được cấu hình để lấy thông tin client credentials một cách động từ ARN của resource` auth0/credentials/${Auth0-dev-domain}`** **trong Secrets Manager.

**Khuyến nghị bảo mật**

Có một số biện pháp để nâng cao bảo mật hệ thống xác thực, bao gồm: triển khai quản lý secret tập trung với mã hóa dữ liệu khi lưu trữ (encryption at rest), cấu hình Lambda với quyền tối thiểu (least-privilege), chỉ giới hạn truy cập đến các resource cần thiết trong Secrets Manager và ALB listener, giúp giảm phạm vi rủi ro bảo mật (security blast radius).

Sử dụng CloudWatch alarms để giám sát các sự kiện quan trọng, bao gồm Cập nhật secret, Thất bại khi cập nhật secret, Các vấn đề liên quan đến credential của ALB và Sử dụng [AWS Config](https://aws.amazon.com/config/) để theo dõi cấu hình rule và thực hiện đánh giá bảo mật định kỳ.

Bằng cách Tạo secret riêng biệt cho từng ALB listener rule, cho phép kiểm soát truy cập chi tiết (granular access control) và thu hẹp phạm vi quyền hạn, giúp nâng cao bảo mật tổng thể của hệ thống.

Bằng cách tuân theo các thực hành này, bạn có thể thiết lập khung bảo mật vững chắc cho ứng dụng, đồng thời đảm bảo bảo vệ dữ liệu và quản lý truy cập đúng cách.

**Yêu cầu tiên quyết**

Giải pháp này giả định rằng các điều kiện sau đã được đáp ứng trước khi triển khai:

- Một ALB hiện có được cấu hình với listener và target group, sử dụng làm `Listenerarn` và `targetarn` trong CloudFormation template.

- Tài khoản OIDC IdP (ví dụ: Auth0) và ứng dụng client.

- Thông tin đăng nhập client của ứng dụng Auth0 IdP đã được lưu trữ trong Secrets Manager.

```
     {
       "domain": "your-tenant.auth0.com",
       "client_id": "your-client-id",
       "client_secret": "your-client-secret"
       }
```
**Chi tiết triển khai**

**Lưu ý**: Giải pháp này minh họa tự động xoay vòng OIDC client secret sử dụng Auth0 làm IdP. Mặc dù các nguyên tắc cốt lõi và mô hình kiến trúc có thể áp dụng rộng rãi, các chi tiết triển khai cụ thể có thể khác nhau tùy từng nhà cung cấp danh tính. Người dùng nên tham khảo tài liệu của IdP cụ thể để biết các bước cấu hình chính xác, cách tương tác API và các cơ chế xác thực tương thích với AWS.

Đây là một phương pháp tự động, đơn giản và có thể mở rộng, sử dụng CloudFormation custom resource để tạo các resource được minh họa trong sơ đồ kiến trúc. CloudFormation template và AWS Lambda implementation được lưu trữ trong [demo-stack](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/stackinfo?stackId=arn%3Aaws%3Acloudformation%3Aus-east-1%3A088720121597%3Astack%2Foidc-test-y%2Ffdd4f990-0ff6-11f0-b227-0ed8c3b51117).

**Các thành phần cốt lõi**

Trong phần này, tôi sẽ giải thích các thành phần chính của giải pháp.

**Quy tắc làm mới thông tin xác thực**

Một EventBridge rule được lên lịch để kích hoạt Lambda function `Auth0CredentialHandler`** **mỗi 15 phút, sử dụng `LambdaInvokePermission` của [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam) role.

**Auth0CredentialHandler Lambda function**

Lambda function` ``Auth0CredentialHandler` chịu trách nhiệm quản lý client credentials một cách an toàn. Nó truy xuất cấu hình Auth0 từ Secrets Manager tại resource `auth0/credentials/${Auth0-dev-domain}`, thực hiện API call đến domain Auth0 để lấy token mới, quản lý cập nhật thông tin đăng nhập mới vào Secrets Manager. Lambda này cần quyền truy cập Secrets Manager, được cấp thông qua execution role.

IAM role của Lambda có hai bộ quyền chính:

- AWS managed policy` ``AWSLambdaBasicExecutionRole`, cho phép Lambda tạo log trên CloudWatch. \


- Custom policy, cấp quyền cụ thể trên Secrets Manager `(``GetSecretValue``, ``CreateSecret``, ``UpdateSecret``)` cho các secret dưới path `auth0/credentials/${Auth0-dev-domain}`.

Lambda sẽ thử lại 3 lần trong vòng 30 phút. Nếu tất cả các lần thử thất bại, CloudWatch sẽ ghi cảnh báo và tạo alarms.

```
ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: SecretsManagerAccess
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - secretsmanager:GetSecretValue
                  - secretsmanager:CreateSecret
                  - secretsmanager:UpdateSecret
                Resource:
                  - !Sub 
arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:auth0/*

# Permission for Amazon EventBridge to invoke Lambda
  
  LambdaInvokePermission:
    Type: AWS::Lambda::Permission
    Properties:
      Action: lambda:InvokeFunction
      FunctionName: !Ref Auth0CredentialHandler
      Principal: events.amazonaws.com
      SourceArn: !GetAtt CredentialRefreshRule.Arn
```
**ALB Listener Rules**

Các resource listener rule của Elastic Load Balancing trong CloudFormation được cấu hình để lấy động thông tin client credentials từ Secrets Manager và chuyển tiếp các request đã xác thực đến target group cụ thể. Chúng tích hợp với thông tin đăng nhập Auth0, được Lambda** **`Auth0CredentialHandler`** **tự động làm mới định kỳ. Cấu hình này yêu cầu quyền đọc (read access) trên Secrets Manager để lấy Auth0 client credentials phục vụ quá trình xác thực.

```
# ALB Listener Rules - replace the Oidc config with your endpoints. Only Client credentials are stored in SecretsManager
  ListenerRule1:
    Type: AWS::ElasticLoadBalancingV2::ListenerRule
    Properties:
      ListenerArn: arn:aws:elasticloadbalancing:region:account-id:listener/app/my-load-balancer/1234567890/abcdef
      Priority: 1
      Actions:
        - Type: authenticate-oidc
          AuthenticateOidcConfig:
            ClientId: 
'{{resolve:secretsmanager:auth0/credentials/your-tenant.auth0.com:SecretString:client_id}}'
            ClientSecret: 
'{{resolve:secretsmanager:auth0/credentials/your-tenant.auth0.com:SecretString:client_secret}}'
            Issuer: https://idp1.example.com
            AuthorizationEndpoint: https://idp1.example.com/auth
            TokenEndpoint: https://idp1.example.com/token
            UserInfoEndpoint: https://idp1.example.com/userinfo
            OnUnauthenticatedRequest: authenticate
        - Type: forward
          TargetGroupArn: 
arn:aws:elasticloadbalancing:region:account-id:targetgroup/target-group-1/1234567890abc
      Conditions:
        - Field: path-pattern
          Values:
            - /app1/*
```
**Giám sát và cảnh báo với CloudWatch**

CloudFormation template được cung cấp được cấu hình để thiết lập giám sát bảo mật cho các cập nhật secret. Template này thực hiện Cấu hình cảnh báo cho cả cập nhật secret thành công và cập nhật thất bại, Tạo các CloudWatch metric filter sử dụng [AWS CloudTrail](https://aws.amazon.com/cloudtrail) logs, Thiết lập các alarm tương ứng với các ngưỡng đã định, Tạo một [ Amazon Simple Notification Service (Amazon SNS)](https://aws.amazon.com/sns) topic để gửi cảnh báo tổng hợp. Khi triển khai, giải pháp hạ tầng như mã (infrastructure-as-code) này sẽ tự động phát hiện và thông báo các sự kiện bảo mật tiềm ẩn liên quan đến quản lý secret và các nỗ lực truy cập trái phép.

```
 # CloudWatch Log Group
  CloudTrailLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: secrets-manager-monitoring
      RetentionInDays: 14
      
  # Combined Metric Filter for Both Success and Failed Updates
  SecretUpdateMetricFilter:
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName: !Ref CloudTrailLogGroup
      FilterPattern: !Sub '{ $.eventSource = secretsmanager.amazonaws.com && ($.eventName = UpdateSecret || $.eventName = PutSecretValue) && $.responseElements.ARN = "${MyCustomResource.SecretArn}" }'
      MetricTransformations:
        - MetricNamespace: 'SecretsManager/Updates'
          MetricName: 'SecretUpdates'
          MetricValue: '1'
          DefaultValue: 0

  # Combined Alarm for Both Success and Failed Updates
  SecretUpdateAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Sub '${AWS::StackName}-secret-update'
      AlarmDescription: !Sub 'Alarm for any updates (success or failure) to secret ${MyCustomResource.SecretArn}'
      MetricName: SecretUpdates
      Namespace: SecretsManager/Updates
      Statistic: Sum
      Period: 300
      EvaluationPeriods: 1
      Threshold: 0
      ComparisonOperator: GreaterThanThreshold
      TreatMissingData: notBreaching
      AlarmActions:
        - !Ref SecretMonitoringTopic
```
Để nâng cao độ tin cậy của quá trình xoay vòng secret, hãy triển khai giám sát toàn diện bằng cách: Tạo CloudWatch alarms để phát hiện thất bại khi Lambda thực hiện xoay vòng vượt quá ngưỡng và tỷ lệ lỗi xác thực cao, Giám sát các đột biến bất thường trong tỷ lệ lỗi HTTP 4xx và 5xx từ ALB, Sử dụng CloudTrail để theo dõi các API call và thay đổi cấu hình liên quan đến secrets trong Secrets Manager và các thiết lập load balancer. Bằng cách triển khai các alarm tùy chỉnh này cùng với các cấu hình mặc định, các sự cố bảo mật tiềm ẩn và nỗ lực truy cập trái phép có thể được phát hiện nhanh chóng trên toàn bộ tài nguyên AWS của bạn. Phương pháp nhiều lớp này giúp duy trì khả năng giám sát quá trình xoay vòng secret và nhanh chóng xác định, phản ứng với các vấn đề tiềm ẩn.

Xem hướng dẫn chi tiết tại [Creating CloudWatch alarms for CloudTrail events: examples](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudwatch-alarms-for-cloudtrail.html).

Quy trình triển khai

Triển khai CloudFormation template bằng [ AWS Command Line Interface (AWS CLI)](https://aws.amazon.com/cli) hoặc AWS Management Console. \
 Thay `<your-region>` bằng AWS Region mà bạn muốn triển khai giải pháp.

```
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name oidc-credential-manager-stack \
  --capabilities CAPABILITY_IAM \
  --region 
```
**Lưu ý:** Bạn có thể thêm các tham số bổ sung nếu được yêu cầu bởi cấu hình IdP của bạn.

**Kiểm thử và xác minh**

Tuyên bố: Khuyến nghị thử nghiệm trong môi trường riêng biệt, không quan trọng để đảm bảo mọi cài đặt cụ thể của khách hàng được xác minh đầy đủ trước khi triển khai vào môi trường sản xuất.

Đối với cập nhật secret, hãy xác minh rằng các CloudWatch alarms đã được cấu hình được kích hoạt. Đối với xác thực ALB, kiểm tra ALB access logs để tìm các entry `authentication_success` và sự hiện diện của OIDC identity tokens.

Thiết lập CloudWatch metrics và alarms để giám sát quá trình xoay vòng secret và tỷ lệ xác thực thành công. Xác minh các trường hợp thất bại bằng cách sửa thủ công cấu hình ALB rule trỏ tới một secret ARN khác và xác nhận rằng CloudWatch alarm được kích hoạt. Dưới đây là một ví dụ về sự kiện CloudTrail cho một cập nhật Secrets Manager thành công.

```
{
  "source": ["aws.secretsmanager"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["secretsmanager.amazonaws.com"],
    "eventName": ["UpdateSecret"],
    "responseElements": {"status": "Success"}
  }
}
```
Sau đây là một ví dụ về nhật ký truy cập ALB:

```
/aws/alb/<your-alb-name>:
- Look for entries containing:
  "authentication_success"
  "id_token_authentication_successful"
  "x-amzn-oidc-identity"
  HTTP status code 200
- Example log pattern:
  timestamp elb_name client:port target:port request_processing_time 
  target_processing_time response_processing_time status_code 
  "authentication_success" "x-amzn-oidc-identity: [token]"
```
**Kịch bản nâng cao**

Trong phần này, bạn sẽ học cách giảm thời gian chờ và làm cho cập nhật Secrets Manager gần như đồng bộ (synchronous).

- **Tối ưu hóa đồng bộ Secrets Manager**: Sử dụng [EventBridge](https://aws.amazon.com/eventbridge/integrations/)[ partner](https://aws.amazon.com/eventbridge/integrations/) tích hợp để cấu hình EventBridge gọi Lambda function dựa trên các sự kiện nhận được từ IdP bên thứ ba. Xem hướng dẫn chi tiết tại [Receiving events from a SaaS partner with Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-saas.html) . \


- **Xoay vòng client ID**: Trong khi xoay vòng client secret là kịch bản phổ biến nhất, đôi khi cũng cần xoay vòng client ID. Trong hầu hết các IdP, điều này có nghĩa là tạo một client ứng dụng mới và di chuyển các resource. Để thực hiện, Lambda `Auth0CredentialHandler` cần quyền sửa đổi ALB listener rules (`elasticloadbalancing``:``ModifyRule``, ``elasticloadbalancing``:DescribeListeners, ``elasticloadbalancing``:DescribeRules`). Xoay vòng client ID có thể gây gián đoạn xác thực tạm thời, vì vậy thử nghiệm kỹ lưỡng là cần thiết. Sử dụng AWS Config để giám sát cấu hình ALB rule nhằm phát hiện các thay đổi bất ngờ. Tính năng này tăng cường bảo mật tổng thể, mặc dù có thể tăng độ phức tạp của giải pháp và đôi khi cần can thiệp thủ công. \


- **Chiến lược đa nhà cung cấp (multi-provider)**: Nếu tổ chức của bạn quản lý nhiều IdP, hãy triển khai framework xoay vòng tập trung để trừu tượng hóa các khác biệt riêng của nhà cung cấp, tập trung vào các nguyên tắc bảo mật cốt lõi được nêu trong bài viết này. Các cân nhắc chính bao gồm: tạo giao diện độc lập với nhà cung cấp để hỗ trợ giám sát toàn diện và giảm thiểu chi phí cấu hình.

**Kết luận**

Trong bài viết này, bạn đã khám phá cách tiếp cận toàn diện để tự động xoay vòng OIDC client secret sử dụng các dịch vụ của AWS. Bằng cách triển khai giải pháp này, bạn có thể: Nâng cao bảo mật ứng dụng, Giảm thiểu chi phí quản lý thủ công, Duy trì chiến lược xác thực vững chắc. \
Hãy cân nhắc khám phá các kỹ thuật quản lý danh tính nâng cao hoặc tích hợp xác thực đa yếu tố (MFA) với triển khai OIDC của bạn. \
 Nếu bạn mới làm quen với tự động xoay vòng secrets, tham khảo bài viết [Back to Basics: Secrets Management](https://youtu.be/6oPHw7rT9OI?did=ta_card&trk=ta_card).

| ![](/images/3-BlogsTranslated/blog_2/fig_3_blog_2.png) | Kani Murugan là một kỹ sư bảo mật đã được bổ nhiệm biên chế (tenured) tại Amazon Security, nơi cô chuyên về bảo mật sản phẩm, tập trung vào bảo mật ứng dụng, mạng và dữ liệu. Với hơn 8 năm kinh nghiệm trong nhiều lĩnh vực bảo mật khác nhau, Kani mang đến cho công việc của mình một nền tảng kiến thức phong phú. Ngoài công việc, Kani là người đam mê anime và là một độc giả “không kén chọn”, đọc nhiều chủ đề đa dạng. |
|---|---|
