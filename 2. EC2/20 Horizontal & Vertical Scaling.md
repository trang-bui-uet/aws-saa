**Hiểu về Scaling và Các Loại Scaling**

**Chào mừng bạn quay lại.**

Bài học này chúng ta sẽ tập trung vào **lý thuyết** một chút. Đây là phần kiến thức rất quan trọng từ bây giờ trở đi trong khóa học, vì các chủ đề và ví dụ sắp tới sẽ ngày càng phức tạp hơn.

**Horizontal Scaling** (mở rộng ngang) và **Vertical Scaling** (mở rộng dọc) là **hai cách hoàn toàn khác nhau** để hệ thống có thể mở rộng (scale) nhằm xử lý tải (load) tăng hoặc đôi khi giảm từ người dùng.

Hãy cùng phân tích sự khác biệt, ưu nhược điểm và yêu cầu của từng loại.

### Scaling là gì?

**Scaling** xảy ra khi hệ thống cần **tăng hoặc giảm** quy mô để đáp ứng tải từ khách hàng. Về mặt kỹ thuật, bạn đang **thêm hoặc bớt tài nguyên** cho hệ thống. Hệ thống có thể chỉ là **một EC2 instance**, nhưng cũng có thể là hàng trăm, hàng nghìn, thậm chí hàng chục nghìn instance.

### Vertical Scaling (Mở rộng dọc – Scale Up)

Đây là cách **tăng tài nguyên cho một instance duy nhất**.

**Ví dụ**: Ứng dụng đang chạy trên instance **t3.large** (2 vCPU, 8 GiB RAM). Khi tải tăng cao, instance này không còn đủ sức xử lý → khách hàng gặp chậm, lag, hoặc crash.

**Giải pháp**: Resize instance lên instance lớn hơn

- t3.xlarge (4 vCPU, 16 GiB)
- t3.2xlarge (8 vCPU, 32 GiB) → Tăng gấp đôi tài nguyên.

#### Điểm quan trọng & Hạn chế của Vertical Scaling

- **Downtime**: Khi resize instance, EC2 **phải restart** → gây gián đoạn dịch vụ cho khách hàng.
- **Maintenance Windows**: Chỉ có thể scale trong **khoảng thời gian bảo trì đã thỏa thuận** (outage window). Nếu tải thay đổi đột ngột, bạn không thể phản ứng nhanh.
- **Chi phí không hiệu quả**: Instance càng lớn thì giá càng cao **không tuyến tính** (càng to càng đắt hơn nhiều).
- **Giới hạn tối đa**: Mỗi instance có **kích thước tối đa** (maximum instance size). Dù AWS liên tục cải tiến, vẫn luôn tồn tại giới hạn phần cứng.

#### Lợi ích của Vertical Scaling

- **Rất đơn giản**.
- **Không cần sửa đổi ứng dụng**: Ứng dụng chạy được trên instance nhỏ thì cũng chạy được trên instance lớn hơn.
- **Hoạt động với mọi loại ứng dụng**, kể cả **monolithic** (toàn bộ code nằm trong một ứng dụng duy nhất).

### Horizontal Scaling (Mở rộng ngang – Scale Out)

Đây là cách **thêm nhiều instance** thay vì làm một instance to hơn.

Khi tải tăng:

- 1 instance → 2 instance → 4 instance → 8 instance… → Mỗi instance vẫn giữ kích thước nhỏ và **phân tải đều**.

**Yêu cầu quan trọng**: Phải có **Load Balancer** đứng trước để phân phối traffic đều cho các instance.

#### Tầm quan trọng của Session Management

Khi scale ngang, **session** là vấn đề then chốt.

- Session = trạng thái phiên làm việc của người dùng (đăng nhập, giỏ hàng, vị trí video…).
- Với **1 instance**: session lưu trên instance đó → ổn.
- Với **nhiều instance**: nếu session lưu trên từng instance riêng lẻ, người dùng sẽ bị **logout** mỗi khi chuyển instance.

**Giải pháp**: Sử dụng **off-host sessions** (lưu session bên ngoài)

- Lưu vào **database bên ngoài** (ElastiCache, DynamoDB…).
- Các instance trở thành **stateless** (không lưu trạng thái).

#### Lợi ích của Horizontal Scaling

- **Không gián đoạn khi scale**: Thêm/bớt instance mà khách hàng **không bị ảnh hưởng**.
- **Không có giới hạn thực sự**: Có thể thêm bao nhiêu instance tùy ý.
- **Tiết kiệm chi phí**: Dùng nhiều instance nhỏ (commodity) thay vì instance lớn đắt tiền.
- **Scale chi tiết hơn (Granular Scaling)**: Thêm 1 instance nhỏ chỉ tăng ~20% tài nguyên (thay vì nhân đôi như vertical).

### Tóm tắt & Hình ảnh minh họa (Power-up cho kỳ thi)

Để dễ nhớ sự khác biệt:

- **Horizontal Scaling** = Thêm nhiều **Bob** → 1 Bob → 2 Bob → 4 Bob → … → **Vô hạn Bob** (không giới hạn).
- **Vertical Scaling** = Làm **một Bob to hơn** → Small Bob → Medium Bob → Large Bob.

Khi thi, chỉ cần nhớ hình ảnh **“nhiều Bob” vs “một Bob to”** là bạn sẽ không bao giờ nhầm lẫn.

---

**Kết thúc bài học lý thuyết.** Phần còn lại sẽ được học sâu hơn trong chương **High Availability & Scaling**.

Hoàn thành video và hẹn gặp bạn ở bài học tiếp theo! 🚀