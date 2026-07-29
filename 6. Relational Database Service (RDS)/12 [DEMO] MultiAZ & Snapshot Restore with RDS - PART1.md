**Chào mừng trở lại**, và trong bài demo lesson này, chúng ta sẽ tiếp tục triển khai kiến trúc này. Trong bài demo trước, bạn đã **migrate database** từ MariaDB tự quản lý trên EC2 sang **RDS**. Trong bài này, bạn sẽ được trải nghiệm làm việc với chế độ **Multi-AZ (Availability Zone)** của RDS, cũng như tạo **snapshot**, khôi phục snapshot và thử nghiệm **failover** của RDS.

### Chuẩn bị hạ tầng

Để hoàn thành bài demo này, bạn cần một số hạ tầng sẵn có. Hãy chuyển sang **AWS Console**. Bạn cần đăng nhập vào tài khoản AWS general (tức là management account của organization) và luôn chọn region **US East (N. Virginia)**.

Attached với bài học này có một **one-click deployment link**. Hãy mở link đó. Nó sẽ đưa bạn đến trang **Create stack** với mọi thứ đã được điền sẵn.

- **Stack name**: RDS-MultiAZ-Stack
- Tất cả parameters giữ giá trị mặc định
- **MultiAZ** hiện đang để **false** → giữ nguyên
- Tick vào ô **capabilities** ở dưới cùng
- Click **Create stack**

Hạ tầng này sẽ mất khoảng **15 phút** để hoàn tất. Bạn cần đợi stack chuyển sang trạng thái **CREATE_COMPLETE** trước khi tiếp tục. Hãy tạm dừng video và resume khi CloudFormation đã ở trạng thái **CREATE_COMPLETE**.

### Cài đặt WordPress và thêm bài post thử nghiệm

Khi stack đã **CREATE_COMPLETE**, chúng ta cần hoàn tất cài đặt WordPress và thêm bài blog post thử nghiệm vì sẽ dùng chúng xuyên suốt bài demo.

Đây là thao tác bạn đã làm nhiều lần nên chúng ta sẽ làm nhanh:

1. Vào **Services** → **EC2** → **Running Instances**
2. Sao chép **Public IPv4 address** của instance **A4L-WordPress**
3. Mở địa chỉ đó trong tab mới

Cài đặt WordPress với thông tin sau:

- **Site Title**: The Best Cats
- **Username**: admin
- **Password**: mật khẩu mạnh AnimalsForLife
- **Email**: [test@test.com](mailto:test@test.com)

Click **Install WordPress**, sau đó đăng nhập bằng admin + mật khẩu.

Sau khi đăng nhập:

- Vào **Posts** → xóa bài “Hello world!” (chuyển vào Trash)
- Tạo bài mới với tiêu đề: **“The best cats ever”**
- Click dấu **+** → chọn **Gallery**
- Tải file ảnh đính kèm bài học → giải nén (có 4 ảnh)
- Upload 4 ảnh đó vào bài
- Click **Publish** (hai lần)

**Lưu ý quan trọng**: Ảnh được lưu trên **file system cục bộ** của instance, còn metadata của bài post được lưu trong database trên **RDS**.

### Tạo Snapshot

Hãy tưởng tượng bài blog này là một **ứng dụng production** (ví dụ hệ thống quản lý nội dung). Chúng ta sẽ xem mọi thao tác trong bài demo dưới góc nhìn production.

Quay lại **AWS Console** → **Services** → **RDS** → **Databases**.

1. Chọn database (được tạo bởi one-click deployment)
2. Chọn **Actions** → **Take snapshot**

**Snapshot** là bản sao **point-in-time** của database. Lần snapshot đầu tiên là **full snapshot** (copy toàn bộ dữ liệu), nên tốn dung lượng bằng với dữ liệu hiện có của RDS.

Đặt tên snapshot theo định dạng: a4l-wordpress-with-cat-post-mysql- + phiên bản MySQL (không dấu chấm, không khoảng trắng).

_(Kiểm tra tên chính xác trong phần mô tả bài học vì phiên bản MySQL có thể khác tùy thời điểm bạn làm bài.)_

Click **Take snapshot**. Thời gian hoàn thành phụ thuộc vào lượng dữ liệu, tốc độ AWS và đây có phải snapshot đầu tiên hay không. Snapshot đầu luôn lâu nhất; các snapshot sau chỉ copy phần dữ liệu thay đổi.

