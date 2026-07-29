**Chào mừng trở lại.** Đây là **Part 2** của bài học. Chúng ta sẽ tiếp tục ngay từ điểm kết thúc của Part 1.

### Mô phỏng Data Corruption

Bước tiếp theo là trình diễn cách **khôi phục RDS** khi xảy ra data corruption. Chúng ta sẽ mô phỏng bằng cách quay lại WordPress blog và làm hỏng một phần dữ liệu:

- Đổi tiêu đề bài post từ **“The best cats ever”** thành **“Not the best cats ever”** (rõ ràng là sai).
- Click **Update** để lưu thay đổi.

Đây chính là giả lập tình huống dữ liệu bị corrupt trong ứng dụng production.

### Restore từ Manual Snapshot

Giả sử chúng ta cần khôi phục database từ snapshot trước đó. (Tạm thời bỏ qua tính năng automatic backup của RDS, chỉ dùng **manual snapshot**.)

1. Quay lại **RDS Console** → **Snapshots**
2. Chọn snapshot đã tạo từ đầu bài demo (chứa bài post đúng)
3. Chọn **Actions** → **Restore snapshot**

**Điểm quan trọng cần nhớ** (xuất hiện rất nhiều trong exam và thực tế): Khi restore snapshot trên **normal RDS**, hệ thống sẽ **tạo một database instance hoàn toàn mới**, **không restore đè lên instance cũ**.

Cấu hình restore như sau:

- **DB instance identifier**: a4l-wordpress-restore
- **Deployment option**: chọn **Single DB instance** (demo nên không cần Multi-AZ)
- **Instance type**: Burstable classes → chọn t2.micro hoặc t3.micro
- **Storage**: giữ mặc định
- **VPC**: chọn **A4L-VPC1**
- **Subnet group**: dùng subnet group được tạo bởi one-click deployment
- **Public access**: **No**
- **VPC security group**: chọn **RDS-MultiAZ-Snap-RDS-SecurityGroup** (không chọn security group của EC2), sau đó xóa default
- Các phần Authentication & Encryption giữ mặc định
- Click **Restore DB instance**

Hãy tạm dừng video và đợi instance mới chuyển từ trạng thái **Creating** sang **Available** (thường mất khoảng 10 phút).

### Điểm mấu chốt quan trọng nhất

Khi restore hoàn tất, bạn sẽ có instance mới tên a4l-wordpress-restore.

So sánh endpoint DNS:

- Instance gốc có một chuỗi ngẫu nhiên + region + .rds.amazonaws.com
- Instance restore có endpoint hoàn toàn khác (bắt đầu bằng a4l-wordpress-restore...)

→ **Restore trên normal RDS luôn tạo instance mới + endpoint DNS mới**. → Ứng dụng **phải được cập nhật** để trỏ sang endpoint mới thì mới dùng được dữ liệu đã khôi phục.

### Cập nhật cấu hình WordPress

1. Mở tab EC2 → chuột phải vào instance **A4L-WordPress** → **Connect** → chọn **EC2 Instance Connect** (username: ec2-user)
2. Chạy lần lượt các lệnh:

Bash

```
cd /var/www/html
ls -la
sudo nano wp-config.php
```

3. Tìm dòng DB_HOST
4. Xóa toàn bộ giá trị cũ (chỉ để lại hai dấu nháy đơn '')
5. Quay lại RDS Console → copy **Endpoint** của instance a4l-wordpress-restore
6. Dán vào giữa hai dấu nháy → lưu file (Ctrl+O → Enter → Ctrl+X)

Quay lại WordPress và refresh trang → bạn sẽ thấy tiêu đề đúng trở lại: **“The best cats ever”**.

### Tóm tắt kiến thức quan trọng

- Với **normal RDS** (MySQL, PostgreSQL, Oracle, SQL Server…): restore snapshot = tạo **instance mới** + **endpoint mới** → bắt buộc phải cập nhật cấu hình ứng dụng.
- **Không thể restore in-place**.
- Aurora có cơ chế khác (sẽ học sau).

### Dọn dẹp hạ tầng

1. Vào **Databases** → chọn a4l-wordpress-restore
    - **Actions** → **Delete**
    - **Không** tạo final snapshot
    - **Không** retain automated backups
    - Gõ delete me để xác nhận → **Delete**
2. Giữ lại **manual snapshot** gốc (có bài post mèo) vì sẽ dùng ở phần sau của khóa học.
3. Đợi instance restore biến mất khỏi danh sách.
4. Quay lại **CloudFormation** → chọn stack RDS-MultiAZ-Snap → **Delete** → xác nhận.

Sau khi xóa xong, tài khoản sẽ trở về trạng thái ban đầu, **ngoại trừ** manual snapshot vẫn còn.

---

Bài demo này giúp bạn có trải nghiệm thực tế về các tính năng **resilience** và **recovery** của normal RDS. Mặc dù phải chờ khá nhiều, nhưng đây là kiến thức quan trọng cho cả kỳ thi lẫn công việc thực tế.

Khi sẵn sàng, hãy chuyển sang bài học tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Mô phỏng data corruption
- Restore snapshot → tạo instance mới
- Endpoint DNS thay đổi
- Cập nhật wp-config.php (DB_HOST)
- Không thể restore in-place trên normal RDS
- Cleanup: xóa instance restore + xóa CloudFormation stack

**Notes (Chi tiết):**

- Thay đổi tiêu đề bài post thành “Not the best cats ever” để giả lập corruption.
- Restore snapshot luôn tạo **database instance mới** với **endpoint DNS hoàn toàn khác**.
- Phải sửa file /var/www/html/wp-config.php → thay DB_HOST bằng endpoint của instance restore.
- Sau khi cập nhật, WordPress sẽ hiển thị lại dữ liệu đúng.
- Khi xóa instance restore: không tạo final snapshot, không retain automated backups, gõ delete me.
- Giữ lại manual snapshot gốc để dùng ở các bài sau.
- Cuối cùng xóa CloudFormation stack RDS-MultiAZ-Snap.

**Summary (Tóm tắt ngắn):** Part 2 trình diễn quy trình khôi phục dữ liệu bị corruption bằng manual snapshot. Restore trên normal RDS luôn tạo instance mới kèm endpoint mới, buộc phải cập nhật cấu hình ứng dụng (wp-config.php). Kết thúc bằng quy trình dọn dẹp: xóa instance restore và CloudFormation stack, chỉ giữ lại manual snapshot.