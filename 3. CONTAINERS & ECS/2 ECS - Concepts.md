### Giới thiệu về ECS

Trong bài học này, chúng ta sẽ tìm hiểu về **Elastic Container Service (ECS)**.

Trong bài trước, bạn đã tạo Docker image và chạy container trên **EC2 instance** sử dụng Docker engine. Đây là cách truyền thống. Tuy nhiên, **ECS** là dịch vụ giúp bạn chạy container trên hạ tầng mà **AWS quản lý hoàn toàn hoặc một phần**, giảm đáng kể công việc quản trị.

**ECS** đối với container cũng giống như **EC2** đối với máy ảo.

ECS hoạt động ở **hai chế độ**:

- **EC2 mode**: Sử dụng các EC2 instances làm container hosts (bạn có thể thấy và quản lý các instance này bình thường).
- **Fargate mode**: Chạy theo kiểu **serverless**, AWS hoàn toàn quản lý phần hạ tầng, bạn chỉ cần tập trung định nghĩa và kiến trúc ứng dụng bằng container.

---

### Các khái niệm chính của ECS

**ECS** là dịch vụ quản lý container. Bạn cung cấp container image và các chỉ dẫn, ECS sẽ chịu trách nhiệm **orchestrate** (phân bổ và chạy) chúng.

#### 1. Cluster (Cụm)

- Đây là nơi các container của bạn chạy.
- Bạn tạo một **cluster**, sau đó triển khai task hoặc service vào cluster đó.

#### 2. Container Definition (Định nghĩa Container)

- Dùng để khai báo thông tin của **một container đơn lẻ**.
- Bao gồm:
    - Đường dẫn đến **container image** (từ Docker Hub, **ECR**, hoặc registry khác).
    - Cổng (port) mà container sử dụng (ví dụ: port 80).
- Đây chỉ là "con trỏ" trỏ đến image và cấu hình cơ bản.

#### 3. Task Definition (Định nghĩa Task)

- **Task** đại diện cho một **ứng dụng hoàn chỉnh**.
- Một task có thể chứa **1 hoặc nhiều container**.
    - Ví dụ: 1 container (như ứng dụng "container of cats").
    - Hoặc nhiều container (web app + database).

**Task Definition** chứa những thông tin quan trọng sau:

- Các **container definitions**.
- **Task Role** (IAM Role) — rất quan trọng.
- CPU và Memory cần sử dụng.
- Networking mode.
- Compatibility (EC2 hay Fargate).

> **Task Role**: Là IAM Role mà task đảm nhận. Các container bên trong task sẽ sử dụng quyền từ role này để tương tác với các dịch vụ AWS khác. Đây là **best practice** để cấp quyền cho container.

#### 4. Service (Dịch vụ) và Service Definition

- **Task** đơn lẻ **không tự scale** và **không có high availability**.
- **Service** giúp khắc phục điều đó.

**Service** cho phép bạn:

- Quyết định số lượng bản sao (copies) của task muốn chạy (ví dụ: 5 copies).
- Tự động thay thế task bị lỗi.
- Kết hợp với **Load Balancer** để phân tải.
- Cung cấp khả năng **scale** và **high availability**.

**Lưu ý**: Trong demo cuối phần này, chúng ta sẽ chỉ chạy **một task đơn lẻ**, nên không cần dùng Service.

---

### Tóm tắt các khái niệm quan trọng

**1. Container Definition** Định nghĩa image và port của một container. Chỉ trỏ đến container image và khai báo port.

**2. Task Definition** Định nghĩa toàn bộ ứng dụng. Có thể chứa 1 hoặc nhiều container. Chứa **Task Role**, tài nguyên (CPU/Memory), và các cấu hình khác.

**3. Task Role** IAM Role được task assume để container truy cập các dịch vụ AWS một cách an toàn.

**4. Service** Dùng để chạy nhiều bản sao của task, cung cấp scale và high availability. Thường dùng cho ứng dụng production.

---

**Kết luận**: Bạn tạo **Cluster** → Định nghĩa **Task** (chứa container) → (Tùy chọn) Bọc trong **Service** để scale và HA → Triển khai vào cluster (EC2 mode hoặc Fargate mode).

Các khái niệm này giống nhau ở cả hai chế độ. Ở bài sau, chúng ta sẽ đi sâu hơn vào sự khác biệt giữa **EC2 mode** và **Fargate mode**.

---

Bạn đã hoàn thành bài học này. Khi sẵn sàng, hãy chuyển sang bài tiếp theo!