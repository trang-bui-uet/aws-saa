**RDS Proxy** là một tính năng quan trọng của Amazon RDS. Nó không chỉ hữu ích độc lập mà còn hỗ trợ nhiều kiến trúc khác liên quan đến RDS. Hãy cùng tìm hiểu chi tiết.

### Tại sao nên sử dụng RDS Proxy?

Mở và đóng kết nối đến cơ sở dữ liệu **tốn thời gian** và **tiêu tốn tài nguyên**. Đây thường là phần lớn chi phí của các thao tác database nhỏ. Khi bạn chỉ cần đọc/ghi một lượng dữ liệu rất nhỏ, **overhead** của việc thiết lập kết nối có thể chiếm tỷ trọng lớn.

Đặc biệt rõ ràng khi sử dụng **serverless**. Nếu có nhiều hàm **Lambda** liên tục gọi hoặc truy cập RDS, sẽ tạo ra rất nhiều kết nối mở-đóng liên tục. Trong khi bạn chỉ bị tính phí theo thời gian compute thực tế của Lambda.

Một vấn đề quan trọng khác là **xử lý sự cố** khi database instance bị lỗi rất khó:

- Phải chờ kết nối bao lâu?
- Ứng dụng nên làm gì trong lúc chờ?
- Khi nào thì coi là thất bại?
- Phản ứng ra sao?
- Làm thế nào để xử lý failover sang standby instance của RDS?

Việc xử lý tất cả những điều này trong code ứng dụng tạo ra **overhead lớn** và **rủi ro cao**.

Database proxy có thể giúp giải quyết, nhưng nếu bạn chưa có kinh nghiệm hoặc không muốn quản lý proxy ở quy mô lớn thì **RDS Proxy** chính là giải pháp mang lại giá trị.

### RDS Proxy hoạt động như thế nào?

Ở mức cao, **RDS Proxy** (và bất kỳ database proxy nào) thay đổi kiến trúc:

- Thay vì ứng dụng kết nối trực tiếp đến database mỗi lần sử dụng,
- Ứng dụng kết nối đến **proxy**,
- Proxy duy trì một **pool kết nối dài hạn** đến database.

Các kết nối từ client đến proxy có thể sử dụng lại pool kết nối đã được thiết lập sẵn. Proxy còn hỗ trợ **multiplexing**: duy trì số lượng kết nối đến database **ít hơn** so với số kết nối từ client đến proxy, và chia sẻ (multiplex) các request trên pool đó. Điều này đặc biệt hữu ích với các database instance nhỏ, nơi tài nguyên bị hạn chế.

### Ví dụ kiến trúc sử dụng RDS Proxy

Giả sử có một VPC ở us-east-1 với 3 Availability Zone và 3 subnet trong mỗi AZ.

- Ở AZ-b: Primary RDS instance đang replicate sang Standby ở AZ-c.
- Ứng dụng **Catagram** chạy trên các web subnet.
- Một số Lambda function được cấu hình VPC networking, chạy từ subnet ở AZ-b (có Lambda ENI).

**Không có RDS Proxy**:

- Các EC2 của Catagram kết nối trực tiếp đến database mỗi lần cần dữ liệu.
- Mỗi lần Lambda được invoke cũng phải kết nối trực tiếp → tăng đáng kể thời gian chạy.

**Có RDS Proxy**:

- Proxy là dịch vụ managed, chạy bên trong VPC, trải rộng cả 3 AZ (a, b, c).
- Proxy duy trì **pool kết nối dài hạn** đến primary node ở AZ-b. Các kết nối này được tạo và giữ lâu dài, không phụ thuộc vào từng request của ứng dụng hay Lambda.
- Client (EC2 của Catagram và Lambda) kết nối đến **RDS Proxy** thay vì database.
- Kết nối đến proxy thiết lập rất nhanh và **không gây tải** lên database server.
- Pool kết nối giữa Proxy và database được **tái sử dụng**. Ngay cả khi Lambda invoke liên tục, chúng vẫn dùng chung bộ kết nối dài hạn.
- **Multiplexing** giúp số kết nối thực tế đến database ít hơn nhiều so với số kết nối từ client → giảm tải cho database server.
- Khi xảy ra sự cố hoặc failover, Proxy **che giấu** hoàn toàn khỏi ứng dụng. Client vẫn kết nối đến cùng một endpoint (RDS Proxy) và chờ. Proxy sẽ tự thiết lập kết nối mới đến primary mới ở phía sau.