Hãy tạm dừng video và đợi đến khi snapshot chuyển sang trạng thái **Available**.

Khi hoàn tất, snapshot này chứa chứa bản sao database MySQL chứa phiên bản cụ thể, chứa WordPress database cùng bài post về mèo. Bạn có thể tạo thêm snapshot khác (sẽ nhanh hơn nhiều) nhưng bài demo không thực hiện bước đó.

**Lưu ý production**: Snapshot thủ công **tồn tại độc lập** với vòng đời của RDS instance. Bạn phải tự xóa chúng (hoặc dùng script) nếu muốn dọn dẹp. Chúng không được RDS quản lý tự động → quan trọng cho **Disaster Recovery** và quản lý chi phí.

### Bật chế độ Multi-AZ

Hiện tại database đang chạy **single instance**, không chịu được sự cố của một Availability Zone.

Để bật Multi-AZ:

1. Chọn database → **Modify**
2. Trong phần **Availability & durability**, đổi từ “Do not create a standby instance” thành **“Create a standby instance”**
3. Scroll xuống → chọn **Continue**
4. Chọn **Apply immediately** (vì đây là demo)
5. Click **Modify DB instance**

**Lưu ý**: Multi-AZ **không nằm trong Free Tier**, sẽ phát sinh chi phí nhỏ.

**Multi-AZ** tạo một **standby replica** đồng bộ ở Availability Zone khác (trong cùng DB subnet group). **Lợi ích chính**:

- Dự phòng dữ liệu
- Các thao tác ảnh hưởng I/O (như backup) chạy trên standby → không ảnh hưởng primary
- Bảo vệ khỏi sự cố Availability Zone: nếu primary fail, CNAME endpoint sẽ chuyển sang standby → giảm thiểu gián đoạn

Quá trình phía sau: AWS chụp snapshot primary → restore sang standby → thiết lập synchronous replication. Hãy tạm dừng và đợi status chuyển từ **Modifying** sang **Available** (thường mất khoảng 10 phút).

### Mô phỏng Failover

Khi status đã **Available**, chúng ta có thể mô phỏng sự cố:

1. Chọn database → **Actions** → **Reboot**
2. Tick **Reboot with failover?**
3. Click **Confirm**

Quay lại trang WordPress và reload bài post. Bạn sẽ thấy trang **không load ngay lập tức**. Thời gian failover điển hình là **60–120 giây**. Đây là điểm quan trọng cần nhớ khi triển khai RDS cho hệ thống critical – failover **không tức thì**.

Sau khoảng thời gian đó, trang sẽ load lại vì CNAME đã chuyển sang standby (giờ trở thành primary mới).

---

**Đây là hết Part 1** của bài học. Phần này hơi dài nên mình tách ra để bạn nghỉ ngơi hoặc pha cà phê. Part 2 sẽ tiếp tục ngay từ điểm kết thúc của Part 1. Hãy hoàn thành video này và khi sẵn sàng thì gặp lại ở Part 2.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Deploy stack RDS-MultiAZ-Stack (MultiAZ = false)
- Cài WordPress + thêm bài post “The best cats ever”
- Tạo manual snapshot
- Bật Multi-AZ (Create standby instance)
- Reboot with failover
- Thời gian failover 60–120 giây

**Notes (Chi tiết):**

- Stack CloudFormation mất khoảng **15 phút** để CREATE_COMPLETE.
- Snapshot đầu tiên là **full copy**; các snapshot sau chỉ chứa phần thay đổi.
- Snapshot thủ công **không bị xóa tự động** khi xóa RDS instance → cần quản lý thủ công.
- Multi-AZ tạo **standby replica đồng bộ** ở AZ khác → tăng độ sẵn sàng và giảm ảnh hưởng I/O.
- Khi bật Multi-AZ, chọn **Apply immediately** cho demo (production nên cân nhắc maintenance window).
- **Reboot with failover** dùng để mô phỏng sự cố AZ; CNAME endpoint sẽ chuyển sang standby.
- Ứng dụng sẽ bị gián đoạn ngắn (thường 1–2 phút) trong quá trình failover.

**Summary (Tóm tắt ngắn):** Bài demo hướng dẫn triển khai RDS Multi-AZ, tạo manual snapshot và mô phỏng failover. Sau khi deploy stack, cài WordPress + thêm dữ liệu thử nghiệm, tạo snapshot chứa bài post, bật Multi-AZ và thực hiện reboot with failover để quan sát thời gian chuyển đổi (60–120 giây).