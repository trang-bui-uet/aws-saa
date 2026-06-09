**Chào mừng bạn quay trở lại** và chào mừng đến với **phần 2** của bài demo. Chúng ta sẽ tiếp tục ngay từ phần 1.

Hãy quay lại terminal của instance. Nhấn Enter vài lần để chắc chắn tab chưa timeout.

---

**Bước sửa lỗi nhỏ (bug fix):** **CloudWatch Agent** yêu cầu phần mềm **collectd** phải được cài đặt. Trên Amazon Linux, collectd chưa có sẵn. Chúng ta cần thực hiện 2 lệnh sau:

1. Tạo thư mục mà agent mong đợi.
2. Tạo file database mà agent mong đợi.

Sau khi chạy xong 2 lệnh này, chúng ta sẵn sàng bước tiếp theo.

---

**Bước cuối cùng:** Khởi động **CloudWatch Agent** và cung cấp cấu hình từ **Parameter Store**.

Chạy lệnh sau (có trong lesson commands):

Lệnh này sẽ:

- Khởi động **amazon-cloudwatch-agent-ctl**
- Lấy cấu hình từ **Parameter Store** (qua ssm:AmazonCloudWatch-linux)
- Cấu hình agent theo file JSON đã lưu
- Bắt đầu thu thập và gửi log vào **CloudWatch Logs**

Nhờ IAM Role đã gắn trước đó, agent có đủ quyền để thực hiện việc này.

---

**Kiểm tra kết quả trong CloudWatch:**

1. Vào **CloudWatch console** → **Log groups**.
2. Scroll xuống dưới cùng, bạn sẽ thấy các log group chúng ta đã cấu hình:
    - **/var/log/secure**
    - **/var/log/httpd/access_log**
    - **/var/log/httpd/error_log**

Lúc đầu có thể chỉ thấy **error_log**. Để tạo dữ liệu cho **access_log**, hãy:

- Quay lại **EC2 console** → copy **Public IPv4 address** của instance.
- Mở địa chỉ IP đó trong tab mới (truy cập website) → điều này sẽ tạo traffic cho Apache và ghi log vào access_log.

Sau khi refresh **CloudWatch Logs**, bạn sẽ thấy đầy đủ 3 log group.

Mở từng **Log Stream** (theo Instance ID):

- **access_log**: Ghi lại các truy cập thành công đến web server.
- **error_log**: Ghi lại các lỗi (trang không tồn tại, lỗi server, lỗi module…).
- **/var/log/secure**: Ghi lại các sự kiện đăng nhập an toàn.

---

**Khám phá Metrics mới từ CloudWatch Agent:**

Vì đã cài **CloudWatch Agent**, bạn sẽ có thêm namespace **CWAgent** trong **CloudWatch Metrics**.

Các metrics mới bao gồm:

- **Disk IO** (đọc/ghi)
- **CPU per core** (user, system, iowait…)
- Memory, swap, network… ở mức chi tiết hệ điều hành

Đây là những chỉ số **bên trong OS** mà trước đây bạn không thể thấy chỉ với CloudWatch thông thường.

Hãy dành thời gian khám phá các metric và log group mới này.

---

**Dọn dẹp (Cleanup):**

1. **EC2 console** → Right-click instance → **Security** → **Modify IAM role** → Gỡ bỏ **CloudWatchRole**.
2. **IAM console** → **Roles** → Chọn **CloudWatchRole** → **Delete role**.
3. **Parameter Store** → Giữ lại parameter **AmazonCloudWatch-linux** (vì là Standard tier, không mất phí và sẽ dùng lại trong các bài sau).
4. **CloudFormation console** → Chọn stack **cwagent** → **Delete** → Xác nhận xóa.

Sau khi hoàn tất, toàn bộ hạ tầng demo sẽ bị xóa và tài khoản trở về trạng thái ban đầu.

---

**Kết luận bài demo:**

Bài demo này hướng dẫn cách **cài đặt thủ công CloudWatch Agent** trên EC2 instance. Trong thực tế, bạn có thể tự động hóa bằng **Systems Manager**, bake vào AMI, hoặc bootstrap khi launch instance.

**CloudWatch Agent** sẽ được sử dụng xuyên suốt các bài demo sau trong khóa học để thu thập metrics và logs chi tiết.

Đây là toàn bộ nội dung tôi muốn bạn thực hiện trong bài demo này. Xem hết video và sẵn sàng sang bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Bug fix: collectd directory + database file
- Khởi động agent: amazon-cloudwatch-agent-ctl + ssm:AmazonCloudWatch-linux
- Kiểm tra Log Groups: /var/log/secure, access_log, error_log
- Tạo traffic để sinh access_log
- CWAgent namespace trong Metrics (Disk IO, CPU per core…)
- Cleanup: Gỡ IAM Role, xóa Role, giữ Parameter, xóa CloudFormation stack

**Notes (Chi tiết):**

- Trước khi start agent, cần tạo thư mục và file database cho **collectd** trên Amazon Linux.
- Lệnh khởi động agent lấy config trực tiếp từ **Parameter Store** (ssm:AmazonCloudWatch-linux).
- Sau khi agent chạy, log xuất hiện trong **CloudWatch Log Groups** (mỗi file log = 1 Log Group, mỗi instance = 1 Log Stream).
- Truy cập website qua Public IP để sinh dữ liệu **access_log**.
- **CWAgent** mang lại metrics chi tiết cấp OS (Disk IO, CPU per core, Memory…).
- **Cleanup** giữ lại Parameter Store config vì sẽ tái sử dụng sau này.

**Summary (Tóm tắt ngắn):** Phần 2 hoàn tất việc khởi động **CloudWatch Agent** bằng cấu hình từ Parameter Store. Sau khi agent chạy, log từ 3 file được gửi vào CloudWatch Logs và xuất hiện thêm namespace **CWAgent** với nhiều metrics chi tiết cấp hệ điều hành. Bài demo kết thúc bằng quy trình dọn dẹp (giữ lại Parameter để dùng sau). Đây là cách thủ công cài CloudWatch Agent, sẽ được tự động hóa trong các bài sau.