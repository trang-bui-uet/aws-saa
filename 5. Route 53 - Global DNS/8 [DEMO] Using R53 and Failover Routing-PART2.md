Chào mừng trở lại! Đây là **phần hai** của bài học demo này. Chúng ta sẽ tiếp tục ngay từ phần kết thúc của phần một, vì vậy hãy bắt đầu nào.

Bây giờ, một điều cuối cùng trước khi kết thúc bài học demo này: Tôi muốn nói về **Private Hosted Zone** (Vùng lưu trữ DNS riêng tư).

**Tạo Private Hosted Zone**

1. Quay lại bảng điều khiển **Route 53** và đi đến **Hosted Zones**.
2. Nhấp vào **Create Hosted Zone**. Vì đây là private hosted zone, nó thậm chí không cần phải là tên miền mà bạn thực sự sở hữu.
3. Đặt tên cho hosted zone là **ilikedogsreally.com** và đặt loại là **Private hosted zone**.
4. Hiện tại, liên kết nó với **default VPC** trong khu vực **us-east-1**. Chọn vùng **US East (N. Virginia)**.
5. Nhấp vào ô **VPC ID**. Bạn sẽ thấy hai VPC được liệt kê. Một là **Animals for Life VPC** (được gắn tag **A4L-VPC1**), nhưng **đừng chọn** cái đó. Hãy chọn **default VPC** (cái không có văn bản nào sau nó).
6. Sau khi thiết lập xong, nhấp vào **Create hosted zone**.

**Xác định bản ghi đơn giản (Defining a Simple Record)**

