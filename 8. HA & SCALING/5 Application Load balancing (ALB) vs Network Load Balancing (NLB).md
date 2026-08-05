Chào mừng bạn quay lại. Trong bài này, chúng ta sẽ tìm hiểu chi tiết hơn về Application Load Balancer (ALB) và Network Load Balancer (NLB). Với kỳ thi, rất quan trọng là bạn hiểu khi nào chọn ALB và khi nào chọn NLB — chúng được dùng cho những tình huống hoàn toàn khác nhau.

## Consolidation of Load Balancers (Gộp Load Balancer)

### Cách cũ với Classic Load Balancer (CLB)

Trước đây, khi dùng Classic Load Balancer, bạn gắn instance trực tiếp vào load balancer, hoặc tích hợp Auto Scaling Group trực tiếp với load balancer:

- Một domain: `catagram.io`
- Một Classic Load Balancer
- Một SSL certificate gắn với domain đó
- Một Auto Scaling Group gắn vào load balancer
- CLB phân phối kết nối tới các instance

Vấn đề: cách này không scale được vì:

- Classic Load Balancer không hỗ trợ [[SNI]]
- Không thể có nhiều SSL certificate trên một load balancer
- Mỗi ứng dụng HTTPS riêng biệt đều cần một CLB riêng

Ví dụ: `catagram` và `dogogram` đều là ứng dụng HTTPS → bắt buộc dùng hai Classic Load Balancer khác nhau. Đây là một trong nhiều lý do nên tránh Classic Load Balancer.

![[Pasted image 20260805221448.png]]
### Cách mới với Application Load Balancer

Cùng kiến trúc hai ứng dụng `catagram` và `dogogram`, nhưng lần này dùng một Application Load Balancer duy nhất:

- ALB xử lý cả hai ứng dụng
- Dùng listener-based rules (sẽ nói rõ hơn sau)
- Mỗi rule có thể gắn SSL certificate để xử lý HTTPS cho cả hai domain
- Dùng host-based rules để hướng kết nối tới nhiều Target Group
- Các Target Group chuyển tiếp tới nhiều Auto Scaling Group

→ Đây là consolidation 2 → 1: giảm một nửa số load balancer cần thiết.

Nếu có 100 ứng dụng legacy, mỗi cái một CLB, thì chuyển sang ALB (version 2) mang lại lợi thế lớn — trong đó có consolidation.

---

## Điểm then chốt về Application Load Balancer

Đây là những đặc điểm riêng của ALB (version 2).

### Layer 7 Load Balancer thật sự

- ALB là true Layer 7 load balancer
- Được cấu hình lắng nghe trên HTTP hoặc HTTPS
- Hiểu và diễn giải thông tin của hai protocol ứng dụng Layer 7 này

Mặt trái: ALB không hiểu các protocol Layer 7 khác như:

- SMTP
- SSH
- Protocol game tùy chỉnh

→ Không được hỗ trợ bởi Layer 7 load balancer như ALB.

Ngoài ra:

- ALB phải dùng listener HTTP/HTTPS
- Không thể cấu hình lắng nghe trực tiếp bằng TCP, UDP, hoặc TLS
- Điều này mang lại một số giới hạn quan trọng (sẽ nói sau trong bài)

### Hiểu nội dung Layer 7

Vì là Layer 7, ALB hiểu được:

- Loại nội dung (content type)
- Cookies của ứng dụng
- Custom headers
- Vị trí người dùng (user location)
- Hành vi ứng dụng (application behavior)

ALB có thể inspect toàn bộ thông tin protocol Layer 7 và ra quyết định dựa trên đó.  
NLB không làm được điều này — chỉ Layer 7 load balancer (ALB) mới hiểu các thành phần này.

### SSL/TLS luôn bị terminate trên ALB

Mọi kết nối đến ALB (HTTP hoặc HTTPS) đều bị terminate trên Application Load Balancer.

- HTTPS chỉ là HTTP đi qua SSL/TLS
- Không thể có kết nối SSL không bị ngắt từ client thẳng tới application instance
- Luôn: terminate trên LB → tạo kết nối mới từ LB tới application

Điều này quan trọng với security team. Trong môi trường bảo mật nghiêm ngặt, điều này có thể loại ALB khỏi lựa chọn.

Hệ quả:

