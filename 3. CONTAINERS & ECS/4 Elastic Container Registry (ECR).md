**Bài học: Elastic Container Registry (ECR) – Lý thuyết**

### 1. ECR là gì?

**Elastic Container Registry (ECR)** là dịch vụ **quản lý container image registry** do AWS cung cấp.

Nó đóng vai trò như **Docker Hub**, nhưng được tích hợp sâu trong hệ sinh thái AWS. ECR dùng để **lưu trữ, quản lý và phân phối container images** cho các dịch vụ như:

- **ECS** (Elastic Container Service)
- **EKS** (Elastic Kubernetes Service)
- Hoặc bất kỳ nền tảng container nào khác.

---

### 2. Cấu trúc của ECR

ECR có cấu trúc phân cấp rõ ràng:

|Cấp độ|Mô tả|Ví dụ|
|---|---|---|
|**Registry**|Cấp cao nhất. Mỗi AWS account được cấp **1 Public Registry** và **1 Private Registry**|-|
|**Repository**|Bên trong mỗi Registry có nhiều Repository (giống như repo trong Git)|my-app, nginx, backend-api|
|**Container Image**|Bên trong mỗi Repository có nhiều phiên bản image|Image my-app|
|**Tag**|Mỗi image có thể có nhiều tag (tag phải là duy nhất trong repository)|v1.0, latest, prod, sha256:abc123...|

**Lưu ý quan trọng**: Tag phải **duy nhất** trong cùng một repository.

---

### 3. Public Registry vs Private Registry

Đây là điểm khác biệt quan trọng về **bảo mật**:

|Loại Registry|Ai có thể **Pull** (đọc)|Ai có thể **Push** (ghi)|Yêu cầu quyền|
|---|---|---|---|
|**Public Registry**|**Bất kỳ ai** (read-only)|Chỉ người có quyền|Push cần quyền|
|**Private Registry**|Chỉ người có quyền|Chỉ người có quyền|**Cả Pull và Push đều cần quyền**|

- **Public Registry**: Phù hợp chia sẻ image công khai (ai cũng pull được).
- **Private Registry**: An toàn hơn, phù hợp cho image nội bộ của công ty/doanh nghiệp.

---

### 4. Các tính năng và lợi ích chính của ECR

#### **Bảo mật & Quyền truy cập**

- Tích hợp sâu với **IAM**: Toàn bộ quyền truy cập (đọc/ghi) được kiểm soát bởi IAM policies.
- Hỗ trợ **Image Scanning** (quét lỗ hổng bảo mật):
    - **Basic scanning**
    - **Enhanced scanning** (sử dụng **Inspector**): Quét theo từng layer, phát hiện vấn đề ở **hệ điều hành** và **các gói phần mềm** bên trong container.

#### **Giám sát & Logging**

- Gửi **metrics gần thời gian thực** vào **CloudWatch** (số lần push, pull, xác thực…).
- Ghi lại mọi hành động API vào **CloudTrail**.
- Tạo **events** gửi vào **EventBridge** → hỗ trợ xây dựng workflow tự động khi có thay đổi image.

#### **Replication (Nhân bản)**

- Hỗ trợ **nhân bản image** giữa các Region (**cross-region**).
- Hỗ trợ nhân bản giữa các AWS Account (**cross-account**).

---

### 5. Tóm tắt nhanh (dễ nhớ cho kỳ thi)

|Nội dung|Chi tiết quan trọng|
|---|---|
|**ECR là gì?**|Managed container image registry của AWS (thay thế Docker Hub)|
|**Cấu trúc**|Registry → Repository → Image → Tag|
|**Public vs Private**|Public: ai cũng pull được Private: phải có quyền mới pull/push|
|**Bảo mật**|Tích hợp IAM + Image Scanning (Basic & Enhanced với Inspector)|
|**Giám sát**|CloudWatch + CloudTrail + EventBridge|
|**Tính năng nổi bật**|Cross-region & Cross-account replication|

---

### Lời khuyên học tập

- Phần lý thuyết ECR khá ngắn gọn vì **AWS muốn bạn thực hành nhiều hơn**.
- Bạn sẽ được thực hành **push image lên ECR** và **pull image** khi triển khai ECS trong các bài lab sau.
- Với kỳ thi **Solutions Architect Associate**, bạn cần nắm rõ:
    - Sự khác biệt giữa **Public** và **Private** registry.
    - ECR tích hợp với **IAM**, **Inspector**, **CloudWatch**, **CloudTrail**, **EventBridge**.
    - Hỗ trợ replication giữa region và account.

---

Bạn đã sẵn sàng chuyển sang phần **thực hành (demo)** với ECR chưa? Nếu muốn, tôi có thể tóm tắt lại toàn bộ phần **ECS + ECR** thành một bảng so sánh tổng hợp để dễ ôn tập hơn.