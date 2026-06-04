**Bài học: Kubernetes Fundamentals – Kiến trúc và Các khái niệm cốt lõi**

### 1. Kubernetes là gì?

**Kubernetes** (thường gọi tắt là **K8s**) là một hệ thống **container orchestration** mã nguồn mở.

Nó được sử dụng để **tự động hóa việc triển khai, scaling và quản lý** các ứng dụng được đóng gói dưới dạng container.

Kubernetes giúp bạn:

- Chạy container một cách **tin cậy và có khả năng mở rộng**
- Sử dụng tài nguyên hiệu quả
- Expose ứng dụng ra bên ngoài (cho người dùng hoặc hệ thống khác)

Nói một cách hình ảnh: **Kubernetes giống như Docker nhưng có “robot + trí tuệ nhân tạo”** để tự động vận hành và quản lý toàn bộ vòng đời container.

Kubernetes là **cloud-agnostic**, nghĩa là bạn có thể chạy trên on-premise hoặc trên bất kỳ nền tảng đám mây công cộng nào (AWS, Azure, GCP…).

---

### 2. Kiến trúc Cluster Kubernetes

Một **Kubernetes Cluster** là một tập hợp các tài nguyên máy tính hoạt động như một đơn vị thống nhất và có tính **high availability**.

#### Các thành phần chính:

|Thành phần|Mô tả|Vai trò|
|---|---|---|
|**Control Plane**|Bộ não của cluster|Quản lý toàn bộ cluster: scheduling, scaling, deployment, healing…|
|**Nodes**|Máy chủ worker (VM hoặc physical server)|Chạy thực tế các container|
|**Container Runtime**|Phần mềm chạy container (thường là **containerd**)|Thực thi container trên node|
|**Kubelet**|Agent chạy trên mỗi node|Giao tiếp với Control Plane qua Kubernetes API|

**Tóm lại**: **Control Plane** điều phối → **Nodes** thực thi container.

---

### 3. Pods – Đơn vị nhỏ nhất trong Kubernetes

**Pod** là **đơn vị nhỏ nhất** mà bạn quản lý trong Kubernetes.

- Một Pod có thể chứa **một hoặc nhiều container**.
- Các container trong cùng Pod chia sẻ **storage** và **networking**.
- Kiến trúc phổ biến nhất: **1 Pod = 1 Container**.

**Lưu ý quan trọng**:

- Bạn **không nên** quản lý Pod trực tiếp.
- Pod là **tạm thời** (ephemeral). Chúng có thể bị xóa, bị evict do thiếu tài nguyên, hoặc node bị lỗi.
- Chỉ nên đặt nhiều container vào cùng một Pod khi chúng **rất chặt chẽ** (tightly coupled) và cần giao tiếp với nhau ở mức độ cao.

**Quy tắc vàng**: **Hãy nghĩ về Pod thay vì nghĩ về Container** khi làm việc với Kubernetes.

---

### 4. Các thành phần trong Control Plane

|Thành phần|Chức năng chính|Ghi chú|
|---|---|---|
|**kube-apiserver**|API Server – điểm giao tiếp chính của cluster|Có thể scale horizontally|
|**etcd**|Key-value store (database) của cluster|Lưu trữ toàn bộ trạng thái cluster, cần high availability|
|**kube-scheduler**|Lên lịch gán Pod vào Node|Dựa trên tài nguyên, affinity, data locality…|
|**kube-controller-manager**|Quản lý các controller (Node Controller, Job Controller, Endpoint Controller…)|Tự động sửa chữa và duy trì trạng thái mong muốn|
|**Cloud Controller Manager**|Tích hợp Kubernetes với nền tảng đám mây (AWS, Azure, GCP…)|**Tùy chọn** – dùng khi chạy Kubernetes trên cloud|

---

### 5. Các thành phần trên Node

|Thành phần|Chức năng|
|---|---|
|**Kubelet**|Agent giao tiếp với Control Plane, đảm bảo các Pod trên node hoạt động đúng|
|**kube-proxy**|Quản lý networking, implement **Service**, cho phép giao tiếp giữa các Pod và từ bên ngoài vào cluster|

---

### 6. Bảng tóm tắt các khái niệm quan trọng

|Thuật ngữ|Mô tả ngắn gọn|Ghi chú|
|---|---|---|
|**Cluster**|Toàn bộ hệ thống Kubernetes|Bao gồm Control Plane + nhiều Nodes|
|**Node**|Máy chủ worker thực thi container|Có thể là VM hoặc physical server|
|**Pod**|Đơn vị nhỏ nhất chứa container|Tạm thời, dễ bị thay thế|
|**Service**|Lớp trừu tượng ổn định phía trên Pod|Đây là thứ bạn thường tương tác như một ứng dụng|
|**Job**|Công việc chạy một lần (ad-hoc)|Tạo Pod → chạy xong → tự động xóa|
|**Ingress**|Cách để truy cập Service từ bên ngoài cluster|Thường dùng Load Balancer|
|**Ingress Controller**|Phần mềm triển khai Ingress (ví dụ: AWS ALB Ingress Controller, NGINX)|Cần cài đặt riêng|

---

### 7. Stateless vs Persistent Storage (Rất quan trọng)

**Pods là tạm thời** → Do đó, **storage mặc định** trong Kubernetes cũng là **ephemeral** (mất khi Pod bị xóa hoặc di chuyển sang node khác).

- Giống như **Instance Store** trên EC2.
- Nếu ứng dụng cần lưu trữ dữ liệu lâu dài (session, database, file…), bạn **không nên** lưu trong Pod.

**Giải pháp**:

- Sử dụng **Persistent Volume (PV)** → Vòng đời của volume độc lập với Pod.
- Dữ liệu sẽ được giữ lại ngay cả khi Pod bị xóa hoặc di chuyển giữa các node.

**Best Practice**: Thiết kế ứng dụng theo hướng **stateless** ở mức Pod. State nên được lưu ở Persistent Volume hoặc các dịch vụ bên ngoài (RDS, S3, DynamoDB…).

---

### Tóm tắt nhanh để ôn thi

- **Kubernetes** = Hệ thống orchestration container mã nguồn mở.
- **Control Plane** = Bộ não (apiserver, etcd, scheduler, controller manager…).
- **Node** = Nơi chạy thực tế container.
- **Pod** = Đơn vị nhỏ nhất (thường 1 container/pod).
- **Service** = Lớp ổn định để truy cập ứng dụng.
- **Pod là tạm thời** → Dùng **Persistent Volume** nếu cần lưu trữ lâu dài.
- Kubernetes có thể chạy trên **AWS thông qua EKS**.

---

Bài học này chỉ là **high-level overview**. Nếu bạn đang học khóa AWS, các bài sau sẽ đi sâu vào **Amazon EKS** (Elastic Kubernetes Service) – cách AWS triển khai và quản lý Kubernetes.

Bạn muốn tôi tóm tắt so sánh nhanh giữa **ECS** và **Kubernetes/EKS** không? Hoặc bạn muốn tiếp tục với phần EKS?