- ALB không hỗ trợ end-to-end unbroken SSL encryption giữa client và instance
- Mọi ALB dùng HTTPS phải cài SSL certificate trên load balancer, vì kết nối buộc phải terminate tại đó rồi mới tạo kết nối mới tới instance

### Hiệu năng và Health Check

- ALB chậm hơn NLB vì phải xử lý nhiều tầng mạng hơn
- Càng nhiều tầng stack → càng phức tạp → xử lý càng chậm
- Trong câu hỏi thi khắt khe về performance → cân nhắc NLB hơn ALB

Lợi ích của ALB: vì là Layer 7, có thể đánh giá application health ở Layer 7:

- Không chỉ kiểm tra kết nối mạng thành công
- Có thể gửi application-layer request để xác nhận ứng dụng đang hoạt động đúng

### Rules trên ALB

ALB có khái niệm rules — quyết định xử lý kết nối đến listener:

- Rules được xử lý theo priority order
- Có thể có nhiều rule ảnh hưởng cùng một tập traffic
- Rule cuối cùng được xử lý là default rule (catch-all)

#### Conditions trong rule có thể kiểm tra:

- Host headers
- HTTP headers
- HTTP request methods
- Path patterns
- Query strings
- Source IP

Ví dụ hành vi:

- Domain khác nhau: `catagram` vs `dogogram`
- Path khác nhau: `images` vs `API`
- Quyết định theo query string
- Quyết định theo source IP của client

#### Actions trong rule có thể:

- Forward tới Target Group
- Redirect sang đích khác (ví dụ domain khác)
- Trả fixed HTTP response (error code hoặc success code)
- Thực hiện authentication (OpenID hoặc Cognito)

### Ví dụ trực quan với Rules

Triển khai đơn giản:

- Domain: `catagram.io`
- Một host-based rule + SSL certificate
- Condition: host header
- Action: forward tới Target Group của ứng dụng catagram
![[Pasted image 20260805221942.png]]
#### Rule theo Source IP (corporate client)

Giả sử client của Bowtie Incorporated dùng IP `1.3.3.7` cần phiên bản ứng dụng khác:

- Thêm listener rule với condition = source IP `1.3.3.7`
- Action = forward tới Target Group / Auto Scaling Group riêng cho corporate client

Vì ALB là thiết bị Layer 7, nó nhìn được bên trong HTTP và ra quyết định dựa trên bất kỳ thứ gì đến Layer 7.

Lưu ý quan trọng: kết nối từ LB tới instance của Target Group 2 là tập kết nối riêng. HTTP từ enterprise user bị terminate trên LB, rồi mới có kết nối mới tới application instance. Không có tùy chọn pass-through kết nối đã mã hóa thẳng tới instance.

Nếu bắt buộc forward encrypted connection tới instance mà không terminate trên LB → phải dùng Network Load Balancer.

#### Rule theo Path / Redirect

Vì là Layer 7, bạn cũng có thể:

- Route theo path hoặc header HTTP
- Redirect ở mức HTTP

Ví dụ: cùng ALB cũng nhận traffic `dogogram.io` → tạo rule khớp domain `dogogram.io`, action = redirect sang `catagram.io`.

Đây chỉ là một phần nhỏ tính năng ALB. Vì Layer 7, gần như có thể routing dựa trên mọi thứ quan sát được ở Layer 7 → rất linh hoạt.

---

## Network Load Balancer (NLB)

### Hoạt động ở Layer 4

- NLB là thiết bị Layer 4
- Hiểu được: TCP, TLS, UDP
- Không hiểu HTTP/HTTPS
- Không đọc được headers, cookies
- Không có session stickiness kiểu HTTP (vì phụ thuộc cookie — thực thể Layer 7)

### Hiệu năng rất cao

- NLB rất nhanh
- Có thể xử lý hàng triệu request/giây
- Latency khoảng ~25% so với ALB (thấp hơn nhiều)
- Lý do: không xử lý các tầng trên nặng về tính toán; chỉ làm việc ở Layer 4

### Phù hợp protocol không phải HTTP/HTTPS

Ví dụ:

- SMTP (email)
- SSH
- Game server không dùng web protocol
- Ứng dụng tài chính không dùng HTTP/HTTPS

→ Nếu câu hỏi thi nói về thứ không phải web/secure web, không dùng HTTP/HTTPS → mặc định nghĩ tới NLB.

### Health Check hạn chế hơn

Vì không nhận biết Layer 7:

