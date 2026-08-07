Chào mừng bạn quay lại. Trong bài học này, mình sẽ nói về Auto Scaling groups và health checks. Nội dung khá ngắn, nhưng rất quan trọng cho kỳ thi. Hãy bắt đầu luôn.

## Health checks làm gì?

Auto Scaling groups đánh giá tình trạng sức khỏe của các instance trong nhóm bằng health checks. Nếu một instance fail health check, nó sẽ bị thay thế trong Auto Scaling group. Đây là cách tự động chữa lành (auto healing) các instance trong nhóm.

## Ba loại health check

Có ba loại health check dùng với Auto Scaling groups:

1. EC2 — mặc định
2. ELB checks — có thể bật trên Auto Scaling group
3. Custom health checks

### 1. EC2 checks (mặc định)

Với EC2 checks, bất kỳ trạng thái nào không phải running đều được xem là unhealthy. Cụ thể, instance bị coi là unhealthy nếu đang:

- stopping
- stopped
- terminated
- shutting down
- hoặc impaired — nghĩa là không đạt 2/2 status checks

### 2. Load balancer (ELB) health checks

Khi dùng option này, để instance được xem là healthy, nó phải:

- đang running, và
- pass load balancer health check

Điểm quan trọng: nếu dùng Application Load Balancer (ALB), các check này có thể nhận biết ứng dụng (application-aware). Bạn có thể:

- chỉ định một trang cụ thể của ứng dụng làm health check
- dùng text pattern matching

Khi tích hợp với Auto Scaling group, các kiểm tra mà ASG thực hiện trở nên application-aware hơn nhiều.

### 3. Custom health checks

Với custom health checks, một hệ thống bên ngoài có thể đánh dấu instance là healthy hoặc unhealthy. Cách này giúp mở rộng khả năng health check của Auto Scaling group bằng quy trình riêng theo business hoặc công cụ bên ngoài.

## Health check grace period

Mặc định, health check grace period là 300 giây (5 phút). Đây là giá trị có thể cấu hình: phải hết thời gian này thì health checks mới bắt đầu có hiệu lực với instance đó.

Ví dụ với 300 giây: hệ thống có 5 phút để:

- launch
- thực hiện bootstrapping
- chạy các bước startup / cấu hình ứng dụng

trước khi có thể bị đánh fail health check.

Điều này rất hữu ích khi bạn bootstrap EC2 instances do Auto Scaling group launch. Đây cũng là điểm thường gặp trong kỳ thi, và thường là nguyên nhân ASG liên tục provision rồi terminate instance:

- Nếu grace period quá ngắn, health checks có hiệu lực trước khi ứng dụng cấu hình xong.
- Instance bị xem là unhealthy → bị terminate → instance mới được provision.
- Chu kỳ này lặp lại liên tục.

Bạn cần biết instance mất bao lâu để launch + bootstrap + cấu hình, rồi đặt health check grace period đúng bằng khoảng thời gian đó.

---

Đó là toàn bộ nội dung bài lý thuyết ngắn này — nhằm giúp bạn nắm các lựa chọn health check trong Auto Scaling groups. Hãy hoàn thành video, và khi sẵn sàng, mình sẽ gặp bạn ở bài tiếp theo.

---

Tóm tắt theo Cornell Note

Cues (Từ khóa / Ý chính):

- Auto healing bằng health checks trong ASG
- Ba loại: EC2, ELB, Custom
- EC2 check: mọi trạng thái ≠ running = unhealthy
- ELB/ALB: application-aware health checks
- Custom health checks từ hệ thống bên ngoài
- Health check grace period mặc định 300s
- Grace period quá ngắn → vòng lặp terminate/provision

Notes (Chi tiết):

- ASG dùng health checks để đánh giá instance; fail thì thay thế → auto healing.
- EC2 (mặc định): stopping/stopped/terminated/shutting down/impaired (không đạt 2/2 status checks) đều = unhealthy.
- ELB checks: instance phải running và pass load balancer check; với ALB có thể check theo trang ứng dụng + text pattern matching → application-aware.
- Custom: hệ thống ngoài đánh dấu healthy/unhealthy, mở rộng theo business/tool riêng.
- Grace period mặc định 300 giây (5 phút) — thời gian chờ trước khi health check có hiệu lực (dùng cho bootstrap/startup).
- Grace period không đủ dài → app chưa sẵn sàng đã bị fail → terminate → provision mới → lặp vô hạn; cần đặt bằng thời gian launch + bootstrap + config thực tế.

Summary (Tóm tắt ngắn): Bài học giải thích cách Auto Scaling groups dùng health checks để auto heal instance, với ba loại EC2, ELB/ALB, và custom. Cần cấu hình đúng health check grace period (mặc định 300s) để tránh ASG liên tục terminate/provision khi app chưa kịp khởi động.