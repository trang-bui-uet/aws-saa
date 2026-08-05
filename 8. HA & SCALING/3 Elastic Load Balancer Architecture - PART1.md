Chào mừng bạn quay lại. Trong bài học này, tôi muốn nói về **kiến trúc của Elastic Load Balancer**. Tôi sẽ đề cập khá chi tiết về load balancer trong phần này của khóa học, vì vậy bài học này sẽ đóng vai trò như một **nền tảng**. Tôi sẽ trình bày kiến trúc logic và vật lý ở mức cao của sản phẩm, nhằm giúp bạn ôn lại một số kiến thức hoặc lần đầu tiếp cận những điểm chi tiết hơn về load balancing.

Trước khi bắt đầu, hãy nhớ rằng **nhiệm vụ của một load balancer** là nhận kết nối từ khách hàng rồi **phân phối** các kết nối đó đến các tài nguyên compute backend đã đăng ký. Điều này giúp người dùng được **trừu tượng hóa** khỏi hạ tầng vật lý. Số lượng hạ tầng có thể tăng hoặc giảm mà không ảnh hưởng đến khách hàng. Và vì hạ tầng vật lý bị trừu tượng hóa, nên hạ tầng có thể bị lỗi và được sửa chữa mà khách hàng hoàn toàn không biết.

Với phần ôn nhanh đó, chúng ta sẽ bắt đầu đi vào kiến trúc của Elastic Load Balancer.

![[Pasted image 20260803230851.png]]

Tôi sẽ đi qua một số điểm kiến trúc quan trọng một cách trực quan. Bắt đầu với một **VPC** sử dụng hai Availability Zone: **AZ-A** và **AZ-B**. Trong các AZ này có một số subnet: hai subnet public và một số subnet private. Tiếp theo, tôi thêm một người dùng tên **Bob**, cùng với một cặp load balancer.

Như đã đề cập, nhiệm vụ của load balancer là nhận kết nối từ người dùng rồi phân phối đến các dịch vụ backend. Trong ví dụ này, chúng ta giả định các dịch vụ đó là compute chạy lâu dài hoặc **EC2**, nhưng như bạn sẽ thấy sau này, không nhất thiết phải là EC2. Đặc biệt, **Application Load Balancer** hỗ trợ nhiều loại dịch vụ compute khác nhau, không chỉ EC2.

Khi bạn **provision** một load balancer, bạn phải quyết định một số cấu hình quan trọng:

- Chọn **IPv4 only** hoặc **dual-stack** (dual-stack nghĩa là sử dụng cả IPv4 và IPv6).
- Chọn các **Availability Zone** mà load balancer sẽ sử dụng. Cụ thể, bạn chọn **một subnet** trong **hai hoặc nhiều Availability Zone**.

Điểm này rất quan trọng vì nó dẫn đến kiến trúc thực sự của Elastic Load Balancer. Dựa trên các subnet bạn chọn trong từng AZ, khi provision load balancer, sản phẩm sẽ **đặt một hoặc nhiều load balancer node** vào các subnet đó. Những gì bạn thấy là một đối tượng load balancer duy nhất thực chất được tạo thành từ **nhiều node**, và các node này nằm trong các subnet bạn đã chọn. Vì vậy, khi tạo load balancer, bạn chọn AZ bằng cách chọn **một và chỉ một subnet** trong mỗi AZ đó. Trong ví dụ trên màn hình, tôi đã chọn subnet public ở AZ-A và AZ-B, nên sản phẩm đã triển khai một hoặc nhiều load balancer node vào mỗi subnet đó.

Khi load balancer được tạo, nó thực sự được tạo kèm theo **một DNS record duy nhất** (loại A record). A record này trỏ đến **tất cả** các Elastic Load Balancer node được tạo ra. Mọi kết nối sử dụng DNS name của load balancer thực chất đều được gửi đến các node của load balancer đó. DNS name resolve thành tất cả các node riêng lẻ. Điều này có nghĩa là mọi request đến sẽ được **phân phối đều** trên tất cả các node của load balancer. Các node này nằm ở nhiều Availability Zone và có thể **scale** bên trong từng AZ. Vì vậy chúng có tính **highly available**: nếu một node bị lỗi, nó sẽ được thay thế; nếu tải đến load balancer tăng lên, các node bổ sung sẽ được provision bên trong mỗi subnet mà load balancer được cấu hình sử dụng.

Một lựa chọn khác rất quan trọng khi tạo load balancer (và quan trọng cho kỳ thi) là quyết định load balancer đó sẽ là **internet-facing** hay **internal**. Lựa chọn này kiểm soát việc **địa chỉ IP** của các load balancer node:

- Nếu chọn **internet-facing**, các node sẽ được cấp cả **public IP** và **private IP**.
- Nếu chọn **internal**, các node chỉ có **private IP**.

Đó là điểm khác biệt duy nhất. Về mặt kiến trúc, chúng giống nhau hoàn toàn: cùng có các node và các tính năng load balancer như nhau. Sự khác biệt duy nhất giữa internet-facing và internal là việc các node có được cấp public IP hay không.

Các kết nối từ khách hàng đến các load balancer node được xử lý thông qua cấu hình **Listener**. Như tên gọi, cấu hình này kiểm soát những gì load balancer đang lắng nghe — tức là **protocol** và **port** nào sẽ được chấp nhận ở phía trước (front side) của load balancer. Sẽ có một bài học riêng sau này tập trung cụ thể vào cấu hình Listener. Ở đây tôi chỉ muốn giới thiệu khái niệm.

