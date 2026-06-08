**Hướng dẫn Bootstrapping EC2 Instance bằng User Data (Phần 1)**

Chào mừng trở lại! Trong bài học demo này, bạn sẽ được trải nghiệm cách **khởi tạo tự động (bootstrapping)** một phiên bản **EC2** bằng **User Data**.

Đây là khả năng chạy script trong quá trình cung cấp (provisioning) EC2 instance, giúp **tự động cấu hình instance ngay từ đầu** mà không cần tạo Amazon Machine Image (AMI) tùy chỉnh.

**Lưu ý quan trọng**: Tạo AMI rất nhanh nhưng cấu hình bị “nướng cứng”. Với **User Data**, bạn linh hoạt hơn rất nhiều vì có thể thay đổi cấu hình mỗi lần launch.

### 1. Tạo Animals for Life VPC bằng CloudFormation

Trước tiên, chúng ta cần tạo VPC cho bài lab.

- **Đăng nhập** bằng IAM Admin User.
- Chọn region **Northern Virginia (us-east-1)**.
- Mở link **One-click deployment** đính kèm bài học.
- **Stack name**: bootstrap (đã điền sẵn).
- Giữ nguyên các giá trị mặc định.
- **Đánh dấu** vào ô **Capabilities**.
- Nhấp **Create Stack**.

**Chờ** đến khi stack chuyển sang trạng thái **CREATE_COMPLETE** thì mới tiếp tục.

### 2. Tìm hiểu nội dung User Data

- Mở link **User Data** đính kèm bài học → tải file userdata.txt về máy.
    
- Mở file bằng trình soạn thảo. Bạn sẽ thấy **toàn bộ các lệnh** đã từng thực hiện thủ công trước đây, bao gồm:
    
    - Cài đặt MariaDB, Apache, wget, cowsay.
    - Cài đặt PHP và các thư viện cần thiết.
    - Thiết lập dịch vụ tự động khởi động.
    - Đặt root password cho MariaDB.
    - Tải và giải nén WordPress.
    - Cấu hình file wp-config.php.
    - Sửa quyền thư mục web root.
    - Tạo database và user cho WordPress.
    - Cấu hình **custom login banner** bằng cowsay.

Tất cả các bước thủ công trước đây giờ được **đóng gói thành một script duy nhất**.

### 3. Khởi chạy EC2 Instance với User Data

Quay lại **EC2 Console** → nhấp **Launch Instance**.

Thực hiện theo các bước sau:

1. **Name**: Nhập a4l-manual-wordpress.
2. **Application and OS Images**: Chọn **Amazon Linux 2023 (64-bit x86)**.
3. **Instance type**: Chọn loại **Free Tier eligible** (thường là t2.micro).
4. **Key pair**: Chọn **Proceed without a key pair**.
5. **Network settings** → nhấp **Edit**:
    - **VPC**: Chọn a4l-vpc1.
    - **Subnet**: Chọn sn-web-a.
    - **Auto-assign public IP**: Chọn **Enable**.
    - **Firewall (security groups)**: Chọn bootstrap-instance.
6. **Storage**: Giữ nguyên mặc định.
7. **Advanced details** → mở rộng:
    - Tìm ô **User Data**.
    - Dán **toàn bộ nội dung** file userdata.txt vào ô này.
    - **Không tick** ô “As base64 encoded” (vì dán text thường).

Nhấp **Launch Instance**.

### 4. Kết nối và kiểm tra kết quả

- Chờ instance chuyển sang trạng thái **Running** thêm **2–3 phút** để quá trình bootstrapping hoàn tất.
- Right-click vào instance → **Connect** → chọn **EC2 Instance Connect**.
- **User**: ec2-user → nhấp **Connect**.

Bạn sẽ thấy ngay **custom Animals for Life login banner** — dấu hiệu script User Data đã chạy thành công.

Mở trình duyệt và truy cập **Public IP** của instance (dùng HTTP). Bạn sẽ thấy trang cài đặt **WordPress** xuất hiện.

### 5. Kiểm tra cách User Data hoạt động

**Xem User Data đã gửi vào instance (IMDSv2)** (Vì dùng Amazon Linux 2023, cần dùng Instance Metadata Service v2):

Bash

```
# Lấy token
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`

# Xem User Data
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/user-data
```

Bạn sẽ thấy lại **toàn bộ script** đã dán lúc launch.

**Xem log để chẩn đoán** Di chuyển vào thư mục log:

Bash

```
cd /var/log
ls -la
```

Hai file log quan trọng nhất:

- cloud-init.log
- cloud-init-output.log ← **file quan trọng nhất**

Xem nội dung:

Bash

```
sudo cat /var/log/cloud-init-output.log
```

Trong file này bạn sẽ thấy **toàn bộ lệnh** đã được thực thi theo thứ tự (cài MariaDB, Apache, PHP, WordPress, tạo database, sửa quyền, cấu hình banner…).

---

**Kết thúc Phần 1** Bài học này khá dài nên tôi chia thành 2 phần để bạn dễ theo dõi. **Phần 2** sẽ tiếp nối ngay sau.

Hãy tạm dừng ở đây, nghỉ ngơi một chút, sau đó quay lại khi sẵn sàng để tiếp tục phần sau.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Bootstrapping EC2 bằng User Data
- Tạo VPC qua CloudFormation (stack bootstrap)
- Script userdata.txt chứa toàn bộ cài đặt thủ công
- Launch Instance với User Data (Amazon Linux 2023)
- Kiểm tra qua EC2 Instance Connect, Public IP WordPress, IMDSv2
- Log quan trọng: cloud-init-output.log

**Notes (Chi tiết):**

- **User Data** cho phép chạy script tự động ngay khi instance khởi tạo, linh hoạt hơn AMI.
- Script bao gồm cài web server, database, WordPress và custom banner bằng cowsay.
- Sử dụng **VPC a4l-vpc1**, subnet **sn-web-a**, security group **bootstrap-instance**.
- Sau launch, chờ 2-3 phút, kết nối Instance Connect để xem banner và truy cập HTTP Public IP.
- Kiểm tra User Data và log cloud-init để debug quá trình bootstrapping.

**Summary (Tóm tắt ngắn):** Phần 1 hướng dẫn **bootstrapping EC2 thủ công** bằng **User Data script** (userdata.txt) sau khi tạo VPC qua CloudFormation. Thực hiện launch instance, kiểm tra kết quả qua banner, WordPress và log cloud-init. Chuẩn bị cho Phần 2.