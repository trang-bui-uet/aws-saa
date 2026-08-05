**SNI (Server Name Indication)** là phần mở rộng của TLS: khi client bắt đầu HTTPS, nó gửi kèm **tên domain** muốn truy cập (ví dụ `catagram.io` hay `dogogram.io`). Nhờ đó, một load balancer/server có thể chọn đúng **SSL certificate** cho đúng domain.

**Classic Load Balancer không hỗ trợ SNI** nghĩa là:

- CLB **không đọc** được “client đang hỏi domain nào” lúc bắt tay TLS
- Vì vậy **một CLB chỉ gắn được một SSL certificate**
- Mỗi ứng dụng HTTPS khác domain thường cần **một CLB riêng**

Ví dụ:
- `catagram.io` → CLB #1 + cert catagram  
- `dogogram.io` → CLB #2 + cert dogogram  

**ALB hỗ trợ SNI** nên một ALB có thể gắn nhiều certificate và phục vụ nhiều HTTPS domain trên cùng một load balancer.