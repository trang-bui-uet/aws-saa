Chào mừng bạn quay lại. Trong bài ngắn này, mình sẽ đề cập hai tính năng của dòng sản phẩm Elastic Load Balancer: SSL offload và session stickiness. Bạn cần nắm kiến trúc của cả hai cho kỳ thi — không cần chi tiết triển khai, lý thuyết kiến trúc mới là phần quan trọng. Hãy bắt đầu.

Load balancer có ba cách xử lý kết nối bảo mật: bridging, pass-through, và offload. Mỗi cách có ưu/nhược điểm; để thi và thiết kế tốt, bạn cần hiểu kiến trúc cùng điểm mạnh/yếu của từng cách.

## 1. Bridging mode (mặc định của Application Load Balancer)

Với bridging mode:

- Client kết nối tới load balancer qua listener HTTPS.
- Kết nối SSL giữa client và load balancer bị terminate (giải mã) ngay trên load balancer.
- Load balancer cần SSL certificate khớp với domain name của ứng dụng.
- Về lý thuyết, AWS có mức độ truy cập nhất định tới certificate đó — cần lưu ý nếu framework bảo mật yêu cầu kiểm soát chặt nơi lưu certificate.

Sau khi terminate kết nối từ client:

- Load balancer tạo kết nối thứ hai tới backend (ví dụ EC2).
- HTTPS = HTTP bọc trong lớp bảo mật. Khi SSL bị terminate, lớp SSL bị gỡ → load balancer thấy được HTTP (plain text) bên trong.
- Vì vậy, ALB ở bridging mode có thể đọc HTTP và đưa quyết định dựa trên nội dung — đây là lý do bridging là mode mặc định của ALB, và cũng là lý do ALB bắt buộc cần SSL certificate.

Sau đó:

- Load balancer mã hóa lại HTTP và gửi tới EC2.
- EC2 cũng cần SSL certificate khớp domain để giải mã.
- Mọi EC2 backend đều phải thực hiện cryptographic operations → với ứng dụng lưu lượng cao, overhead có thể rất lớn.

Ưu điểm: Load balancer thấy được HTTP chưa mã hóa → có thể hành động dựa trên nội dung.

Nhược điểm:

- Certificate phải lưu trên load balancer → rủi ro bảo mật.
- EC2 cũng cần bản copy certificate → overhead quản trị.
- EC2 phải tốn compute cho crypto.

## 2. SSL pass-through

Kiến trúc này khác hoàn toàn:

- Client kết nối, load balancer chỉ chuyển tiếp tới một backend instance — không giải mã.
- Mã hóa được giữ nguyên từ client tới backend.
- Instance vẫn cần cài SSL certificate, nhưng load balancer không cần.
- Thường dùng với Network Load Balancer (NLB).
- Listener cấu hình TCP → NLB thấy source/destination IP và ports để cân bằng tải cơ bản, nhưng không đụng tới encryption.
- Kết nối là một tunnel mã hóa từ client xuyên suốt tới backend.

Ưu điểm: AWS không cần thấy certificate của bạn — bạn toàn quyền quản lý; thậm chí có thể dùng CloudHSM để tăng bảo mật hơn.

Nhược điểm:

- Không load balance theo nội dung HTTP vì không bao giờ được giải mã.
- Instance vẫn cần certificate và vẫn phải làm crypto (tốn compute).

## 3. SSL offload

Với SSL offload:

- Client kết nối HTTPS tới load balancer; kết nối bị terminate trên load balancer.
- Load balancer cần SSL certificate khớp tên ứng dụng.
- Nhưng load balancer kết nối tới backend bằng HTTP — không mã hóa lại.

Ý nghĩa:

- Phía khách hàng: dữ liệu vẫn được mã hóa trên public internet (client ↔ load balancer).
- Từ load balancer → EC2: dữ liệu đi dạng plain text trên mạng AWS.
- Certificate chỉ cần trên load balancer; EC2 không cần certificate.
- EC2 chỉ xử lý HTTP → không làm crypto → giảm overhead mỗi instance, có thể dùng instance nhỏ hơn.

