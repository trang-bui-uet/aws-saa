Chào mừng bạn quay lại. Đây là phần hai của bài học. Chúng ta sẽ tiếp tục ngay từ điểm kết thúc của phần một.

![[Pasted image 20260805213322.png]]
## Kiến trúc ứng dụng đa tầng (Multi-tiered Application)

Lần này, chúng ta có một ứng dụng đa tầng điển hình:

- Bắt đầu với một VPC, bên trong có hai Availability Zone.
- Bên trái: một Internet-facing Load Balancer.
- Tiếp theo: một Auto Scaling Group cho các web instance — cung cấp khả năng front-end của ứng dụng.
- Sau đó: một Internal Load Balancer khác — các node chỉ được cấp private IP.
- Tiếp theo: một Auto Scaling Group cho các application instance — được các web server sử dụng cho tầng ứng dụng.
- Bên phải: một cặp database instance — giả sử đây là các instance Aurora.

→ Có ba tầng: Web, Application, và Database.

## Vấn đề khi không có Load Balancer

Không có load balancer, mọi thứ sẽ bị ràng buộc chặt với nhau:

- Người dùng Bob phải giao tiếp với một instance cụ thể ở tầng web. Nếu instance đó fail hoặc scale → trải nghiệm của Bob bị gián đoạn.
- Instance mà Bob kết nối cũng sẽ kết nối tới một instance cụ thể ở tầng application. Nếu instance đó fail hoặc scale → trải nghiệm của Bob lại bị gián đoạn.

## Load Balancer giúp trừu tượng hóa giữa các tầng

Để cải thiện kiến trúc, ta đặt load balancer giữa các tầng ứng dụng nhằm trừu tượng hóa (abstract) tầng này khỏi tầng kia.

Cách hoạt động thay đổi như sau:

- Bob thực tế giao tiếp với một ELB node, và node này chuyển kết nối tới một web server cụ thể.
- Bob không biết mình đang kết nối web server nào, vì giao tiếp qua load balancer.
- Nếu instance được thêm/xóa, Bob cũng không nhận ra, vì đã được trừu tượng hóa khỏi hạ tầng vật lý bởi load balancer.

Web instance mà Bob đang dùng cần giao tiếp với tầng application qua internal load balancer — đây cũng là sự trừu tượng hóa giao tiếp:

- Web instance không biết layout vật lý của tầng application.
- Không biết có bao nhiêu instance, cũng không biết đang giao tiếp với instance nào.

Cuối cùng, application server đang được dùng sẽ sử dụng tầng database cho mọi nhu cầu lưu trữ dữ liệu bền vững (persistent data storage).

## Nới lỏng sự phụ thuộc (Loose Coupling) & Scale độc lập

Không dùng load balancer → các tầng gắn kết chặt (tightly coupled):

- Chúng cần nhận biết lẫn nhau.
- Bob kết nối tới một instance cụ thể ở tầng web.
- Instance đó kết nối tới một instance cụ thể ở tầng application.
- Tất cả các tầng đều cần biết về nhau.

Load balancer loại bỏ một phần sự ràng buộc này — chúng nới lỏng coupling.

Điều này cho phép các tầng:

- Hoạt động độc lập nhờ sự trừu tượng hóa.
- Scale độc lập với nhau.

Ví dụ: nếu tải ở tầng application vượt khả năng phục vụ của 2 instance, tầng application có thể scale từ 2 → 4 instance, độc lập với mọi thứ khác. Tầng web vẫn tiếp tục sử dụng mà không gián đoạn hay cấu hình lại, vì đang giao tiếp qua load balancer và không nhận biết những gì xảy ra bên trong tầng application.

Các ý kiến trúc này sẽ được nói sâu hơn sau trong phần khóa học; hiện tại hãy nắm vững các nguyên lý kiến trúc cơ bản.

---

## Cross-Zone Load Balancing

Một nguyên lý quan trọng khác cần nắm vững là cross-zone load balancing.

### Ví dụ: Bob truy cập WordPress “The Best Cats”

Bob dùng thiết bị và truy cập DNS name của ứng dụng — thực chất là DNS name của load balancer.

Một load balancer theo mặc định có ít nhất một node trên mỗi Availability Zone mà nó được cấu hình. Trong ví dụ (Animals for Life VPC rút gọn, 2 AZ):

- Application Load Balancer có tối thiểu 2 node (mỗi AZ một node).
- DNS name của load balancer phân phối request đến đều trên tất cả các node.
- Với 2 node → mỗi node nhận 50% tải.

Trong production có thể dùng nhiều AZ hơn, và ở throughput cao hơn có thể có nhiều node hơn mỗi AZ; ví dụ này giữ mọi thứ đơn giản. Dù tải đến DNS name load balancer lớn đến đâu, mỗi node vẫn nhận 50% tải đó.

### Hạn chế lịch sử (không có Cross-Zone)

Ban đầu, mỗi load balancer node chỉ phân phối kết nối tới instance trong cùng Availability Zone.

Xét kiến trúc: 4 instance ở AZ A, 1 instance ở AZ B:

|Node|Tải nhận được|Cách chia|Mỗi instance nhận|
|---|---|---|---|
|Node A|50% tổng tải|Chia cho 4 instance ở AZ A|12.5% mỗi instance|
|Node B|50% tổng tải|Chỉ có 1 instance ở AZ B|50% cho instance đó|

→ Phân phối rất không đều vì hạn chế lịch sử này.

### Cross-Zone Load Balancing là gì?

