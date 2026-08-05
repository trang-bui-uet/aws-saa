**Chào mừng trở lại!** Trong bài học này, tôi muốn dành vài phút để nói về **sự tiến hóa của sản phẩm Elastic Load Balancer**. Việc hiểu rõ lịch sử và trạng thái hiện tại của nó rất quan trọng cho cả kỳ thi lẫn thực tế sử dụng. Đây sẽ là một bài học **rất ngắn**, vì hầu hết chi tiết sẽ được tôi trình bày trong các bài học chuyên sâu ngay sau đây trong phần này của khóa học. Hãy cùng bắt đầu.

Hiện tại có **ba loại Elastic Load Balancer** khác nhau trong AWS. Khi bạn thấy thuật ngữ **ELB** hoặc **Elastic Load Balancers**, nó đề cập đến **toàn bộ gia đình** — cả ba loại.

Các Load Balancer được chia thành **Version 1** và **Version 2**. Bạn nên **tránh sử dụng Load Balancer Version 1** ở thời điểm hiện tại và hướng tới việc **di chuyển** sang các sản phẩm Version 2, vốn được **ưu tiên** cho mọi triển khai mới. Không còn bất kỳ tình huống nào mà bạn nên chọn Version 1 thay vì một trong các loại Version 2.

### Classic Load Balancer (CLB) – Version 1 duy nhất

Sản phẩm Load Balancer bắt đầu với **Classic Load Balancer** (gọi tắt là **CLB**), đây là **Load Balancer Version 1 duy nhất**. Nó được ra mắt năm **2009**, thuộc hàng những sản phẩm AWS cũ nhất.

**Classic Load Balancer** có thể cân bằng tải cho **HTTP** và **HTTPS**, cũng như các giao thức tầng thấp hơn, nhưng chúng **không thực sự là thiết bị Layer 7**. Chúng không hiểu sâu về HTTP và **không thể đưa ra quyết định** dựa trên các tính năng của giao thức HTTP. Chúng thiếu rất nhiều chức năng nâng cao so với các Load Balancer Version 2, và thường **đắt hơn đáng kể** khi sử dụng.

Một hạn chế phổ biến là **Classic Load Balancer chỉ hỗ trợ một chứng chỉ SSL** trên mỗi Load Balancer. Điều này có nghĩa là với các triển khai lớn, bạn có thể cần **hàng trăm hoặc hàng nghìn** Classic Load Balancer, trong khi chúng có thể được **gộp lại chỉ còn một Load Balancer Version 2**.

Tôi không thể nhấn mạnh đủ: với mọi câu hỏi trong kỳ thi hoặc mọi tình huống thực tế, bạn nên **mặc định không sử dụng Classic Load Balancer**.

### Các Load Balancer Version 2

Bây giờ chúng ta đến với các Load Balancer **Version 2** mới hơn.

- **Application Load Balancer (ALB)** Đây là thiết bị **Layer 7** thực thụ (tầng ứng dụng). Chúng hỗ trợ các giao thức **HTTP**, **HTTPS** và **WebSocket**. Đây thường là loại Load Balancer bạn nên chọn cho bất kỳ tình huống nào sử dụng các giao thức này.
- **Network Load Balancer (NLB)** Cũng thuộc Version 2, nhưng hỗ trợ các giao thức **TCP**, **TLS** (dạng TCP bảo mật) và **UDP**. Network Load Balancer là lựa chọn dành cho các ứng dụng **không sử dụng HTTP hoặc HTTPS**. Ví dụ: cân bằng tải cho **máy chủ email**, **máy chủ SSH**, hoặc một trò chơi sử dụng giao thức tùy chỉnh — khi đó bạn sẽ dùng **Network Load Balancer**.

Nhìn chung, các Load Balancer **Version 2** nhanh hơn và hỗ trợ **Target Groups** cùng **Rules**. Điều này cho phép bạn sử dụng **một Load Balancer duy nhất** cho nhiều mục đích khác nhau, hoặc xử lý cân bằng tải khác nhau tùy theo khách hàng đang sử dụng. Tôi sẽ trình bày chi tiết khả năng của từng loại Version 2 cũng như về Rules trong các bài học riêng, nhưng tôi muốn giới thiệu chúng ngay từ bây giờ như một tính năng quan trọng.

### Điểm quan trọng cho kỳ thi

Đối với kỳ thi, bạn **thực sự cần** có khả năng **lựa chọn giữa Network Load Balancer và Application Load Balancer** cho từng tình huống cụ thể. Đó chính là điều chúng ta sẽ tập trung luyện tập trong các bài học sắp tới.

Hiện tại, đây chỉ là bài học **giới thiệu** về sự tiến hóa của các sản phẩm này, và đó là tất cả những gì tôi muốn trình bày trong bài này. Hãy hoàn thành bài học, và khi sẵn sàng, tôi rất mong được gặp bạn ở bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Ba loại Elastic Load Balancer (ELB)
- Version 1 vs Version 2
- Classic Load Balancer (CLB)
- Application Load Balancer (ALB)
- Network Load Balancer (NLB)
- Layer 7 vs Layer 4
- Target Groups & Rules
- Không nên dùng CLB cho triển khai mới

**Notes (Chi tiết):**

- **ELB** là tên gọi chung cho cả ba loại Load Balancer.
- **Version 1**: Chỉ có **Classic Load Balancer (CLB)** – ra mắt năm 2009.
    - Hỗ trợ HTTP/HTTPS và giao thức thấp hơn.
    - **Không phải Layer 7 thực thụ**, thiếu nhiều tính năng nâng cao.
    - Chỉ hỗ trợ **1 SSL certificate** → dễ phải dùng rất nhiều CLB.
    - Đắt hơn và nên **tránh hoàn toàn** cho triển khai mới.
- **Version 2** (nên ưu tiên):
    - **Application Load Balancer (ALB)**: Layer 7 thực thụ → HTTP, HTTPS, WebSocket.
    - **Network Load Balancer (NLB)**: Hỗ trợ TCP, TLS, UDP → dùng cho ứng dụng không phải HTTP/HTTPS (email, SSH, game protocol…).
- Version 2 nhanh hơn, hỗ trợ **Target Groups** và **Rules** → có thể dùng một Load Balancer cho nhiều mục đích.
- Kỳ thi yêu cầu khả năng **phân biệt và chọn đúng** giữa ALB và NLB tùy theo tình huống.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu sự tiến hóa của **Elastic Load Balancer**. Hiện có 3 loại: **Classic Load Balancer (Version 1 – không nên dùng nữa)**, **Application Load Balancer (ALB – Layer 7 cho HTTP/HTTPS/WebSocket)** và **Network Load Balancer (NLB – cho TCP/TLS/UDP)**. Luôn ưu tiên Version 2 cho mọi triển khai mới. Kỳ thi đòi hỏi khả năng chọn đúng loại Load Balancer dựa trên giao thức và yêu cầu cụ thể.