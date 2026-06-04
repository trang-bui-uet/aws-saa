**Bài học: Amazon EKS (Elastic Kubernetes Service) – Tổng quan**

### 1. EKS là gì?

**Amazon EKS** là dịch vụ **Kubernetes được quản lý (managed)** do AWS cung cấp.

AWS lấy Kubernetes mã nguồn mở và biến nó thành một dịch vụ hoàn chỉnh, tích hợp sâu với hệ sinh thái AWS. Về bản chất, đây vẫn là **Kubernetes chuẩn**, nhưng được AWS vận hành và mở rộng để hoạt động tốt hơn trên AWS.

**Lý do chọn EKS**:

- Bạn muốn dùng Kubernetes nhưng **không muốn tự quản lý Control Plane**.
- Bạn đã có hệ thống Kubernetes và muốn chuyển lên cloud mà không bị lock-in vendor.
- Cần tích hợp chặt chẽ với các dịch vụ AWS (IAM, VPC, Load Balancer, ECR…).

EKS có thể chạy ở nhiều môi trường:

- Trong AWS (phổ biến nhất)
- Trên **AWS Outposts**
- **EKS Anywhere** (on-premise)
- **EKS Distro** (phiên bản mã nguồn mở)

---

### 2. Kiến trúc EKS

Trong EKS, AWS quản lý **Control Plane**, còn bạn quản lý **Worker Nodes**.

#### Control Plane (do AWS quản lý)

- Được AWS vận hành hoàn toàn.
- Tự động **scale** theo tải.
- Chạy phân tán trên **nhiều Availability Zone** (high availability).
- **etcd** cũng do AWS quản lý và phân tán đa AZ.
- Tích hợp sẵn với:
    - **IAM** (xác thực & phân quyền)
    - **VPC** (networking)
    - **Elastic Load Balancer**
    - **ECR** (lưu trữ container image)

#### Worker Nodes (bạn quản lý)

Bạn có **3 lựa chọn** chính để chạy Pod:

|Loại Node|Mô tả|Ai quản lý|Ưu điểm|Nhược điểm|Khi nào nên dùng|
|---|---|---|---|---|---|
|**Self-managed Nodes**|EC2 instances do bạn tự tạo và quản lý|Bạn|Toàn quyền kiểm soát|Phải tự scaling, patching, cập nhật|Cần tùy biến cao|
|**Managed Node Groups**|EC2 do AWS provisioning và quản lý vòng đời|AWS (một phần)|Dễ quản lý hơn self-managed|Vẫn phải trả tiền EC2|Hầu hết các trường hợp production|
|**Fargate**|Chạy Pod serverless, không cần EC2|AWS hoàn toàn|Không lo capacity, scaling, patching|Hạn chế một số tính năng|Workload không cần GPU, Windows, hay tùy biến sâu|

**Cảnh báo quan trọng**: Một số tính năng chỉ hỗ trợ ở một số loại node nhất định:

- Windows Pods
- GPU / Inferentia
- Bottlerocket
- AWS Outposts / Local Zones

Bạn cần kiểm tra kỹ khả năng hỗ trợ trước khi chọn loại node.

---

### 3. Persistent Storage trong EKS

Vì Pod là tạm thời, EKS hỗ trợ các giải pháp lưu trữ bền vững:

- **Amazon EBS** → Block storage (giống volume gắn vào EC2)
- **Amazon EFS** → File storage chia sẻ (nhiều Pod có thể mount cùng lúc)
- **Amazon FSx** → Hiệu năng cao (cho workload chuyên biệt)

---

### 4. Mô hình mạng trong EKS (Kiến trúc 2 VPC)

Khi triển khai EKS, bạn sẽ có **2 VPC**:

1. **AWS Managed VPC**
    - Chứa **EKS Control Plane**
    - Do AWS quản lý hoàn toàn
    - Chạy trên nhiều Availability Zone
2. **Customer VPC** (VPC của bạn)
    - Chứa **Worker Nodes** (nếu dùng EC2)
    - Chứa các tài nguyên khác của ứng dụng

**Cách giao tiếp**:

- Control Plane giao tiếp với Worker Nodes thông qua **Elastic Network Interface (ENI)** được inject vào Customer VPC.
- Có thể dùng **Public Control Plane Endpoint** để quản trị.
- Người dùng truy cập ứng dụng qua **Ingress** → Load Balancer trong Customer VPC.

---

### 5. Tóm tắt nhanh các điểm quan trọng

|Thành phần|Ai quản lý|Ghi chú|
|---|---|---|
|**Control Plane**|AWS|Tự động scale, đa AZ, etcd được quản lý|
|**etcd**|AWS|Phân tán đa AZ|
|**Worker Nodes**|Bạn (hoặc AWS một phần)|3 lựa chọn: Self-managed, Managed Node Group, Fargate|
|**Networking**|Kết hợp|ENI injection + VPC CNI|
|**Storage**|Bạn cấu hình|EBS, EFS, FSx|
|**Security**|IAM + VPC|Tích hợp sâu với AWS|
|**Container Registry**|ECR|Tích hợp sẵn|

---

### So sánh nhanh: ECS vs EKS

|Tiêu chí|**ECS**|**EKS**|
|---|---|---|
|Loại orchestration|AWS proprietary|Kubernetes chuẩn|
|Control Plane|AWS quản lý|AWS quản lý|
|Độ phức tạp|Dễ hơn|Phức tạp hơn|
|Tính linh hoạt|Trung bình|Rất cao|
|Hệ sinh thái|AWS-only|Cloud-agnostic + lớn|
|Khi nào nên dùng|Muốn đơn giản, nhanh triển khai|Cần Kubernetes chuẩn, đa cloud, hoặc đã có hệ thống K8s|

---

**Kết luận**:

- **EKS** là lựa chọn khi bạn cần **Kubernetes thực thụ** trên AWS mà không muốn tự quản lý Control Plane.
- Bạn có thể chọn giữa **Managed Node Groups** (phổ biến nhất) hoặc **Fargate** tùy theo nhu cầu.
- Luôn kiểm tra khả năng hỗ trợ tính năng (GPU, Windows…) trước khi chọn loại node.

Bạn muốn tôi làm bảng so sánh chi tiết hơn giữa **ECS Fargate** vs **EKS Fargate** không? Hoặc bạn muốn đi sâu vào phần thực hành EKS?