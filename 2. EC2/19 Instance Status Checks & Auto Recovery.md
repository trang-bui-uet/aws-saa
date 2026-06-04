**Chào mừng bạn quay lại.**

Bài học này sẽ **rất ngắn gọn**. Chúng ta sẽ tìm hiểu về **Instance Status Checks** và **EC2 Auto Recovery**. Cả hai đều là những tính năng khá đơn giản, nhưng hãy cùng xem nhanh chính xác chúng làm gì và những khả năng chúng mang lại, vì đây là những kiến thức rất đáng nắm vững.

Mọi **instance** trong EC2 đều có **hai status check cấp cao** theo từng instance. Khi bạn mới launch instance, bạn có thể thấy chúng hiển thị trạng thái **initializing**, sau đó chỉ một trong hai check pass. Nhưng cuối cùng, tất cả các instance đều phải đạt **two-out-of-two passed checks** – điều này cho biết mọi thứ đang hoạt động bình thường. Nếu không đạt, bạn đang gặp vấn đề.

Mỗi status check đại diện cho một tập hợp kiểm tra riêng biệt. Việc **fail** bất kỳ check nào đều chỉ ra một nhóm vấn đề cơ bản khác nhau:

- **System status check** (kiểm tra hệ thống): Failure có thể do mất nguồn điện hệ thống, mất kết nối mạng, hoặc lỗi phần mềm/phần cứng trên **EC2 host**. → Check này tập trung vào vấn đề ảnh hưởng đến **dịch vụ EC2** hoặc **EC2 host**.
- **Instance status check** (kiểm tra instance): Failure có thể do file system bị hỏng, cấu hình networking sai trên instance (ví dụ: đặt tĩnh public IPv4 trên interface nội bộ – điều mà bạn đã biết là **không bao giờ hoạt động**), hoặc kernel của hệ điều hành gặp vấn đề khiến instance không khởi động được. → Check này tập trung vào vấn đề **bên trong instance**.

Giả sử bạn không vừa launch instance, bất kỳ trường hợp nào **không đạt two-out-of-two checks** đều là vấn đề cần khắc phục.

### Cách xử lý khi status check fail

Bạn có thể xử lý **thủ công**:

- Stop và Start instance
- Restart instance
- Terminate rồi tạo instance mới

Tuy nhiên, EC2 cung cấp tính năng **tự động** để xử lý các vấn đề liên quan đến **system check**:

- Stop instance
- Reboot instance
- Terminate instance
- **Auto-recovery** (khôi phục tự động) ← Đây là tính năng quan trọng nhất

**EC2 Auto Recovery** sẽ:

- Di chuyển instance sang **host mới**
- Khởi động lại với **cấu hình hoàn toàn giống hệt** (giữ nguyên IP, EBS volumes…)
- Nếu phần mềm bên trong instance được thiết lập auto-start, instance có thể **tự động khôi phục hoàn toàn**

> **Lưu ý quan trọng**: Auto-recovery **chỉ hoạt động trong cùng Availability Zone (AZ)**. Nó không bảo vệ bạn khỏi sự cố toàn bộ AZ.

### Demo thực tế trên Console AWS

Tôi đang đăng nhập vào **AWS Management Account** với IAM Admin user, region **Northern Virginia (us-east-1)**.

(Phần demo rất ngắn, nên bạn **không nhất thiết phải theo dõi** theo. Nếu muốn thực hành, có **one-click CloudFormation stack** đính kèm bài học.)

Sau khi stack tạo xong (Create complete):

1. Vào **EC2 Console** → Chọn instance được tạo bởi stack.
2. Chuyển sang tab **Status Checks**.
    - Bạn sẽ thấy:
        - **System status check**: Passed
        - **Instance status check**: Passed

Để thiết lập **auto-recovery**:

1. Click **Actions** → **Create status check alarm**.
2. Alarm này là **CloudWatch Alarm**:
    - Kích hoạt khi **bất kỳ status check nào fail** trong **1 khoảng thời gian liên tiếp 5 phút**.
    - Có thể gửi thông báo qua **SNS topic**.
3. Tick vào ô **Perform actions** và chọn:
    - **Reboot this instance**
    - **Stop this instance**
    - **Terminate this instance**
    - **Recover this instance** ← **Khuyến nghị**

**Recover this instance** sử dụng chính **EC2 Auto Recovery**:

- Có thể chỉ restart hoặc di chuyển sang host mới.
- Giữ nguyên **cùng AZ**, cùng IP, cùng cấu hình.
- **Yêu cầu**:
    - Instance phải dùng loại **supported** (A1, C4, C5, M4, M5, R3, R4, R5…).
    - Chỉ dùng **EBS volumes** (không hỗ trợ instance store volumes).
    - Phải có spare capacity trên host.

### Tóm tắt quan trọng

- **Auto-recovery** chỉ giải quyết các sự cố **cô lập** (host/instance), **không phải** sự cố lớn hoặc phức tạp.
- Đây là tính năng đơn giản giúp **tránh phải gọi sysadmin** trong một số trường hợp cụ thể.
- Các giải pháp HA phức tạp hơn sẽ được học ở những bài sau.

**Kết thúc bài học** Chúng ta sẽ **không xóa** stack này vì bài học tiếp theo sẽ dùng lại để demo **EC2 Termination Protection**.

Hoàn thành video và hẹn gặp bạn ở bài học tiếp theo!