Chào mừng đến bài học demo này nơi bạn sẽ có được kinh nghiệm cấu hình **failover routing** cũng như **private hosted zones**. Với bài học demo này, bạn có thể lựa chọn theo dõi thực hành trong môi trường AWS của riêng mình hoặc chỉ xem tôi thực hiện các bước.

Nếu bạn muốn thực hành theo, bạn **cần có một domain name đã đăng ký trong Route 53** (đây là bước tùy chọn ở đầu khóa học). Trong trường hợp của tôi, tôi đã đăng ký **animalsforlife1337.org**. Nếu bạn đã đăng ký domain, hãy **thay thế animalsforlife1337.org** bằng domain của bạn ở mọi nơi xuất hiện trong bài học. Nếu bạn chưa đăng ký domain, bạn chỉ có thể xem tôi thực hiện vì không thể làm bài này mà không có domain đã đăng ký.

Để bắt đầu, hãy đảm bảo bạn đã đăng nhập bằng **IAM admin user** của tài khoản quản lý (management account) trong AWS Organization và đã chọn vùng **Northern Virginia (us-east-1)**.

### Tạo hạ tầng cần thiết bằng CloudFormation

Chúng ta cần tạo một số hạ tầng để thực hiện bài demo. Đính kèm bài học là **link one-click deployment**. Hãy nhấp vào link đó ngay bây giờ. Bạn sẽ được đưa đến màn hình **Quick Create Stack**.

Mọi thứ đã được điền sẵn:

- **Stack name**: DNS-and-Failover-Demo

Chỉ cần cuộn xuống dưới, **tick vào ô Capabilities** (xác nhận CloudFormation có thể tạo IAM resources), sau đó nhấp **Create stack**.

Quá trình tạo stack sẽ mất vài phút. Hãy **tạm dừng video**, chờ stack chuyển sang trạng thái **CREATE_COMPLETE**, sau đó tiếp tục.

### Kiểm tra EC2 Instance và cấp phát Elastic IP

Khi stack đã **CREATE_COMPLETE**, nó đã tạo nhiều tài nguyên, quan trọng nhất là **public EC2 instance**.

1. Gõ **EC2** vào thanh tìm kiếm, nhấp chuột phải → **Open in new tab**.
2. Chọn **Instances (running)** → bạn sẽ thấy instance tên **a4l-web**.
3. Chọn instance này. Ở mục **Public IPv4 address**, nhấp biểu tượng copy (không nhấp “Open address” vì nó dùng HTTPS).
4. Mở tab mới và dán IP → bạn sẽ thấy trang chủ **Animals for Life siêu tối giản**. Nếu thấy trang này → mọi thứ hoạt động đúng.

Tiếp theo, chúng ta cần cấp **Elastic IP** (địa chỉ IPv4 tĩnh) cho instance này:

1. Trong menu bên trái EC2, cuộn xuống **Network & Security** → chọn **Elastic IPs**.
2. Nhấp **Allocate Elastic IP address** → đảm bảo vùng **us-east-1** → nhấp **Allocate**.
3. Chọn Elastic IP vừa tạo → **Actions** → **Associate Elastic IP address**.
4. Chọn **Instance** → tìm và chọn **a4l-web**.
5. Chọn Private IP address của instance.
6. **Tick** ô “Allow this Elastic IP address to be reassociated”.
7. Nhấp **Associate**.

Bây giờ EC2 instance đã có địa chỉ IPv4 tĩnh.

### Tạo và cấu hình S3 Bucket (làm website dự phòng)

EC2 instance sẽ là **primary record**. Chúng ta sẽ tạo **S3 bucket** làm website dự phòng khi EC2 gặp sự cố.

1. Gõ **S3** vào thanh tìm kiếm → mở tab mới.
2. Nhấp **Create bucket**.
3. **Bucket name**: www.animalsforlife1337.org (thay bằng www. + domain bạn đã đăng ký).
4. Region: **US East (N. Virginia) – us-east-1**.
5. **Bỏ tick** “Block all public access” → tick xác nhận bạn hiểu rủi ro.
6. Cuộn xuống cuối → nhấp **Create bucket**.

Upload file website dự phòng:

- Tải file assets đính kèm bài học → giải nén → vào thư mục R53_Zones_and_Failover/02_a4l-failover.
- Chọn cả hai file: **index.html** và **minimal.jpeg** → Upload → Close.

Bật Static website hosting:

1. Vào bucket vừa tạo → tab **Properties**.
2. Cuộn xuống **Static website hosting** → nhấp **Edit**.
3. Chọn **Host a static website**.
4. Index document: index.html
5. Error document: index.html
6. Save changes.

Thêm Bucket Policy để bucket public:

1. Tab **Permissions** → **Bucket policy** → **Edit**.
2. Mở file bucket_policy.json trong thư mục assets → copy nội dung.
3. Dán vào ô Bucket policy.
4. Copy **Bucket ARN** (nhấp biểu tượng bên cạnh).
5. Thay thế đoạn arn:aws:s3:::example-bucket bằng ARN thật của bạn.
6. Nhấp **Save changes**.

Bây giờ website dự phòng trên S3 đã sẵn sàng.

### Thiết lập Route 53 Health Check

1. Gõ **Route 53** → mở tab mới → chọn **Health checks**.
2. Nhấp **Create health check**.
3. Tên: a4l-health
4. Loại: **Endpoint health check**.
5. Protocol: **HTTP**
6. Endpoint: dán **Elastic IP** của EC2 instance.
7. Path: /index.html
8. Mở **Advanced configuration** → chọn **Fast** (thay vì 30 giây mặc định).
9. Nhấp **Next** → chọn **No** (không tạo alarm) → **Create health check**.