Nhược điểm: dữ liệu plain text trên mạng AWS — nếu chấp nhận được thì đây là giải pháp rất hiệu quả.

## Session stickiness

Connection stickiness rất quan trọng khi thiết kế giải pháp có khả năng mở rộng với load balancer.

### Không có stickiness

Ví dụ: user Bob → load balancer → nhiều EC2 backend.

- Mọi session được phân phối theo cân bằng tải và health checks → phân bố khá đều.
- Vấn đề: nếu ứng dụng không quản lý session bên ngoài, mỗi lần Bob sang instance mới sẽ như bắt đầu lại (phải login lại, giỏ hàng trống…).
- Ứng dụng stateless (state lưu ở DynamoDB, v.v.) dùng kiến trúc non-sticky ổn.
- Nếu state nằm trên một server cụ thể, không thể phân tán session thoải mái — chuyển server sẽ ảnh hưởng trải nghiệm.

### Có session stickiness (ALB)

Trên Application Load Balancer, stickiness được bật ở target group:

- Lần đầu user gửi request, load balancer tạo cookie AWSALB.
- Thời hạn cookie do bạn đặt: từ 1 giây đến 7 ngày.
- Các request sau kèm cookie → session của user đó luôn về cùng một backend (ví dụ EC2-2).

Stickiness duy trì cho đến khi:

1. Server fail (ví dụ EC2-2 hỏng) → user được chuyển sang instance khác; hoặc
2. Cookie hết hạn → quy trình lặp lại, user nhận cookie mới và được gán backend mới.

Mục đích: cho phép ứng dụng dùng load balancer khi session state lưu trên từng server.

Vấn đề: có thể gây tải không đều — một user dù tạo tải lớn cũng chỉ dùng một server.

Khuyến nghị: thiết kế stateless servers, lưu session/state bên ngoài (ví dụ DynamoDB). Khi đó EC2 hoàn toàn stateless, load balancer phân phối công bằng mà không cần cookie.

---

Đó là toàn bộ về connection stickiness và kết thúc bài học. Hai kỹ thuật này khá quan trọng cho kỳ thi. Hãy hoàn thành video, và khi sẵn sàng, mình sẽ gặp bạn ở bài tiếp theo.

---

Tóm tắt theo Cornell Note

Cues (Từ khóa / Ý chính):

- Ba cách xử lý SSL: bridging, pass-through, offload
- Bridging (ALB mặc định): terminate rồi mã hóa lại
- Pass-through (NLB/TCP): không giải mã, AWS không thấy cert
- SSL offload: terminate trên LB, backend dùng HTTP
- Session stickiness & cookie AWSALB
- Thời hạn cookie: 1 giây – 7 ngày
- Stateless vs state trên server (DynamoDB)

Notes (Chi tiết):

- Bridging: client↔LB và LB↔EC2 đều HTTPS; ALB thấy HTTP → quyết định theo nội dung; cert trên cả LB và EC2; crypto trên mọi instance; rủi ro lưu cert trên LB + overhead quản trị/compute.
- Pass-through: NLB listener TCP, chuyển tiếp tunnel mã hóa; chỉ EC2 có cert; AWS không thấy cert (có thể dùng CloudHSM); không LB theo HTTP.
- SSL offload: terminate HTTPS trên LB, backend HTTP plain text; cert chỉ trên LB; EC2 không crypto → giảm overhead; dữ liệu plain text trên mạng AWS.
- Stickiness (ALB target group): cookie AWSALB gắn user vào một backend; hết khi server fail hoặc cookie expire.
- Ứng dụng nên stateless (state ở DynamoDB…) để cân bằng tải công bằng, tránh phụ thuộc stickiness.

Summary (Tóm tắt ngắn): Bài học so sánh ba kiến trúc SSL trên ELB — bridging, pass-through, offload — về chỗ terminate, nơi lưu certificate, và overhead crypto. Đồng thời giải thích session stickiness (cookie AWSALB, 1s–7 ngày) để giữ user trên một server khi state local, đồng thời khuyến nghị thiết kế stateless với state bên ngoài.