Đến thời điểm này, Bob đã khởi tạo kết nối đến DNS name của load balancer, nghĩa là anh ấy đã kết nối đến các load balancer node trong kiến trúc của chúng ta. Sau đó, các load balancer node có thể tạo kết nối đến các instance đã đăng ký với load balancer này. Load balancer **không quan tâm** instance đó là public EC2 (có public IP) hay private EC2 (nằm trong private subnet và chỉ có private address).

Tôi muốn nhấn mạnh điều này vì đây thường là điểm gây nhầm lẫn cho người mới học load balancer. Một **internet-facing load balancer** (nghĩa là các node có public address nên có thể kết nối từ internet công cộng) **có thể kết nối đến cả public lẫn private EC2 instance**. Các instance được sử dụng **không bắt buộc** phải là public. Điều này quan trọng vì trong kỳ thi, khi gặp câu hỏi về số lượng subnet hoặc số tầng (tier) cần thiết cho một ứng dụng, nó sẽ kiểm tra kiến thức của bạn rằng internet-facing load balancer **không yêu cầu** instance phải là private hay public — nó có thể làm việc với cả hai. Yêu cầu duy nhất là các load balancer node **có thể giao tiếp** được với các backend instance. Việc giao tiếp này có thể xảy ra dù instance có public address hay chỉ có private address. Điểm quan trọng là: nếu bạn muốn load balancer có thể truy cập được từ internet công cộng, nó **phải** là internet-facing load balancer vì về logic, nó cần được cấp public address.

Để hoạt động, load balancer cần **tám hoặc nhiều hơn** địa chỉ IP trống trong các subnet mà chúng được triển khai. Nói một cách chặt chẽ, điều này tương đương với subnet **/28** (cung cấp tổng 16 IP, trừ 5 IP bị AWS giữ lại, còn lại 11 IP trống mỗi subnet). Tuy nhiên, AWS **khuyến nghị** sử dụng subnet **/27 hoặc lớn hơn** để triển khai Elastic Load Balancer nhằm đảm bảo khả năng scale. Hãy nhớ rằng, về mặt kỹ thuật, cả /28 và /27 đều đúng theo cách riêng của chúng khi nói về kích thước subnet tối thiểu cho load balancer. AWS ghi trong tài liệu là cần /27, nhưng họ cũng nói cần tối thiểu 8 IP trống. Về logic, một /28 chỉ còn 11 IP trống sẽ không đủ không gian để triển khai cả load balancer lẫn các backend instance. Vì vậy trong hầu hết trường hợp, tôi khuyên bạn nhớ **/27** là giá trị đúng cho kích thước tối thiểu của load balancer. Nhưng nếu bạn thấy câu hỏi chỉ hiện /28 mà không có /27, thì /28 có thể là đáp án đúng.

**Internal load balancer** về kiến trúc giống hệt internet-facing load balancer, ngoại trừ việc các node chỉ được cấp private IP. Do đó, internal load balancer thường được dùng để **tách các tầng (tier)** của ứng dụng. Trong ví dụ này, người dùng Bob kết nối qua internet-facing load balancer đến web server, sau đó web server có thể kết nối đến application server thông qua một internal load balancer. Điều này cho phép chúng ta tách các tầng ứng dụng và scale độc lập.

Okay, đến đây là kết thúc **phần 1** của bài học này. Nội dung hơi dài một chút, và tôi muốn cho bạn cơ hội nghỉ ngắn, đứng dậy duỗi chân hoặc pha cà phê. **Phần 2** sẽ tiếp tục ngay từ điểm này, vậy hãy hoàn thành video này, và khi sẵn sàng, tôi mong được gặp bạn ở phần 2.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Nhiệm vụ & lợi ích của Load Balancer
- Cấu trúc node của Elastic Load Balancer
- Lựa chọn AZ & subnet khi tạo LB
- DNS A record & phân phối traffic
- Internet-facing vs Internal Load Balancer
- Listener configuration
- Kết nối đến public & private instance
- Yêu cầu IP trống trong subnet (/27 vs /28)
- Ứng dụng Internal LB để tách tầng

**Notes (Chi tiết):**

- **Load Balancer** nhận kết nối từ client rồi phân phối đến backend compute → trừu tượng hóa hạ tầng, cho phép scale và xử lý lỗi mà client không bị ảnh hưởng.
- Khi tạo LB, chọn **IPv4 only** hoặc **dual-stack**, và chọn **một subnet** trong **ít nhất 2 AZ**.
- LB thực chất gồm **nhiều node** được đặt vào các subnet đã chọn. Một **DNS A record** duy nhất trỏ đến tất cả các node → traffic được phân phối đều, có tính **highly available** và tự scale.
- **Internet-facing**: node có cả public + private IP. **Internal**: node chỉ có private IP. Về kiến trúc và tính năng thì giống nhau hoàn toàn.
- **Listener** quyết định protocol và port mà LB lắng nghe ở phía trước.
- Internet-facing LB **có thể** kết nối đến cả public lẫn private EC2 instance. Chỉ cần node giao tiếp được với backend.
- Cần tối thiểu **8 IP trống** trong subnet. AWS khuyến nghị dùng **/27 hoặc lớn hơn** để scale tốt. /28 vẫn technically đúng (còn 11 IP) nhưng thường không đủ thực tế.
- Internal LB thường dùng để tách các tầng ứng dụng (ví dụ: web tier → app tier), cho phép scale độc lập.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu kiến trúc nền tảng của **Elastic Load Balancer**. LB thực chất gồm nhiều node nằm trong các subnet thuộc nhiều AZ, được truy cập qua một DNS A record duy nhất. Điểm quan trọng cần nắm cho kỳ thi là sự khác biệt giữa **internet-facing** và **internal**, khả năng internet-facing LB kết nối được cả public lẫn private instance, và yêu cầu kích thước subnet tối thiểu (ưu tiên **/27**).