Health check ban đầu sẽ ở trạng thái **Unknown**. Sau vài phút nó sẽ chuyển sang **Healthy** (bạn có thể xem tab Health checkers để theo dõi).

### Tạo Failover Record

1. Trong Route 53 → **Hosted zones** → chọn hosted zone của domain bạn đã đăng ký.
2. Nhấp **Create record** → chuyển sang **Wizard mode** → chọn **Failover** → **Next**.
3. Record name: www
4. TTL: chọn **1m** (60 giây).
5. Nhấp **Define failover record**.

**Tạo Primary record (EC2):**

- Chọn **IP address or another value**
- Dán **Elastic IP** của EC2
- Failover record type: **Primary**
- Chọn health check: a4l-health
- Record ID: EC2
- Nhấp **Define failover record**

**Tạo Secondary record (S3):**

- Nhấp lại **Define failover record**
- Chọn **Alias to an S3 website endpoint**
- Region: **us-east-1**
- Chọn bucket S3 bạn vừa tạo (www.yourdomain.org)
- Failover record type: **Secondary**
- **Không** gán health check
- Record ID: S3
- Nhấp **Define failover record**

Cuối cùng nhấp **Create records**.

### Mô phỏng sự kiện Failover

1. Copy DNS name www.yourdomain.org → mở tab mới → bạn sẽ thấy website trên **EC2**.
2. Quay lại EC2 console → chọn instance **a4l-web** → **Instance state** → **Stop instance** → xác nhận.
3. Quay lại Route 53 → Health checks → chọn a4l-health → tab **Health checkers** → Refresh liên tục.
4. Bạn sẽ thấy các lỗi **Connection timed out** → sau khoảng 1 phút health check chuyển sang **Unhealthy**.

Vì TTL = 60 giây, sau khi cache DNS hết hạn, refresh lại tab website → bạn sẽ thấy **website dự phòng trên S3** (trang Animals for Life failover).

### Khôi phục Primary Instance (Failback)

1. EC2 console → chọn instance → **Start instance**.
2. Chờ instance chuyển sang **Running** (mất vài phút).
3. Quay lại Health check → Refresh liên tục ở tab Health checkers.
4. Khi thấy nhiều dòng **HTTP status code 200 (OK)** → health check sẽ chuyển lại **Healthy**.
5. Refresh lại tab website → trang sẽ quay về **website trên EC2**.

Như vậy failover record đã hoạt động **hai chiều**: chuyển sang S3 khi EC2 down và chuyển ngược lại khi EC2 phục hồi.

Đây là kết thúc **Phần 1** của bài học. Phần 2 sẽ tiếp tục ngay sau. Hãy nghỉ ngơi một chút, uống cà phê, sau đó quay lại với phần tiếp theo!

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Yêu cầu: Domain đã đăng ký Route 53 + IAM Admin + region **us-east-1**
- One-click CloudFormation stack: **DNS-and-Failover-Demo**
- Cấp **Elastic IP** tĩnh cho EC2 instance **a4l-web**
- Tạo S3 bucket **[www.yourdomain.org](http://www.yourdomain.org)** làm static website + bucket policy public
- Tạo **Health Check** a4l-health (Fast, HTTP /index.html)
- Tạo **Failover record** www: Primary (EC2 + health check) + Secondary (S3 Alias)
- Mô phỏng failover: **Stop EC2** → health check Unhealthy → DNS chuyển sang S3 sau TTL 60s
- Khôi phục: **Start EC2** → health check Healthy → failback về EC2

**Notes (Chi tiết):**

- **CloudFormation stack** tự động tạo EC2 instance public (a4l-web) và các tài nguyên cần thiết. Sau khi stack **CREATE_COMPLETE**, phải tự cấp **Elastic IP** để có địa chỉ tĩnh.
- S3 bucket phải đặt tên đúng chuẩn www.yourdomain.org, **bỏ Block Public Access**, bật **Static website hosting** với index.html, và thêm **Bucket Policy** (thay placeholder ARN bằng ARN thật của bucket).
- **Health Check** dùng chế độ **Fast** để phát hiện lỗi nhanh nhất. Ban đầu ở trạng thái Unknown → sau vài phút chuyển Healthy khi EC2 hoạt động bình thường.
- **Failover record** gồm 2 record cùng tên www:
    - **Primary**: trỏ đến Elastic IP của EC2 + gắn health check a4l-health. Chỉ được dùng khi health check Healthy.
    - **Secondary**: Alias trỏ đến S3 website endpoint (us-east-1). Chỉ được dùng khi Primary fail.
- TTL = **60 giây** nên sau khi health check chuyển Unhealthy, phải chờ tối đa 1 phút DNS cache hết hạn thì mới thấy chuyển sang S3.
- Khi Start lại EC2, health check cần vài phút để thu thập đủ dữ liệu “OK” từ các health checker toàn cầu trước khi chuyển lại Healthy và failback.

**Summary (Tóm tắt ngắn):** Bài học hướng dẫn cách thiết lập **failover DNS tự động** giữa EC2 instance (primary) và S3 static website (secondary) bằng Route 53 Health Check. Quy trình bao gồm: tạo hạ tầng qua CloudFormation, cấp Elastic IP, cấu hình S3 website dự phòng, tạo health check nhanh, định nghĩa failover record Primary/Secondary, sau đó mô phỏng cả hai chiều chuyển mạch (failover khi EC2 dừng và failback khi EC2 phục hồi). TTL 60 giây đảm bảo chuyển đổi nhanh chóng khi có sự cố.