![[Pasted image 20260730220503.png]]

### Khi nào nên dùng RDS Proxy? (quan trọng cho exam)

- Khi gặp lỗi **“too many connections”** → Proxy giúp giảm số kết nối thực tế đến database. Đặc biệt quan trọng với instance nhỏ (T2, T3) hoặc instance dạng burstable.
- Khi sử dụng **AWS Lambda**: không phải thiết lập/hủy kết nối mỗi lần invoke; tái sử dụng pool dài hạn của Proxy; hỗ trợ IAM authentication sẵn có của Lambda execution role.
- Ứng dụng chạy lâu dài (SaaS) cần **độ trễ thấp**: không phải thiết lập kết nối mới mỗi lần người dùng tương tác.
- Khi **khả năng chịu lỗi database** là ưu tiên: client chỉ kết nối đến Proxy, Proxy xử lý failover → giảm thời gian failover và làm cho quá trình hoàn toàn trong suốt với ứng dụng.

Client luôn kết nối đến **một endpoint duy nhất** của RDS Proxy. Ngay cả khi failover xảy ra phía sau, ứng dụng không nhận ra và không cần chờ CNAME của database chuyển từ primary sang standby.

### Các điểm quan trọng cần nhớ (Key Facts)

- **RDS Proxy** là database proxy **fully managed**, dùng được với **RDS** và **Aurora**.
- **Auto-scaling** và **highly available** mặc định → overhead quản trị thấp hơn nhiều so với tự quản lý proxy.
- Cung cấp **connection pooling** → giảm đáng kể tải database nhờ hai lý do:
    1. Không còn liên tục mở/đóng kết nối.
    2. **Multiplexing** → số kết nối giữa Proxy và database ít hơn số kết nối từ client đến Proxy.
- Chỉ truy cập được **từ trong VPC** (không thể truy cập từ internet công cộng).
- Client kết nối qua **proxy endpoint** (giống hệt endpoint database thông thường, hoàn toàn trong suốt với ứng dụng).
- Có thể **bắt buộc SSL/TLS**.
- Giảm thời gian failover **hơn 60%** với Aurora (khoảng 66–67% so với kết nối trực tiếp).
- **Che giấu hoàn toàn** sự cố database khỏi ứng dụng: ứng dụng chỉ cần chờ, Proxy sẽ tự kết nối lại đến instance mới và tiếp tục phục vụ request.

Đó là toàn bộ nội dung bài học tổng quan về **RDS Proxy**.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- RDS Proxy là gì & tại sao cần dùng
- Overhead mở/đóng connection + vấn đề với Lambda
- Connection pooling + Multiplexing
- Kiến trúc VPC với Primary/Standby + Lambda
- Failover trong suốt với ứng dụng
- Use cases quan trọng cho exam
- Key facts: fully managed, VPC only, giảm failover >60%

**Notes (Chi tiết):**

- Mở/đóng connection tốn tài nguyên; đặc biệt nặng với Lambda vì mỗi invoke đều tạo connection mới.
- RDS Proxy duy trì **pool kết nối dài hạn** và dùng **multiplexing** → số connection thực tế đến DB ít hơn nhiều so với client.
- Client (EC2 + Lambda) chỉ kết nối đến **Proxy endpoint**; Proxy tự xử lý kết nối đến Primary/Standby.
- Khi failover xảy ra, ứng dụng vẫn giữ nguyên kết nối đến Proxy và chờ; Proxy tự kết nối lại ở phía sau.
- Nên dùng khi: gặp “too many connections”, dùng Lambda, ứng dụng SaaS cần latency thấp, hoặc cần resilience cao.
- Chỉ truy cập được từ trong VPC; hỗ trợ SSL/TLS; giảm thời gian failover Aurora hơn 60%.

**Summary (Tóm tắt ngắn):** **RDS Proxy** là dịch vụ proxy fully managed cho RDS/Aurora, cung cấp connection pooling và multiplexing để giảm tải database, đặc biệt hữu ích với Lambda và các instance nhỏ. Nó giúp failover trở nên trong suốt với ứng dụng và giảm thời gian failover đáng kể, đồng thời chỉ hoạt động bên trong VPC.