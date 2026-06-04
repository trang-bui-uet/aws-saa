### Giới thiệu

Trong bài học này, chúng ta sẽ tìm hiểu **hai chế độ cluster** chính khi chạy container trên **Amazon ECS**:

- **EC2 Mode**
- **Fargate Mode**

Chế độ cluster quyết định **mức độ quản lý** mà bạn phải chịu trách nhiệm so với mức độ AWS quản lý. Đây là điểm khác biệt quan trọng nhất giữa hai chế độ. Ngoài ra, còn có sự khác biệt về chi phí và các tình huống sử dụng phù hợp với từng chế độ.

---

### 1. Chế độ EC2 Mode

**EC2 Mode** là chế độ truyền thống của ECS, trong đó bạn **tự quản lý** các EC2 instance đóng vai trò là container host.

#### Kiến trúc EC2 Mode

- ECS vẫn chịu trách nhiệm các thành phần quản lý cấp cao: **scheduling**, **orchestration**, **cluster management** và **placement engine**.
- Cluster được tạo trong **VPC** của bạn và tận dụng nhiều **Availability Zone** (AZ).
- Bạn chỉ định số lượng EC2 instance ban đầu → ECS sử dụng **Auto Scaling Group** để quản lý số lượng container instances.
- Các container image được lưu tại **Container Registry** (ECR hoặc Docker Hub).
- Task và Service định nghĩa cách triển khai container lên các EC2 instance.

#### Những gì bạn phải quản lý

- **Toàn bộ EC2 instances** (container hosts)
- Dung lượng (capacity) và tính sẵn sàng (availability) của cluster
- Scaling, vá lỗi, bảo trì các EC2 instance
- **Bạn phải trả tiền cho EC2 instances** dù có chạy container hay không (miễn là instance đang chạy)

**EC2 Mode không phải là giải pháp serverless.** Bạn vẫn phải lo về capacity planning và quản lý hạ tầng.

#### Ưu điểm của EC2 Mode

- Phù hợp nếu bạn đã có **Reserved Instance** hoặc muốn sử dụng **Spot Instance** để tiết kiệm chi phí.
- Chi phí thường thấp hơn Fargate nếu workload lớn và ổn định.
- Bạn có toàn quyền kiểm soát container host.

#### Nhược điểm

- Phải quản lý server → tốn effort vận hành.
- Vẫn phải trả tiền cho EC2 instances ngay cả khi không chạy task nào.

---

### 2. Chế độ Fargate Mode

**Fargate Mode** là chế độ **serverless** của ECS. AWS quản lý hoàn toàn hạ tầng chạy container.

#### Kiến trúc Fargate Mode

- Bạn **không còn thấy EC2 instances** nào cả.
- AWS duy trì một **hạ tầng Fargate chia sẻ** (shared infrastructure platform) cho tất cả khách hàng.
- Task và Service vẫn được định nghĩa như bình thường (image, port, CPU, memory…).
- Khi chạy task, AWS sẽ **inject** task vào **VPC** của bạn thông qua **Elastic Network Interface (ENI)**.
- Mỗi task có IP riêng trong VPC của bạn và hoạt động như một tài nguyên VPC thông thường.

#### Điểm khác biệt quan trọng

- **Không quản lý server**: Bạn không thấy, không quản lý, không patch EC2 instance nào.
- **Chỉ trả tiền cho những gì bạn dùng**: Bạn chỉ bị tính phí theo tài nguyên CPU và memory mà container thực tế tiêu thụ.
- Vẫn sử dụng đầy đủ tính năng của VPC (public/private subnet, security group, NAT Gateway…).

**Fargate là lựa chọn serverless thực thụ** trong ECS.

---

### 3. So sánh EC2 Mode và Fargate Mode

|Tiêu chí|**EC2 Mode**|**Fargate Mode**|**Ghi chú**|
|---|---|---|---|
|Quản lý server|Bạn tự quản lý EC2 instances|AWS quản lý hoàn toàn|Fargate thắng thế|
|Chi phí|Trả tiền EC2 instances (dù có chạy container hay không)|Chỉ trả theo tài nguyên container thực tế sử dụng|EC2 rẻ hơn nếu workload lớn & ổn định|
|Quản lý overhead|Cao hơn|Rất thấp|Fargate phù hợp team nhỏ hoặc ít nhân lực vận hành|
|Workload lớn, ổn định|Phù hợp (đặc biệt dùng Spot/Reserved Instance)|Vẫn dùng được nhưng đắt hơn|-|
|Workload nhỏ, burst, batch|Không hiệu quả (phải trả tiền instance liên tục)|Rất phù hợp|Fargate thắng thế|
|Cần kiểm soát sâu hạ tầng|Có thể (tự quản lý EC2)|Hạn chế|-|
|Production recommendation|Phù hợp nếu có team vận hành mạnh|Khuyến khích cho hầu hết các trường hợp mới|-|

---

### 4. Khi nào nên chọn chế độ nào?

#### Chọn **EC2 Mode** khi:

- Workload **lớn và ổn định** (consistent workload)
- Bạn là tổ chức **nhạy cảm về chi phí** và đã có Reserved Instance hoặc muốn dùng Spot Instance
- Bạn cần kiểm soát sâu hạ tầng container host
- Bạn sẵn sàng bỏ effort để quản lý, scaling và bảo trì cluster

#### Chọn **Fargate Mode** khi:

- Muốn **giảm thiểu overhead vận hành** tối đa
- Workload **nhỏ, không ổn định, bursty** hoặc chạy theo **batch/periodic**
- Đội ngũ chưa có nhiều kinh nghiệm vận hành container host
- Muốn triển khai nhanh và tập trung vào business logic thay vì quản lý server
- Chi phí EC2 instances nhàn rỗi là vấn đề lớn

#### Tóm tắt nhanh để nhớ:

- **EC2 Mode** = **Bạn quản lý server** → Tiết kiệm chi phí nhưng tốn effort.
- **Fargate Mode** = **AWS quản lý server** → Dễ dàng hơn nhưng chi phí có thể cao hơn với workload lớn.

---

### Lời khuyên cho kỳ thi và thực tế

Khi được hỏi nên dùng ECS hay EC2 thuần túy:

- Nếu ứng dụng đã được **containerize** → Nên dùng **ECS**.
- Nếu chỉ test nhanh container → Có thể dùng EC2 + Docker.
- Production thực tế: Hầu như không nên chạy container trực tiếp trên EC2 thủ công.

---

**Tóm lại:**

- **EC2 Mode**: Linh hoạt, kiểm soát cao, phù hợp workload lớn và tiết kiệm chi phí dài hạn (nhưng phải quản lý server).
- **Fargate Mode**: Serverless, ít quản lý, phù hợp workload không ổn định và ưu tiên tốc độ triển khai.

Bài học tiếp theo sẽ là **demo thực tế** chạy container “Container of Cats” trên **ECS Fargate**. Đây là phần rất quan trọng để bạn nắm vững kiến trúc và dễ dàng trả lời câu hỏi trong kỳ thi AWS Solutions Architect Associate.