1. Bên trong hosted zone mới, nhấp vào **Create record**.
2. Chọn chính sách định tuyến **Simple routing** và nhấp **Next**.
3. Nhấp vào **Define simple record**.
4. Đặt subdomain là **www**.
5. Đối với loại bản ghi, chọn **A – Routes traffic to an IPv4 address and some resources**.
6. Nhấp vào ô endpoint, chọn **IP address or another value depending on record type**, và nhập địa chỉ IP thử nghiệm: **1.1.1.1**.
7. Ở dưới cùng, thay đổi cài đặt **TTL** thành **1m** (60 giây).
8. Nhấp **Define simple record**, và cuối cùng nhấp **Create records**. Bây giờ chúng ta có một bản ghi tên là **[www.ilikedogsreally.com](https://www.ilikedogsreally.com)**. Sao chép nó vào clipboard.

**Kiểm tra kết nối (Testing the Connection)**

1. Quay lại bảng điều khiển **EC2**.
2. Nhấp **Dashboard**, sau đó chọn **Instances (running)**.
3. Nhấp chuột phải vào instance của bạn → **Connect** → chọn **EC2 Instance Connect** → nhấp **Connect**.
4. Sau khi kết nối thành công, hãy thử ping bản ghi bạn vừa tạo:

Bash

```
ping www.ilikedogsreally.com
```

5. Nhấn Enter. Bạn sẽ nhận được lỗi **Name or service not known**.

**Lý do**: Private hosted zone chúng ta vừa tạo hiện đang được liên kết với **default VPC**, nhưng EC2 instance này **không chạy** trong default VPC.

**Liên kết VPC (Associating the VPC)**

Để cho phép instance này phân giải được các bản ghi bên trong private hosted zone, chúng ta cần liên kết thêm **Animals for Life VPC** vào hosted zone.

1. Quay lại bảng điều khiển **Route 53**.
2. Mở rộng **Hosted zone details** và nhấp **Edit hosted zone**.
3. Cuộn xuống phần **Add another VPC**.
4. Trong dropdown **Region**, chọn **us-east-1**.
5. Trong ô **VPC ID**, chọn **A4L-VPC1**.
6. Cuộn xuống và nhấp **Save changes**.

**Lưu ý quan trọng**: Việc này có thể mất vài giây đến vài phút để có hiệu lực. Nếu bạn quay lại EC2 instance và chạy lệnh ping ngay lập tức, bạn vẫn có thể nhận lỗi **Name or service not known**. Hãy tạm dừng video, chờ khoảng **4–5 phút**, sau đó thử lại lệnh.

Sau khi liên kết cập nhật thành công, lệnh ping sẽ **phân giải được** [www.ilikedogsreally.com](http://www.ilikedogsreally.com) vì private hosted zone đã được liên kết với VPC mà instance đang chạy.

**Dọn dẹp hạ tầng (Cleaning Up Infrastructure)**

Chúng ta đã hoàn thành nội dung demo. Bây giờ hãy dọn dẹp toàn bộ hạ tầng đã tạo để tránh phát sinh chi phí không cần thiết.

**1. Xóa Health Check**

- Quay lại **Route 53** console → chọn **Health checks**.
- Chọn **A4L-Health** → **Delete health check** → xác nhận xóa.

**2. Xóa các bản ghi Route 53 và Private Hosted Zone**

- Vào **Hosted zones** → mở private hosted zone **ilikedogsreally.com** bạn vừa tạo.
- Chọn bản ghi **[www.ilikedogsreally.com](https://www.ilikedogsreally.com)** → **Delete record** → xác nhận.
- Quay lại **Hosted zones**, chọn toàn bộ private hosted zone → **Delete** → gõ delete để xác nhận.
- Vào public hosted zone của bạn, chọn **hai bản ghi www** đã tạo ở phần trước → **Delete records** → xác nhận.

**3. Làm rỗng và xóa S3 Bucket**

- Vào **S3** console → nhấp vào bucket đã tạo.
- Nhấp **Empty** → gõ permanently delete → nhấp **Empty**.
- Sau khi làm rỗng xong, nhấp **Exit**. Vẫn giữ bucket được chọn → **Delete** → gõ tên đầy đủ bucket → **Delete bucket**.

**4. Giải phóng Elastic IP**

- Vào **EC2** console → mở menu hamburger → cuộn xuống **Elastic IPs**.
- Chọn Elastic IP đang liên kết với instance.
- Nhấp **Actions** → **Disassociate Elastic IP address** → xác nhận.
- Vẫn giữ mục được chọn → **Actions** → **Release Elastic IP addresses** → **Release**.

**5. Xóa CloudFormation Stack**

- Vào **CloudFormation** console → **Stacks**.
- Chọn stack đã tạo ở đầu bài học bằng one-click deployment (tên **DNS-and-Failover-Demo**).
- Nhấp **Delete** → **Delete stack** để xác nhận.

**Kết luận**

Sau khi stack bị xóa hoàn toàn, tài khoản AWS của bạn sẽ trở về **đúng trạng thái ban đầu** như lúc bắt đầu bài học.

Tôi hy vọng đây là một trải nghiệm thực hành thú vị và bổ ích khi học cách sử dụng **failover routing** và **private hosted zones**! Kiến thức này rất hữu ích cho cả kỳ thi chứng chỉ sắp tới lẫn kiến trúc thực tế. Hẹn gặp lại trong bài học tiếp theo!

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Private Hosted Zone trong Route 53
- Tạo hosted zone riêng tư ilikedogsreally.com + liên kết default VPC
- Tạo bản ghi A đơn giản www.ilikedogsreally.com trỏ IP 1.1.1.1 (TTL 1 phút)
- Ping từ EC2 thất bại do instance không nằm trong VPC được liên kết
- Thêm association **A4L-VPC1** vào private hosted zone + chờ 4–5 phút
- Quy trình dọn dẹp toàn diện: Health Check, Records, Hosted Zone, S3 Bucket, Elastic IP, CloudFormation stack
- Trả tài khoản AWS về trạng thái ban đầu

**Notes (Chi tiết):**

- **Private Hosted Zone** cho phép phân giải DNS nội bộ trong VPC mà không cần sở hữu tên miền thật.
- Bản ghi **Simple routing** với TTL ngắn (60 giây) rất tiện để test nhanh sự thay đổi.
- EC2 instance **phải nằm trong VPC** được liên kết với private hosted zone thì mới resolve được tên miền riêng tư.
- Sau khi thêm VPC association, cần chờ **4–5 phút** để DNS propagation có hiệu lực.
- **Cleanup toàn diện** gồm: xóa Health Check **A4L-Health**, xóa record trong private + public hosted zone, empty + delete S3 bucket, disassociate + release Elastic IP, xóa stack **DNS-and-Failover-Demo**.

**Summary (Tóm tắt ngắn):** Bài học hướng dẫn cách **tạo Private Hosted Zone** trong Route 53, định nghĩa bản ghi DNS đơn giản, kiểm tra phân giải từ EC2 instance (và cách khắc phục bằng cách liên kết thêm VPC), đồng thời thực hiện quy trình dọn dẹp hoàn chỉnh tất cả tài nguyên từ toàn bộ bài học (Health Check, Hosted Zones, S3, Elastic IP, CloudFormation stack) để tránh phát sinh chi phí và trả tài khoản về trạng thái ban đầu.