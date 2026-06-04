**Chào mừng bạn quay trở lại.**

Phần này sẽ tập trung vào một loại **điện toán khác** — đó là **container computing** (điện toán dựa trên container).

Để hiểu rõ lợi ích của các sản phẩm và dịch vụ AWS liên quan đến container, bạn cần nắm được **container là gì** và **container computing mang lại những lợi ích gì**.

Bài học này hoàn toàn là **lý thuyết**, nhưng ngay sau đó sẽ là bài **demo thực hành** để bạn tự tay tạo container của mình. Chúng ta có rất nhiều nội dung cần học, nên hãy bắt đầu ngay!

### 1. Bối cảnh: Virtualization (Ảo hóa) thực chất là gì?

Trước khi nói về container, hãy làm rõ khái niệm **virtualization**.

Thực tế, những gì ta thường gọi là **virtualization** nên gọi chính xác là **OS virtualization** (ảo hóa hệ điều hành). Đây là quá trình chạy **nhiều hệ điều hành** trên cùng một phần cứng vật lý.

**Ví dụ kiến trúc AWS**: Một máy chủ EC2 chạy **Nitro hypervisor**, trên đó có nhiều **virtual machines (máy ảo)**.

Mỗi máy ảo là **một hệ điều hành hoàn chỉnh** kèm theo tài nguyên (RAM, ổ đĩa…).

**Vấn đề lớn**: Hệ điều hành khách (guest OS) thường chiếm **60–70%** dung lượng ổ đĩa và rất nhiều bộ nhớ. Chỉ còn lại rất ít tài nguyên cho ứng dụng và môi trường runtime.

Hơn nữa, hầu hết các máy ảo trong doanh nghiệp thường dùng **cùng một hệ điều hành** (Windows hoặc Linux cùng phiên bản).

→ Đây chính là **sự trùng lặp (duplication)** và **lãng phí tài nguyên nghiêm trọng**. Mọi thao tác khởi động, tắt, restart đều phải xử lý **toàn bộ hệ điều hành**.

**Câu hỏi quan trọng**: Chúng ta chỉ muốn chạy **6 ứng dụng** trong các môi trường **riêng biệt, cách ly và được bảo vệ**. Liệu có cần **6 bản sao hệ điều hành** không?

**Câu trả lời: KHÔNG** — khi sử dụng **container**.

### 2. Container hoạt động khác biệt như thế nào?

**Containerization** (cách ly bằng container) xử lý mọi thứ hoàn toàn khác:

- Vẫn có **phần cứng chủ (host hardware)**
- Trên đó chạy **một hệ điều hành duy nhất**
- Trên hệ điều hành là **container engine** (phổ biến nhất là **Docker**)

**Container** giống máy ảo ở chỗ nó cung cấp **môi trường cách ly** cho ứng dụng.

**Nhưng khác biệt rất lớn**:

- Máy ảo → Chạy **toàn bộ hệ điều hành** trên hypervisor
- Container → Chỉ là **một process** chạy trong hệ điều hành chủ

Container sử dụng kernel và tài nguyên của **host OS** (networking, file I/O…) nên cực kỳ **nhẹ**.

**Kiến trúc thực tế**: Một container chỉ chiếm tài nguyên cho **ứng dụng + thư viện + dependencies**. Hệ điều hành chủ có thể chạy **hàng trăm container** cùng lúc.

**Lợi ích nổi bật nhất**: **Mật độ cao (higher density)** — chạy được **nhiều ứng dụng hơn rất nhiều** trên cùng một phần cứng so với máy ảo.

### 3. Docker Image và Docker Container

Một **EC2 instance** là bản chạy của các EBS volume.

Tương tự: **Container = bản chạy của Docker Image**

**Docker Image** là công nghệ đặc biệt vì nó được tạo từ **nhiều layer độc lập** (không phải một đĩa đơn khối).

**Cách tạo Image**:

- Sử dụng **Dockerfile**
- Mỗi lệnh trong Dockerfile tạo ra **một filesystem layer mới**
- Bắt đầu từ **base image** (ví dụ: CentOS 7)
- Sau đó thêm cập nhật, cài phần mềm (Apache), thêm script…

Mỗi layer chỉ chứa **sự thay đổi (differential)** so với layer bên dưới. Tất cả các layer đều là **read-only** (chỉ đọc).

### 4. Sự khác biệt quan trọng giữa Image và Container

- **Docker Image** = tập hợp các layer **read-only**
- **Docker Container** = Image + **một layer read-write** ở trên cùng

Mọi thay đổi trong container (log, file tạm, dữ liệu…) đều nằm ở layer read-write.

**Điểm siêu hay**: Nhiều container có thể **chia sẻ chung** các layer read-only. Chỉ khác nhau ở layer read-write → Tiết kiệm **rất lớn** dung lượng ổ đĩa.

Dù có 200 container dùng cùng một image, chúng vẫn chỉ dùng chung các layer cơ bản.

### 5. Container Registry

**Container Registry** là nơi lưu trữ và phân phối Docker Image. Phổ biến nhất: **Docker Hub** (có thể public hoặc private).

Bạn có thể:

- Push image của mình lên
- Pull base image từ vendor (CentOS, Ubuntu, Amazon Linux…)

### 6. Các khái niệm quan trọng & lợi ích chính của Container

**Lợi ích nổi bật**:

- **Nhẹ (lightweight)** và **tiết kiệm tài nguyên**
- **Khởi động / dừng siêu nhanh**
- **Portable** cao: “**Build once, run anywhere**”
- **Tính nhất quán (consistency)**: Chạy giống hệt trên mọi môi trường
- **Mật độ cao**: Chạy nhiều ứng dụng hơn trên cùng hardware
- **Cách ly tốt** mà không cần full OS
- Hỗ trợ **expose port** (ví dụ: port 80 cho HTTP)
- Một ứng dụng phức tạp có thể gồm **nhiều container** (database container + app container…)

**Tóm tắt so sánh nhanh**:

|Tiêu chí|**Virtual Machine**|**Container**|
|---|---|---|
|Kích thước|Lớn (hàng GB)|Rất nhỏ (hàng MB)|
|Tốc độ start/stop|Chậm|**Siêu nhanh**|
|Tài nguyên chiếm dụng|Toàn bộ OS|Chỉ ứng dụng + thư viện|
|Mật độ trên hardware|Thấp|**Rất cao**|
|Portable|Trung bình|**Rất cao**|

---

**Kết thúc phần lý thuyết.**

Bạn đã nắm được **cách container hoạt động** và **lợi ích vượt trội** so với máy ảo.

Bây giờ hãy chuyển sang **bài học demo** để tự tay tạo Docker Image và Container nhé!

Sẵn sàng chưa? Hãy tiếp tục bài học tiếp theo! 🚀