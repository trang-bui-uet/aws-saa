Chào mừng bạn quay lại. Trong bài demo này, bạn sẽ trải nghiệm nhanh cách session stickiness hoạt động với load balancer. Demo khá ngắn vì phần lớn hạ tầng đã được tự động hóa — trọng tâm chỉ là cấu hình session stickiness. Hãy bắt đầu bằng việc apply một CloudFormation template để tạo hạ tầng cần thiết.

## Chuẩn bị và tạo stack

1. Đăng nhập AWS account với user có quyền admin.
2. Chọn region Northern Virginia (us-east-1).
3. Dùng one-click link trong hướng dẫn demo để deploy hạ tầng.
4. Trên màn hình Create stack, kéo xuống cuối → tick ô Capabilities → bấm Create stack.
5. Stack có thể mất 5–10 phút để tạo xong.

## Kiến trúc được tạo

Template tạo:

- Một VPC
- 3 public subnets, mỗi subnet một AZ
- Một Auto Scaling group gắn launch template
- ASG tạo 6 EC2 instances (2 mỗi AZ)
- Một load balancer chạy từ các public subnet

## Ý tưởng demo

1. Kết nối load balancer khi session stickiness tắt → mỗi lần kết nối có thể tới bất kỳ trong 6 instance (~16.66% mỗi instance).
2. Bật session stickiness và quan sát thay đổi.
3. Lần đầu kết nối khi stickiness bật → cookie AWSALB được tạo và trả về browser.
4. Trong thời gian cookie còn hiệu lực, mọi kết nối bị khóa vào một EC2 cụ thể, cho đến khi:
    - cookie hết hạn, hoặc
    - instance đó fail health check → chuyển sang instance khác.

Đợi CloudFormation chuyển sang CREATE_COMPLETE rồi mới tiếp tục (có thể pause video khi còn CREATE_IN_PROGRESS).

## Kiểm tra 6 EC2 instances

1. Mở Services → tìm EC2 → mở tab mới.
2. Vào Instances (running).
3. Với từng instance: copy Public IPv4 DNS → mở tab mới.
4. Mỗi instance sẽ hiện: Instance ID, nền màu ngẫu nhiên, và GIF mèo animated.
5. Mở cả 6 instances — mỗi cái nền khác và GIF mèo khác.

## Thử load balancer khi chưa có stickiness

1. Vào Load Balancers.
2. Chọn LB tên bắt đầu bằng ALB-ALB….
3. Copy DNS name → mở tab mới.
4. Refresh nhiều lần → bạn sẽ thấy trang xoay vòng giữa các EC2 (có thể trùng đôi lần).
5. Lý do: chưa bật stickiness → chọn backend theo kiểu round-robin.

## Bật session stickiness trên Target Group

Giả sử ứng dụng không lưu state bên ngoài mà lưu trên chính EC2. Với ALB, stickiness cấu hình theo target group:

1. Vào Target Groups → chọn target group.
2. Tab Attributes → Edit.
3. Tick stickiness → chọn Load balancer generated cookie.
4. Đặt thời hạn cookie: để 1, đổi đơn vị từ days sang minutes.
5. Save changes.

Quay lại tab load balancer, tiếp tục refresh:

- Ban đầu có thể đổi instance một lần.
- Sau đó sẽ khóa vào một EC2 cố định — cùng Instance ID, cùng nền, cùng GIF mèo dù refresh nhiều lần.

## Xem cookie AWSALB (Firefox)

1. Tools → Browser Tools → Web Developer Tools.
2. Tab Storage.
3. Thấy cookie AWSALB — đây là cookie điều khiển stickiness.
4. Mỗi lần truy cập, browser gửi cookie này → ALB biết phải nối bạn tới backend nào.
5. Bạn ở lại instance đó đến khi cookie hết hạn hoặc instance fail health check.

## Thử khi instance fail

1. Ghi lại Instance ID đang kết nối (chú ý vài số cuối).
2. EC2 console → Instances (running) → tìm instance đó → Stop → confirm.
3. Đợi instance dừng → quay lại tab load balancer → refresh.
4. LB phát hiện instance không còn hợp lệ → chuyển sang EC2 mới; cookie được cập nhật để khóa vào instance mới.
5. User hầu như không “cảm nhận” backend fail (ngoài việc thấy Instance ID đổi vì demo cố tình highlight).

Nếu Start lại instance cũ:

- Bạn không quay về instance cũ vì đã bị khóa vào instance mới.
- Có thể trong lúc stopped, do dùng ELB health checks, ASG đã terminate và thay thế instance đó — nếu thấy terminated thì bình thường, hệ thống đang hoạt động đúng.

Sau khi cookie hết hạn, vẫn có khả năng bạn được chuyển sang EC2 khác khi refresh.

## Tắt stickiness (trả về cấu hình ban đầu)

1. Target Groups → mở target group → Attributes → Edit.
2. Bỏ tick Stickiness → Save.
3. Cookie không còn khóa kết nối; refresh nhiều lần sẽ lại xoay giữa các backend.

## Điểm cần nhớ cho kỳ thi

- Nếu ứng dụng không xử lý state bên ngoài EC2, cần ALB đảm bảo mỗi user luôn về cùng một instance → dùng session stickiness.
- Nhược điểm: LB phân tải kém linh hoạt hơn — user bị khóa một instance, dù tải không đều giữa các instance.
- Nên thiết kế ứng dụng xử lý session bên ngoài instance và không bật stickiness để có kiến trúc elastic hiệu quả.

## Cleanup

1. Vào CloudFormation → Stacks.
2. Chọn stack ALB → Delete → Delete stack.
3. Toàn bộ hạ tầng tạo lúc đầu sẽ bị xóa.

Chúc mừng! Bạn đã hoàn thành demo và trải nghiệm cách ALB xử lý session stickiness. Hãy hoàn thành video và sẵn sàng cho bài tiếp theo.

---

Tóm tắt theo Cornell Note

Cues (Từ khóa / Ý chính):

- Demo CloudFormation: VPC, 6 EC2, ALB
- Không stickiness = round-robin giữa 6 instance
- Bật stickiness trên Target Group (cookie AWSALB)
- Cookie 1 phút; khóa user vào 1 EC2
- Stop instance → failover sang backend mới
- Nhược điểm stickiness & khuyến nghị stateless
- Cleanup: Delete CloudFormation stack

Notes (Chi tiết):

- Deploy bằng one-click CloudFormation (Northern Virginia, admin) → CREATE_COMPLETE rồi mới làm tiếp.
- Kiến trúc: VPC + 3 public subnets/AZ + ASG 6 instances + ALB.
- Không stickiness: refresh LB DNS → đổi giữa các instance (nền/GIF khác nhau).
- Bật stickiness: Target Group → Attributes → Load balancer generated cookie, 1 minute → cookie AWSALB khóa session.
- Stop instance đang gắn → LB chuyển user sang EC2 khác; ASG/ELB health check có thể terminate instance failed.
- Nên thiết kế state bên ngoài instance và tránh stickiness khi có thể; demo xong thì Delete stack.

Summary (Tóm tắt ngắn): Demo thực hành session stickiness trên ALB: so sánh round-robin khi tắt stickiness với việc khóa user qua cookie AWSALB khi bật; kiểm chứng failover khi stop instance, rồi dọn bằng Delete CloudFormation stack.