Cross-zone load balancing cho phép mọi load balancer node phân phối kết nối đều tới tất cả instance đã đăng ký, trên mọi Availability Zone.

- Node ở AZ A có thể gửi kết nối tới instance ở AZ B.
- Node ở AZ B có thể gửi kết nối tới instance ở AZ A.

→ Phân phối tải đồng đều hơn nhiều.

### Lưu ý thi

- Tính năng này ban đầu không bật mặc định.
- Với Application Load Balancer, cross-zone load balancing được bật sẵn.
- Vẫn cần nắm chắc cho kỳ thi: câu hỏi thường mô tả vấn đề phân phối tải không đều và đáp án là bật/biết đến tính năng này.

---

## Các điểm kiến trúc quan trọng cần nhớ về Elastic Load Balancer

Nếu chỉ nhớ vài điều từ bài này, hãy nhớ những điểm sau:

### 1. Bạn thấy 1 thiết bị — thực tế là nhiều node

Khi provision một Elastic Load Balancer, bạn thấy nó như một thiết bị chạy trên hai hoặc nhiều AZ — cụ thể là một subnet trong mỗi AZ.

Nhưng thực tế bạn đang tạo:

- Một ELB node trong một subnet của mỗi AZ mà load balancer được cấu hình.
- Một DNS record cho load balancer, phân tán request đến tất cả node đang active.

Ban đầu có một số node nhất định (ví dụ một node mỗi AZ), nhưng sẽ tự động scale nếu tải tăng.

### 2. Cross-zone load balancing

- Mặc định (với ALB): node có thể phân phối request sang AZ khác.
- Lịch sử: từng bị tắt → kết nối có thể không cân bằng.
- Với Application Load Balancer: cross-zone load balancing bật mặc định.

### 3. Hai loại Load Balancer

|Loại|Ý nghĩa|
|---|---|
|Internet-facing|Các node được cấp public IPv4. Không đổi vị trí đặt load balancer — chỉ ảnh hưởng địa chỉ IP của node.|
|Internal|Tương tự, nhưng node chỉ được cấp private IP.|

### 4. Internet-facing LB có thể cân bằng tới instance private

Điểm cực kỳ quan trọng (đặc biệt cho kỳ thi):

- Internet-facing load balancer có thể giao tiếp với public instance hoặc private instance.
- EC2 không cần public IP để làm việc với internet-facing load balancer.
- Node của internet-facing LB có public IP → nhận kết nối từ internet công cộng → cân bằng tới cả public và private EC2.

→ Không cần public instance mới dùng được internet-facing load balancer.

### 5. Listener configuration

Load balancer được cấu hình qua listener configuration — điều khiển load balancer lắng nghe cái gì. Sẽ được nói chi tiết hơn sau trong phần khóa học.

### 6. Yêu cầu IP trống trong subnet

Điểm dễ gây nhầm:

- Load balancer cần ít nhất 8 IP trống trên mỗi subnet mà nó được triển khai vào.
- Về mặt kỹ thuật, subnet /28 là đủ.
- Nhưng tài liệu AWS khuyến nghị /27 để cho phép scale.

---

Đó là mọi thứ cần cover trong bài này. Hãy hoàn thành bài học, rồi khi sẵn sàng hãy tiếp tục bài tiếp theo.

---

Tóm tắt theo Cornell Note

Cues (Từ khóa / Ý chính):

- Multi-tier: Web / App / Database + 2 loại ELB
- Loose coupling nhờ Load Balancer
- Scale độc lập giữa các tầng
- Cross-zone load balancing (vấn đề & cách sửa)
- ALB: cross-zone bật mặc định (thi)
- Internet-facing vs Internal; private backend vẫn OK
- ELB cần ≥8 free IP / subnet (/27 khuyến nghị)

Notes (Chi tiết):

- Kiến trúc 3 tầng trong VPC 2 AZ: Internet-facing LB → Web ASG → Internal LB → App ASG → cặp Aurora.
- Không có LB → Bob gắn với instance cụ thể; fail/scale làm gián đoạn trải nghiệm → các tầng tightly coupled.
- Có LB → Bob/web tier chỉ nói chuyện qua LB → không biết instance vật lý nào đang phục vụ → loose coupling, các tầng scale độc lập (ví dụ App 2→4 instance mà Web không cần reconfigure).
- Mỗi AZ có ≥1 ELB node; DNS chia tải đều giữa các node (2 node → mỗi node 50%).
- Không cross-zone: Node A (4 instance) → mỗi instance 12.5%; Node B (1 instance) → 50% → phân phối lệch nặng.
- Cross-zone load balancing: mỗi node chia đều tới mọi registered instance mọi AZ; trên ALB được enable by default — hay ra đề thi về uneven load.
- Internet-facing = node có public IPv4; Internal = chỉ private IP. Internet-facing LB vẫn cân bằng được tới private EC2 (không cần public instance).
- Cấu hình qua listener. Mỗi subnet triển khai ELB cần ≥8 free IP; docs AWS gợi ý /27 để scale (dù /28 kỹ thuật là đủ).

Summary (Tóm tắt ngắn): Bài học giải thích cách Elastic Load Balancer tách các tầng Web–App–DB khỏi nhau để loose coupling và scale độc lập, đồng thời đi sâu cross-zone load balancing để tránh phân phối tải lệch giữa các AZ. Cần nhớ: ALB bật cross-zone mặc định, internet-facing LB vẫn phục vụ được private instance, và mỗi subnet cần đủ IP trống (≥8, khuyến nghị /27).