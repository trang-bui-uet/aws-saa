**Chào mừng bạn quay trở lại** và chào mừng đến với bài demo này. Cùng nhau chúng ta sẽ **cài đặt CloudWatch Agent** để thu thập và gửi log từ **ba file log** khác nhau vào **CloudWatch Logs**, đồng thời giúp chúng ta có quyền truy cập một số **metrics** bên trong hệ điều hành mà trước đây không thể nhìn thấy. Đây sẽ là demo rất tốt để thể hiện sức mạnh của **CloudWatch** và **CloudWatch Logs** khi kết hợp với **CloudWatch Agent**.

Để thực hiện demo này, bạn cần triển khai hạ tầng. Hãy đảm bảo bạn đã đăng nhập vào **management account** của Organization và chọn vùng **Northern Virginia (us-east-1)**. Bài học có đính kèm **URL one-click deployment** để triển khai hạ tầng cần dùng. Click vào link đó → màn hình **Create stack** sẽ hiện ra. Tên stack sẽ là **cwagent**. Scroll xuống dưới cùng, tick vào ô acknowledge capabilities, rồi click **Create stack**.

Ngoài ra, có tài liệu **lesson commands** chứa toàn bộ lệnh cần dùng trong demo. Hãy mở tài liệu đó trong tab mới.

Bạn cần chờ **CloudFormation stack** chuyển sang trạng thái **CREATE_COMPLETE** trước khi tiếp tục. Hãy pause video, chờ trạng thái thay đổi, rồi tiếp tục.

---

Khi stack đã **CREATE_COMPLETE**, chúng ta bắt đầu demo. Trong bài này, chúng ta sẽ cài **CloudWatch Agent** trên một EC2 instance đã được provision bởi stack.

**Bước 1:** Chuyển sang **EC2 console** → chọn **Instances running** → bạn sẽ thấy instance tên **A4L-WordPress**. Right-click vào instance → chọn **Connect** → dùng **EC2 Instance Connect** (username: ec2-user) → Connect.

Bạn sẽ thấy **Animals for Life custom login banner** nếu mọi thứ hoạt động bình thường.

---

Demo này gồm nhiều bước:

- Tải và cài đặt **CloudWatch Agent**
- Tạo file cấu hình
- Gán **IAM Role** cho instance
- Chạy wizard cấu hình agent
- Cấu hình thu thập 3 file log
- Lưu cấu hình vào **Parameter Store**

Trước tiên, chúng ta cần gán **IAM Role** cho instance vì agent cần quyền tương tác với **CloudWatch Logs** và **Parameter Store**.

**Tạo IAM Role:**

- Vào **IAM console** → **Roles** → **Create role**
- Chọn **AWS service** → **EC2** → Next
- Gắn 2 chính sách:
    - **CloudWatchAgentServerPolicy**
    - **AmazonSSMFullAccess**
- Đặt tên role là **CloudWatchRole** → **Create role**

**Gắn role vào instance:**

- Quay lại **EC2 console** → Right-click instance **A4L-WordPress** → **Security** → **Modify IAM role**
- Chọn **CloudWatchRole** → **Update IAM role**

---

Bây giờ kết nối lại vào instance (nếu tab cũ timeout thì đóng và mở lại).

Chạy lệnh **CloudWatch Agent configuration wizard** (lệnh có trong lesson commands).

Trong wizard, hầu hết chọn giá trị mặc định, nhưng có một số chỗ cần chú ý:

- Chọn **3** cho **Advanced** metric collection (để thu thập thêm OS metrics).
    
- Chọn **Yes** để monitor log files.
    
- Thêm **3 file log** sau:
    
    1. **/var/log/secure** → Log group name: **/var/log/secure** → Log stream: dùng **Instance ID** (mặc định)
    2. **/var/log/httpd/access_log** → Log group name: **/var/log/httpd/access_log**
    3. **/var/log/httpd/error_log** → Log group name: **/var/log/httpd/error_log**

Sau khi thêm xong 3 file log, chọn **No** để không thêm log nào nữa.

---

Wizard sẽ hỏi có muốn lưu cấu hình vào **Parameter Store** không. **Mặc định là có** → nhấn Enter.

- Parameter name mặc định: **AmazonCloudWatch-linux** (giữ nguyên).
- Region: tự động nhận **us-east-1**.
- Credentials: lấy từ IAM Role đã gắn trước đó.

Sau khi hoàn tất, cấu hình JSON sẽ được lưu vào **Parameter Store**.

Bạn có thể kiểm tra:

- Vào **Systems Manager** → **Parameter Store**
- Bạn sẽ thấy parameter tên **AmazonCloudWatch-linux**
- Giá trị là một JSON document chứa toàn bộ cấu hình của **CloudWatch Agent**.

**Lợi ích:** Bạn chỉ cần tạo cấu hình một lần và lưu vào Parameter Store. Sau này khi triển khai hàng loạt EC2 instance (ví dụ qua Auto Scaling Groups), bạn có thể sử dụng lại cấu hình này một cách an toàn và dễ dàng.

---

Đây là **phần 1** của bài demo. Phần 2 sẽ tiếp nối ngay sau. Bạn có thể nghỉ ngơi hoặc uống cà phê, sau đó quay lại tiếp tục.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- One-click CloudFormation stack (cwagent)
- EC2 Instance: A4L-WordPress
- Tạo IAM Role: CloudWatchRole (CloudWatchAgentServerPolicy + AmazonSSMFullAccess)
- CloudWatch Agent configuration wizard
- Cấu hình 3 log files: /var/log/secure, access_log, error_log
- Lưu config vào Parameter Store (AmazonCloudWatch-linux)
- Log Group & Log Stream theo Instance ID

**Notes (Chi tiết):**

- Triển khai hạ tầng bằng **CloudFormation one-click deployment** (stack name: cwagent).
- Kết nối instance qua **EC2 Instance Connect**.
- Tạo **IAM Role** “CloudWatchRole” và gắn vào instance để cấp quyền cho agent.
- Chạy **configuration wizard** của CloudWatch Agent:
    - Chọn **Advanced** (3) để thu thập thêm OS metrics.
    - Thêm 3 log file với **full path** làm Log Group name.
    - Log Stream mặc định dùng **Instance ID**.
- Lưu cấu hình agent dưới dạng parameter **AmazonCloudWatch-linux** trong **Parameter Store**.
- Ưu điểm: Tái sử dụng cấu hình dễ dàng khi scale với Auto Scaling Groups.

**Summary (Tóm tắt ngắn):** Bài demo Part 1 hướng dẫn triển khai hạ tầng, tạo IAM Role, chạy **CloudWatch Agent configuration wizard**, cấu hình thu thập 3 file log quan trọng (/var/log/secure, Apache access & error log), và lưu cấu hình vào **Parameter Store** để dễ dàng tái sử dụng ở quy mô lớn. Phần 2 sẽ tiếp tục với việc khởi động agent và kiểm tra kết quả.