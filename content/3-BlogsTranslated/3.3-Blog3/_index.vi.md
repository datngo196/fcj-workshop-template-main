---
title: "Blog 3"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

## Sử dụng Raspberry Pi 5 như các nút lai (Hybrid Nodes) của Amazon EKS cho các khối tải (workload) tại biên (edge) 

*Bởi Alberto Crescini, Gladwin Neo, và Utkarsh Pundir, đăng ngày 17 tháng 9 năm 2025, thuộc các chuyên mục:**[Amazon Elastic Kubernetes Service](https://aws.amazon.com/blogs/containers/category/compute/amazon-kubernetes-service/)** **, **[Manufacturing](https://aws.amazon.com/blogs/containers/category/industries/manufacturing/)** **, **[Technical How-to](https://aws.amazon.com/blogs/containers/category/post-types/technical-how-to/)**.*

Kể từ khi ra mắt, [Amazon Elastic Kubernetes Service (Amazon EKS) ](https://aws.amazon.com/eks/) đã vận hành hàng chục triệu cụm (cluster), giúp người dùng tăng tốc triển khai ứng dụng, tối ưu chi phí, và tận dụng tính linh hoạt của Amazon Web Services (AWS) trong việc lưu trữ và vận hành các ứng dụng container hóa. Amazon EKS loại bỏ những phức tạp trong việc duy trì hạ tầng control plane của Kubernetes, đồng thời tích hợp liền mạch với các tài nguyên và hạ tầng AWS.

Tuy nhiên, một số khối tải (workload) cần được chạy tại biên (edge) để xử lý theo thời gian thực, chẳng hạn như các ứng dụng nhạy cảm với độ trễ (latency-sensitive) hoặc tạo ra lượng dữ liệu khổng lồ cần xử lý nhanh tại chỗ.

Trong những tình huống như vậy, khi có kết nối Internet ổn định, người dùng thường muốn duy trì lợi ích của việc tích hợp đám mây, đồng thời vẫn sử dụng phần cứng tại chỗ (on-premises). Chính vì thế, tại AWS re:Invent 2024, chúng tôi đã giới thiệu [Amazon EKS Hybrid Nodes](https://aws.amazon.com/eks/hybrid-nodes/) — một giải pháp cho phép người dùng mở rộng mặt dữ liệu (data plane) của Kubernetes ra biên, trong khi giữ control plane chạy trong [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/). Amazon EKS Hybrid Nodes giúp thống nhất việc quản lý Kubernetes trên môi trường đám mây, tại chỗ và tại biên, bằng cách cho phép người dùng sử dụng hạ tầng tại chỗ như các node trong cụm EKS, bên cạnh [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/).

Để minh họa cách sử dụng Amazon EKS Hybrid Nodes, chúng tôi trình bày một kịch bản thực tế trong lĩnh vực sản xuất (manufacturing) — nơi các hệ thống thường phụ thuộc vào dữ liệu thời gian thực từ các cảm biến số (digital sensors), cần được xử lý cục bộ do yêu cầu về độ trễ và độ tin cậy, trong khi vẫn tận dụng đám mây để phân tích và lưu trữ dài hạn.

Trong ví dụ này, hệ thống đọc các giá trị khoảng cách từ cảm biến siêu âm (ultrasonic sensor), xử lý dữ liệu trực tiếp trên thiết bị biên (edge device) đang hoạt động như một Hybrid Node, và sau đó lưu trữ dữ liệu vào [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) trên AWS.

Trong bài viết này, chúng tôi sẽ hướng dẫn cách triển khai Amazon EKS Hybrid Nodes sử dụng [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/), một nền tảng phổ biến cho tính toán biên (edge computing). Bài viết bao gồm:

1. Thiết lập cụm EKS kết nối liền mạch giữa hạ tầng đám mây và hạ tầng biên

2. Bảo mật kết nối bằng [WireGuard](https://www.wireguard.com/) VPN để thiết lập truyền thông site-to-site

3. Kích hoạt mạng container bằng [Cilium](https://cilium.io/use-cases/cni/) cho các triển khai sử dụng Hybrid Nodes

4. Trình bày một ứng dụng Internet of Things (IoT) thực tế nhằm minh họa sức mạnh của sự tích hợp giữa điện toán biên và điện toán đám mây (edge–cloud integration)

**Vì sao chọn Raspberry Pi 5?** \
Raspberry Pi 5 có thiết kế nhỏ gọn và có thể triển khai tại biên (edge), cho phép xử lý dữ liệu trước khi truyền lên đám mây. Tận dụng ưu điểm này, chúng tôi xây dựng một ứng dụng kiến trúc microservices, trong đó một phần chạy tại biên trên Raspberry Pi 5 và một phần chạy trên AWS trong môi trường đám mây. Ở phía biên, Raspberry Pi cục bộ được kết nối với cảm biến siêu âm (ultrasonic sensor) để thu nhận dữ liệu khoảng cách theo thời gian thực. Dữ liệu này sau đó được xử lý và tải lên cơ sở dữ liệu DynamoDB trên AWS. Tiếp theo, dữ liệu được hiển thị thông qua một bảng điều khiển (dashboard) chạy như một triển khai độc lập trong cụm (cluster deployment). Với cách triển khai này, bạn có thể tiền xử lý dữ liệu tại chỗ, từ đó giảm lượng dữ liệu cần truyền lên AWS, giúp tối ưu băng thông và chi phí, đồng thời tăng hiệu quả xử lý cục bộ cho các ứng dụng biên.

**Tổng quan kiến trúc** \
 Trong môi trường đám mây, chúng tôi triển khai một [Amazon Virtual Private Cloud (Amazon VPC)](https://aws.amazon.com/vpc/) chứa cụm Amazon EKS. Bên trong VPC này, một phiên bản Amazon EC2 đóng vai trò cổng kết nối (gateway) giữa môi trường đám mây và mạng biên (edge network) tại cơ sở. Phiên bản EC2 này thiết lập một đường hầm VPN bảo mật site-to-site sử dụng WireGuard, kết nối với Raspberry Pi 5, thiết bị đóng vai trò là Hybrid Node của chúng tôi. Khi đường hầm VPN được thiết lập, lưu lượng dữ liệu giữa Raspberry Pi và đám mây sẽ được định tuyến thông qua máy chủ WireGuard đang chạy trên Amazon EC2, giúp mở rộng cụm EKS ra đến vùng biên. Từ góc nhìn của cụm EKS, Raspberry Pi hoạt động giống như bất kỳ node nào khác, mặc dù nó nằm ngoài phạm vi của VPC. Kiến trúc tổng thể được thể hiện trong hình minh họa bên dưới.

![Enter image alt description](Images/rfy_Image_1.png)

Mặt điều khiển Kubernetes (control plane) được quản lý hoàn toàn bởi AWS, bao gồm API server, etcd datastore, scheduler và controller manager. Trong phần hướng dẫn này, chúng tôi cấu hình control plane của Kubernetes với điểm truy cập công khai (public endpoint), cho phép các node Raspberry Pi có thể giao tiếp với control plane thông qua Internet. AWS đảm nhận toàn bộ sự phức tạp trong việc bảo mật và mở rộng control plane của Kubernetes để đảm bảo tính sẵn sàng cao (high availability), giúp bạn có thể tập trung vào phát triển và triển khai ứng dụng của mình.

Chúng tôi sử dụng một phiên bản EC2 chuyên dụng chạy WireGuard, đóng vai trò cổng VPN (VPN gateway), tạo đường hầm bảo mật giữa AWS và hạ tầng biên (edge infrastructure). Máy chủ này hoạt động như trung tâm (hub) trong mô hình hub-and-spoke, cho phép trao đổi dữ liệu giữa control plane của Amazon EKS và các node Raspberry Pi, phục vụ cho các thao tác như kubectl exec, truy xuất log, và xử lý webhook.

Các thiết bị Raspberry Pi chạy các thành phần node tiêu chuẩn của Kubernetes gồm kubelet, kube-proxy, và container runtime, cùng với công cụ dòng lệnh Amazon EKS Hybrid Nodes CLI (nodeadm). Các node này đăng ký với cụm EKS thông qua [AWS Systems Manager](https://aws.amazon.com/systems-manager/), và hiển thị trong cụm như những worker node tiêu chuẩn, mặc dù được vận hành trên phần cứng do người dùng tự quản lý.

Các node Raspberry Pi chủ động thiết lập kết nối với control plane của Amazon EKS thông qua Internet công cộng. Quá trình này bao gồm giao tiếp với API server để đăng ký node, cập nhật trạng thái pod, và gửi yêu cầu tài nguyên (resource requests). Phương thức public endpoint này giúp đơn giản hóa việc kết nối, đồng thời vẫn duy trì mức độ bảo mật cao nhờ xác thực qua [AWS Identity and Access Management (IAM) ](https://aws.amazon.com/iam/)và mã hóa TLS.

**Bắt đầu thiết lập** \
Để kết nối mạng giữa thiết bị Raspberry Pi và cụm EKS đang chạy trên đám mây, trước tiên chúng ta cấu hình một máy chủ WireGuard nhẹ trên một phiên bản EC2. Máy chủ này chỉ hoạt động như một cổng mạng (network gateway), vì vậy phiên bản EC2 t4g.nano tiết kiệm chi phí là đủ cho hầu hết các trường hợp sử dụng.

Sau khi máy chủ WireGuard được khởi động, chúng ta cài đặt client WireGuard trên Raspberry Pi để thiết lập kết nối liên tục (persistent connection), đồng thời cấu hình định tuyến phù hợp (routing) để cho phép trao đổi lưu lượng giữa Raspberry Pi và VPC mà cụm EKS đang sử dụng.

Tiếp theo, chúng ta thêm node lai (hybrid node) vào cụm EKS, cấu hình CNI (Container Network Interface), và cuối cùng cài đặt ứng dụng để hoàn thiện quá trình triển khai.

**Yêu cầu:**

- [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/), chạy [Ubuntu 24.10](https://old-releases.ubuntu.com/releases/oracular/), bật SSH

- Kết nối Ethernet có dây (khuyến nghị để đảm bảo ổn định)

- [AWS Command Line Interface (AWS CLI)](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

- [kubectl](https://kubernetes.io/docs/tasks/tools/)

- [Helm](https://helm.sh/docs/intro/install/)

**Bước 1: Tạo cụm EKS** \
Bắt đầu bằng việc tạo một Amazon VPC trong Region AWS mà bạn chọn, với ít nhất một subnet công khai và một subnet riêng tư. Các subnet này sẽ chứa các worker node trên đám mây và các giao diện mạng cần thiết để giao tiếp với control plane. Khi thiết lập cụm EKS, hãy đảm bảo rằng các tham số mạng từ xa (remote networking parameters) được cấu hình để control plane có thể kết nối với các hybrid node và pod nằm ngoài VPC.

Để đơn giản hóa quá trình này, AWS cung cấp bộ [Template Terraform](https://github.com/aws-samples/sample-eks-hybrid-nodes-raspberry-pi/tree/main/terraform) trên AWS Samples GitHub repository. Các template này tự động hóa nhiều phần cấu hình mạng và Amazon EKS, chẳng hạn như kích hoạt hybrid networking và chuẩn bị các chính sách IAM và CNI cần thiết.

Nếu bạn mới làm quen với Amazon EKS Hybrid Nodes hoặc muốn tìm hiểu sâu hơn về quá trình cấu hình, hãy tham khảo [tài liệu chính thức của AWS](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-overview.html) về việc kích hoạt cụm EKS cho Hybrid Nodes.

**Bước 2: Thiết lập máy chủ VPN** \
Amazon EKS Hybrid Nodes cần kết nối ổn định và mạng riêng giữa môi trường tại chỗ/biên và VPC của bạn. Điều này đòi hỏi thiết lập VPN hoặc giải pháp mạng riêng bảo mật tương tự. Có [nhiều tùy chọn được AWS tài liệu hóa](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/network-to-amazon-vpc-connectivity-options.html), chẳng hạn như[AWS Site-to-Site VPN](https://aws.amazon.com/vpn/site-to-site-vpn/), [AWS Direct Connect](https://aws.amazon.com/directconnect/), hoặc kết nối VPN tự triển khai. Ở đây, chúng tôi sử dụng WireGuard, một phần mềm mã nguồn mở cho kết nối VPN nhanh và bảo mật.

**2.1 Cài đặt WireGuard** \
Chúng tôi cài đặt WireGuard bằng cách triển khai máy chủ trên một EC2 instance trong tài khoản AWS của mình. Bạn có thể tham khảo bất kỳ hướng dẫn cài đặt WireGuard chuẩn nào để cấu hình máy chủ trên EC2, đảm bảo mở cổng UDP/51820 từ IP cục bộ tới security group của EC2. Bắt đầu bằng cách cài đặt WireGuard thông qua APT.

```
sudo apt update && sudo apt install -y wireguard
sudo mkdir -p /etc/wireguard
```
**2.2 Tạo cấu hình WireGuard**

Tiếp theo, sử dụng trình soạn thảo mà bạn ưa thích để thêm cấu hình sau, thay thế các giá trị giữ chỗ (placeholders) bằng Public Key và Private Key của máy chủ WireGuard của bạn.

```
sudo nano /etc/wireguard/wg0.conf
```
Thêm cấu hình sau (thay thế các giá trị giữ chỗ):

```
[Interface]
PrivateKey = <client-private.key>
Address = 10.200.0.2/24
[Peer]
# Public key from AWS server (/etc/wireguard/public.key)
PublicKey = <public.key>
# Your EC2 instance's public IP
Endpoint = <ec2-public-ip>:51820
# WireGuard server network, AWS VPC CIDR & EKS Service CIDR
AllowedIPs = 10.200.0.1/24,10.0.0.0/24,172.16.0.0/16
PersistentKeepalive = 25
```
Sau đó, kích hoạt dịch vụ WireGuard và xác minh rằng kết nối đã được thiết lập với máy chủ Amazon EC2.

```
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
sudo wg show
```
Bạn sẽ thấy kết quả tương tự như sau:

```
interface: wg0
  public key: zH6sK7s93lF4ZkPoe8L7TtyOe0e0zFqUYrqUJo1hXVA=
  private key: (hidden)
  listening port: 51820

peer: 8N3e1FzEJmGaJ8t6C2Zh1n3oA2uNfz8MZp4nCzHn3XA=
  endpoint: 52.14.123.45:51820
  allowed ips: 10.0.0.0/16
  latest handshake: 23 seconds ago
  transfer: 1.47 MiB received, 1.21 MiB sent
  persistent keepalive: every 25 seconds
```
Như bước đầu tiên cho việc cấu hình mạng, cần bật IPv4 forwarding trên instance để nó có thể định tuyến các gói dữ liệu giữa các giao diện mạng: 

```
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
Tiếp theo, để cho phép EC2 instance chuyển tiếp lưu lượng giữa mạng WireGuard và VPC của bạn, hãy cấu hình iptables để thực hiện Network Address Translation (NAT) và cho phép chuyển tiếp gói dữ liệu (packet forwarding).

```
# Enable masquerading for outgoing traffic via Wireguard interface
iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE
# Allow packets from VPC interface to be forwarded to Wireguard
iptables -A FORWARD -i eth0 -o wg0 -j ACCEPT
# Allow return traffic from Wireguard back into VPC for established connections
iptables -A FORWARD -i wg0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```
Lệnh đầu tiên báo cho kernel ghi lại địa chỉ IP nguồn (source IP) của các gói dữ liệu đi qua giao diện WireGuard (wg0) thành địa chỉ IP của EC2 instance, điều này cần thiết để định tuyến lưu lượng trả về. Quy tắc thứ hai cho phép các gói từ giao diện VPC (eth0) được chuyển tiếp tới WireGuard. Quy tắc thứ ba cho phép lưu lượng trả về từ WireGuard quay lại VPC, nhưng chỉ áp dụng cho các kết nối đã được thiết lập trước đó.

Tiếp theo, để đảm bảo các quy tắc này được giữ nguyên sau khi khởi động lại (persist across reboots), hãy cài đặt và cấu hình gói iptables-persistent:

```
sudo apt update
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo systemctl enable netfilter-persistent
```
Điều này lưu các quy tắc hiện tại vào`/etc/iptables/rules.v4`** **và** **`/etc/iptables/rules.v6` và đảm bảo chúng được áp dụng tự động sau mỗi lần khởi động lại.

Ở bước cuối cùng, hãy tắt tính năng kiểm tra source/destination (source/destination check) trên giao diện của instance. Theo mặc định, AWS bật kiểm tra source/destination để đảm bảo một instance chỉ xử lý lưu lượng được gửi đến hoặc đi từ chính nó. Tuy nhiên, vì instance của chúng ta đóng vai trò là gateway, định tuyến gói dữ liệu thay cho các thiết bị khác trong mạng, nên cần tắt giới hạn này.

**Thêm Raspberry Pi vào cụm như một node từ xa**** \
**Khi mạng đã được cấu hình và cụm EKS đã được tạo, bước tiếp theo là thêm node vào cụm để Kubernetes có thể bắt đầu lập lịch (scheduling) pod trên node này.

Trước tiên, đảm bảo node có thể xác thực với cụm. Amazon EKS Hybrid Nodes xác thực với cụm EKS thông qua IAM, do đó cần gán IAM role cho các máy tại chỗ (on-premises). Điều này yêu cầu thiết lập cơ chế xác thực bằng Systems Manager hoặc IAM Roles Anywhere. Hướng dẫn trên GitHub sử dụng Systems Manager Hybrid Activations cho mục đích này. Bạn có thể theo [hướng dẫn](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-creds.html) để tạo AmazonEKSHybridNodesRole bằng một trong hai tùy chọn. Sau đó, đăng ký node bằng nodeadm. Hãy theo dõi các hướng dẫn trong [chỉ dẫn](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-join.html) và chắc chắn chỉ định role mà bạn đã tạo ở bước trước.

**Thiết lập Container Network Interface (CNI)**

Sau khi cụm EKS và các hybrid node được tạo và cấu hình thành công, node của chúng ta vẫn hiển thị trạng thái Not Ready. Nguyên nhân là Container Network Interface (CNI) chưa được cài đặt. CNI là thành phần quan trọng chịu trách nhiệm thiết lập các giao diện mạng bên trong container, cấp phát địa chỉ IP, và cấu hình định tuyến để pod có thể giao tiếp liền mạch trong cụm và với mạng bên ngoài. Nếu không có CNI, các node Kubernetes không thể cung cấp kết nối mạng cần thiết cho pod, từ đó ngăn cản triển khai workload. Vì vậy, cần cài đặt CNI trước khi hybrid node sẵn sàng. Cilium là giải pháp mã nguồn mở, cloud-native để cung cấp, bảo mật và quan sát kết nối mạng giữa các workload, và được [hỗ trợ chính thức](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-cni.html) cho Amazon EKS Hybrid Nodes

**Bước 1: Cài đặt Cilium**

Sau khi cài đặt Helm, chúng ta thêm Cilium Helm chart và cài đặt Cilium vào cụm EKS.

Tạo file cilium-values.yaml: 

```
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: eks.amazonaws.com/compute-type
          operator: In
          values:
          - hybrid
ipam:
  mode: cluster-pool
  operator:
    clusterPoolIPv4MaskSize: 25
    clusterPoolIPv4PodCIDRList:
    - 172.16.0.0/24 
operator:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: eks.amazonaws.com/compute-type
            operator: In
            values:
              - hybrid
  unmanagedPodWatcher:
    restart: false
envoy:
  enabled: false
```
Sau đó chúng ta có thể cài đặt Cilium bằng Helm:

```
[ec2-user@ip-10-0-6-175 terraform]$ helm repo add cilium https://helm.cilium.io/
> helm install cilium cilium/cilium \
> --version 1.16.6 \
> --namespace kube-system \
> --values cilium-values.yaml
NAME: cilium
LAST DEPLOYED: Mon Apr 28 03:50:01 2025
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
Bạn đã cài đặt Cilium thành công, bây giờ hãy đợi cho đến khi cả hai pod đều sẵn sàng:

```
[ec2-user@ip-10-0-6-175 terraform]$ kubectl get pods -A
NAMESPACE     NAME                             READY  STATUS   RESTARTS  AGE
kube-system   cilium-gzvhm                     1/1    Running  0         4m50s
kube-system   cilium-operator-9c54b46b8-whgn9  1/1    Running  0         4m50s
kube-system   coredns-6d87fdb75-95wn2          1/1    Running  0           35s
kube-system   coredns-6d87fdb75-b2xf5          1/1    Running  0           35s
kube-system   kube-proxy-w48jx                 1/1    Running  0         9m31s
```
**Bước 2: Xác minh các nút lai đang chạy**

Chúng ta có thể kiểm tra xem tất cả các nút trong cụm EKS của mình có đang chạy thành công hay không. Chúng ta có thể kiểm tra trạng thái nút:

```
kubectl get nodes
```
Nút này hiện được đánh dấu là Sẵn sàng.

```
[ec2-user@ip-10-0-6-175 terraform]$ kubectl get nodes
NAME                        STATUS  ROLES   AGE     VERSION
ip-10-0-6-175.ec2.internal  Ready   <none>  9m31s   v1.30.9-eks-5d632ec
```
Khi cụm của chúng ta hoạt động và mạng lưới container hoạt động như mong đợi, chúng ta sẽ thấy nút ở trạng thái Sẵn sàng trên Bảng thông tin tổng quan về nút Amazon EKS, như minh họa trong hình sau.

![Enter image alt description](Images/BvO_Image_2.png)

**Triển khai ứng dụng mẫu trên Amazon EKS Hybrid Nodes với tích hợp edge**

Ứng dụng bao gồm hai deployment Kubernetes:

1. **Ultrasonic**: Đọc các giá trị đo từ cảm biến siêu âm và ghi vào DynamoDB. \


2. **Dashboard**: Đọc dữ liệu từ DynamoDB và hiển thị trên giao diện tương tác (UI).

Chúng tôi sử dụng cảm biến siêu âm HC-SR04, thiết bị phát sóng âm và đo thời gian hồi âm để tính khoảng cách. Loại cảm biến này phổ biến trong các ngành sản xuất và ô tô, ví dụ:

- Phát hiện sự hiện diện hoặc vắng mặt của vật thể trên dây chuyền lắp ráp

- Đo mực chất lỏng trong các thùng chứa

- Giám sát tình trạng chỗ đỗ xe

Trong một thiết lập nâng cao hơn, pipeline này có thể mở rộng để chạy mô hình nhận diện vật thể (object detection) tại chỗ và kích hoạt sự kiện, ví dụ gửi thông tin lên hàng đợi [Amazon Simple Queue Service (Amazon SQS)](https://aws.amazon.com/sqs/) dựa trên các điều kiện được phát hiện.

Tuy nhiên, trong demo này, chúng tôi ưu tiên tính rõ ràng và minh bạch. Node sẽ phát hiện khoảng cách của một vật thể đặt trước Raspberry Pi và đẩy giá trị này vào bảng DynamoDB mỗi 10 giây.

### **Bước 1: Yêu cầu phần cứng và thiết lập**

Cảm biến siêu âm HC-SR04:

- Điện trở 1kΩ và 2kΩ (dùng trong mạch chia điện áp)

- Dây nối (Jumper Wires)

- Breadboard

Chúng tôi sử dụng breadboard để nhanh chóng thử nghiệm mà không cần hàn. Breadboard giúp tối ưu hóa việc đi dây, hỗ trợ lặp nhanh, và đặt cảm biến HC-SR04 theo chiều đứng để tối ưu vị trí đo. Mỗi hàng trên breadboard chia sẻ điện liên tục, giúp đơn giản hóa kết nối.

**Kết nối HC-SR04 với GPIO của Raspberry Pi**

Kết nối chân 3.3V và GND của Raspberry Pi với đường nguồn (power rails) của breadboard.

Cắm cảm biến HC-SR04 vào breadboard. Sau đó kết nối:

- `VCC` → Breadboard + rail (dây đỏ)

- `GND` → Breadboard – rail (dây đen)

- `TRIG` → Raspberry Pi GPIO 4 (dây cam)

- `ECHO` → Voltage divider → GPIO 17 (dây xanh)

Mạch chia điện áp sử dụng điện trở 1kΩ và 2kΩ mắc nối tiếp, giúp giảm tín hiệu 5V từ chân **ECHO** của cảm biến xuống khoảng 3.3V, an toàn cho GPIO của Raspberry Pi.

Sơ đồ minh họa được cung cấp để làm rõ cách bố trí này.

Bản đồ GPIO này sau này có thể được trừu tượng hóa và quản lý động qua Kubernetes ConfigMaps, giúp linh hoạt trong việc xử lý cấu hình phần cứng cho các deployment khác nhau. Chúng tôi sẽ trình bày chi tiết ở phần sau.

![Enter image alt description](Images/OB7_Image_3.png)

### **Bước 2: Triển khai bảng DynamoDB**

Chúng tôi lưu dữ liệu vào bảng DynamoDB có tên `eks-timeseries`, được tạo trong Region `eu-west-1` . Bảng sử dụng schema như sau:

- **Partition Key**: `yyyymmdd` \


- **Sort Key**:` hhmmss`

Schema này cho phép truy vấn theo thời gian một cách hiệu quả và phù hợp với các mô hình dữ liệu dạng time series, nơi dữ liệu được truy xuất theo ngày và sắp xếp theo dấu thời gian (timestamp).

[AWS CloudFormation](https://aws.amazon.com/cloudformation/) template:

```
Resources:
  TimeSeriesTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: eks-timeseries
      AttributeDefinitions:
        - AttributeName: yyyymmdd
          AttributeType: S
        - AttributeName: hhmmss
          AttributeType: S
      KeySchema:
        - AttributeName: yyyymmdd
          KeyType: HASH
        - AttributeName: hhmmss
          KeyType: RANGE
      BillingMode: PAY_PER_REQUEST
      Tags:
        - Key: Environment
          Value: EdgeDemo
```
### **Bước 3: Triển khai ứng dụng cảm biến**

Trong [repository GitHub](https://github.com/aws-samples/sample-eks-hybrid-nodes-raspberry-pi/tree/main/examples/ultrasonic-demo), có thư mục examples chứa dự án ultrasonic-demo. Thư mục này bao gồm:

- Các file manifest Kubernetes

- Mã nguồn Python

- Dockerfile để build image container

Bắt đầu bằng việc build Docker image từ thư mục ultrasonic-demo và đẩy lên container registry của bạn, ví dụ [Amazon Elastic Container Registry (Amazon ECR)](https://aws.amazon.com/ecr/).

Hãy chú ý phần **ConfigMap** trong manifest, vì nó định nghĩa các biến môi trường mà script Python cần để truy cập GPIO và DynamoDB, đồng thời cấu hình AWS CLI.

Để triển khai ứng dụng, chạy lệnh:

```
kubectl apply -f manifest.yaml
```
Sau khi triển khai, xác minh rằng pod ultrasonic-sensor đang chạy:

```
kubectl get pods
```
Tiếp theo, kiểm tra logs để giám sát dữ liệu từ cảm biến và các ghi chép vào DynamoDB:

```
kubectl logs <pod-name>
```
Bạn sẽ thấy các giá trị khoảng cách xuất hiện trong logs, và các kết quả tương tự cũng sẽ hiển thị trên bảng DynamoDB.

![Enter image alt description](Images/p6R_Image_4.png)

### **Bước 4: Triển khai frontend dashboard**

Để trực quan hóa dữ liệu từ cảm biến, chúng tôi xây dựng một frontend dashboard truy vấn dữ liệu trực tiếp từ DynamoDB và hiển thị dưới dạng biểu đồ cập nhật theo thời gian thực (live-updating chart).

Bất kỳ người dùng dữ liệu đã xác thực, kể cả các ứng dụng bên ngoài, đều có thể truy vấn DynamoDB trực tiếp.

Chúng tôi muốn tất cả ứng dụng đều được container hóa, do đó quyết định triển khai dashboard thông qua một deployment trong cụm.

Xem lại thư mục frontend trong repository để hiểu cấu trúc.

Build Docker image cho frontend và đẩy lên container registry, tương tự như cách đã làm với backend. Sau đó, cập nhật manifest Kubernetes được cung cấp.

Để triển khai ứng dụng, chạy lệnh:

```
kubectl apply -f manifest.yaml
```
Sau đó, bạn có thể thiết lập port-forwarding cho service trên máy local:

```
kubectl port-forward svc/pi-dashboard 8080:80
```
Truy cập dashboard từ **trình duyệt** bằng cách mở `http://localhost:8080`.

Bạn sẽ thấy biểu đồ trực tiếp (live chart) cập nhật các giá trị khoảng cách theo thời gian thực, được truy xuất trực tiếp từ bảng DynamoDB.

![Enter image alt description](Images/T7l_Image_5.png)

**Kết luận**

Vậy là xong! Bạn vừa biến Raspberry Pi 5 thành một node của cụm Amazon EKS, hoạt động ngoài Amazon VPC, đọc dữ liệu thực từ cảm biến thông qua GPIO, và đẩy dữ liệu một cách an toàn lên đám mây bằng Amazon DynamoDB. Chúng tôi hy vọng ví dụ này với Raspberry Pi có thể làm minh họa thực tế cho cách kiến trúc Kubernetes lai (hybrid Kubernetes) kết nối các môi trường vật lý với đám mây, bất kể bạn đang làm việc với: Cảm biến trong nhà máy, Server tại cửa hàng bán lẻ, Engine xử lý inference trong bệnh viện hoặc sàn giao dịch. Đối với các tổ chức muốn hiện đại hóa hạ tầng phân tán, Amazon EKS Hybrid Nodes cung cấp một hướng đi thực tiễn. Bạn có thể build một lần và chạy trên đám mây, tại biên (edge), hoặc trên máy chủ bare metal của mình. Với sự linh hoạt và mạnh mẽ của phương pháp này, hiện tại là thời điểm lý tưởng để bắt đầu proof of concept và khám phá các khả năng cho tổ chức của bạn.

Muốn tự thử nghiệm? Hãy tham khảo [repository GitHub](https://github.com/aws-samples/sample-eks-hybrid-nodes-raspberry-pi/tree/main), clone ví dụ, và bắt đầu xây dựng. Ngoài ra, hãy xem [hướng dẫn chính thức](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-overview.html) về Amazon EKS Hybrid Nodes, và liên hệ đội ngũ AWS của bạn nếu có thắc mắc khi bắt đầu.

—----------------------------------------------------------------------------

**Về các tác giả**

|  | Alberto Crescini là Enterprise Solutions Architect tại AWS, hỗ trợ các công ty năng lượng và tiện ích ở Vương quốc Anh xây dựng hạ tầng cho quá trình chuyển đổi năng lượng. Anh hỗ trợ khách hàng trong các dự án như cân bằng lưới điện (grid balancing) và sản xuất năng lượng linh hoạt (flexible generation), đồng thời hướng dẫn họ hiện đại hóa hệ thống và nền tảng hyperscale thông qua lĩnh vực tập trung AWS Containers. |
|---|---|
|  | Utkarsh Pundir là Containers Specialist Solutions Architect tại AWS, nơi anh hỗ trợ khách hàng xây dựng các giải pháp trên EKS. Các lĩnh vực trọng tâm của anh bao gồm kiến trúc lai (hybrid architecture) và triển khai workload Generative AI trên EKS như một phần của các sáng kiến go-to-market của AWS. |
|  | Gladwin Neo là Containers Specialist Solutions Architect tại AWS, nơi anh hỗ trợ khách hàng di chuyển và hiện đại hóa các workload để triển khai trên Amazon Elastic Kubernetes Service (EKS) hoặc Amazon Elastic Container Service (ECS). |