- Health check của NLB chỉ kiểm tra [[ICMP]] và TCP handshake cơ bản
- Không application-aware
- Không làm được health check chi tiết như ALB

### Static IP & Whitelisting

- NLB có thể được cấp static IP
- Rất hữu ích cho whitelisting với corporate client
- Client có thể whitelist IP của NLB để đi thẳng qua firewall
- Phù hợp môi trường bảo mật nghiêm ngặt

### Forward TCP / End-to-End Encryption

- NLB có thể forward TCP thẳng tới instance
- Vì NLB không hiểu HTTP/HTTPS, bạn có thể cấu hình listener nhận TCP only rồi forward
- Các tầng xây trên TCP không bị terminate trên LB → không bị ngắt
- → Có thể forward kênh mã hóa không bị ngắt từ client tới application instance

Điểm thi quan trọng: muốn unbroken end-to-end encryption → dùng NLB + TCP listeners.

### PrivateLink

- NLB cũng dùng cho PrivateLink để cung cấp service tới VPC khác
- Đây cũng là điểm quan trọng cần nhớ cho kỳ thi

---

## Cách chọn nhanh: NLB hay ALB?

Dễ nhớ hơn nếu nhớ các trường hợp dùng NLB; nếu scenario không thuộc các trường hợp đó → mặc định dùng ALB.

### Chọn Network Load Balancer khi:

1. Cần unbroken encryption giữa client và instance
2. Cần static IP để whitelisting
3. Cần hiệu năng tuyệt đối (millions of requests/sec, low latency)
4. Protocol không phải HTTP/HTTPS
5. Có yêu cầu liên quan PrivateLink

### Còn lại → mặc định Application Load Balancer

Vì các tính năng bổ sung của ALB thường rất giá trị trong hầu hết scenario.

---

Đó là toàn bộ nội dung cần cover về ALB và NLB cho kỳ thi. Hãy hoàn thành video này, rồi khi sẵn sàng hãy tiếp tục bài tiếp theo.

---

Tóm tắt theo Cornell Note

Cues (Từ khóa / Ý chính):

- CLB không SNI → không consolidation; ALB gộp nhiều app
- ALB = Layer 7 (HTTP/HTTPS only)
- SSL terminate trên ALB; không end-to-end unbroken SSL
- ALB Rules: conditions + actions
- NLB = Layer 4 (TCP/TLS/UDP), cực nhanh
- NLB: static IP, TCP passthrough, PrivateLink
- Chọn NLB khi… còn lại mặc định ALB

Notes (Chi tiết):

- Classic LB không hỗ trợ SNI/nhiều SSL cert → mỗi HTTPS app cần 1 CLB; ALB dùng listener/host rules + nhiều cert/target group để gộp nhiều app (ví dụ catagram + dogogram).
- ALB chỉ hiểu HTTP/HTTPS; không hỗ trợ SMTP/SSH/game protocol tùy chỉnh; không listen trực tiếp TCP/UDP/TLS.
- Mọi HTTPS trên ALB đều terminate tại LB rồi mở kết nối mới tới instance → bắt buộc cài SSL cert trên ALB; không làm được unbroken SSL client→instance.
- ALB chậm hơn NLB nhưng health check được ở Layer 7; rules theo priority, có default catch-all; condition gồm host/header/method/path/query/source IP; action gồm forward/redirect/fixed response/auth (OpenID/Cognito).
- NLB Layer 4: không đọc header/cookie/HTTP stickiness; ~millions req/s, latency ~25% của ALB; health check chỉ ICMP/TCP cơ bản.
- NLB mạnh ở: static IP (whitelist), TCP forward cho end-to-end encryption, protocol non-HTTP(S), và PrivateLink.
- Rule chọn nhanh: cần unbroken encryption / static IP / max performance / non-HTTP(S) / PrivateLink → NLB; còn lại → ALB.

Summary (Tóm tắt ngắn): Bài học so sánh ALB (Layer 7) và NLB (Layer 4) để biết khi nào chọn cái nào cho kỳ thi. ALB mạnh về routing linh hoạt theo HTTP, consolidation nhiều app/cert, nhưng luôn terminate SSL và chậm hơn; NLB tối ưu cho hiệu năng cực cao, protocol không phải web, static IP, end-to-end encryption qua TCP, và PrivateLink — ngoài các case đó thì mặc định dùng ALB.