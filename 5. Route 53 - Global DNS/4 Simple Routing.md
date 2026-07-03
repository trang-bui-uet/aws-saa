Chào mừng bạn quay lại, và trong video này, tôi muốn trình bày về **chính sách định tuyến (routing policy)** đầu tiên trong số nhiều chính sách có sẵn trong **Route 53**. Chúng ta sẽ bắt đầu với chính sách mặc định, và như tên gọi cho thấy, đây là chính sách **đơn giản nhất**. Video này sẽ khá ngắn, vì vậy hãy cùng nhảy vào ngay.

**Simple Routing** bắt đầu với một **hosted zone**. Giả sử đây là **public hosted zone** có tên **animalsforlife.org**. Với Simple Routing, bạn có thể tạo **một record cho mỗi tên**. Trong ví dụ này là **www**, thuộc loại **A record**. Mỗi record sử dụng Simple Routing có thể có **nhiều giá trị** nằm trong cùng một record đó.

Khi client thực hiện yêu cầu phân giải **www** và sử dụng Simple Routing, **tất cả các giá trị sẽ được trả về trong cùng một truy vấn**, theo **thứ tự ngẫu nhiên**. Client sẽ chọn một trong các giá trị đó và kết nối đến server tương ứng dựa trên giá trị được chọn — trong trường hợp này là **1.2.3.4**.

**Simple Routing** rất đơn giản, và bạn nên sử dụng nó khi muốn định tuyến yêu cầu đến **một dịch vụ duy nhất**, ví dụ như **web server**. **Hạn chế** của Simple Routing là **không hỗ trợ health checks**. Tôi sẽ giải thích health checks là gì trong video tiếp theo. Nhưng hãy nhớ rằng, với Simple Routing, **không có bất kỳ kiểm tra nào** xem tài nguyên mà record đang trỏ tới có thực sự hoạt động hay không. Điều này rất quan trọng vì **tất cả các loại routing policy khác** trong Route 53 đều cung cấp một số hình thức **health checking** và trí tuệ định tuyến dựa trên những health check đó. Simple Routing là **loại routing policy duy nhất không hỗ trợ health checks**, vì vậy nó khá hạn chế, nhưng lại **dễ triển khai và quản lý**.

Đó là **Simple Routing**. Như tên gọi, nó rất **đơn giản**. Nó không linh hoạt lắm và cũng không có những tính năng thú vị. Nhưng đừng lo, tôi sẽ trình bày về các loại routing nâng cao trong những video sắp tới. Bây giờ, hãy hoàn thành video này, và khi bạn sẵn sàng, tôi rất mong được gặp bạn trong video tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Simple Routing Policy trong Route 53
- Public Hosted Zone (animalsforlife.org)
- Một record mỗi tên (ví dụ: www A record)
- Multiple values trả về ngẫu nhiên
- Dùng cho single service (web server)
- Không hỗ trợ Health Checks
- So sánh với các routing policy khác

**Notes (Chi tiết):**

- Simple Routing là chính sách **mặc định và đơn giản nhất** của Route 53.
- Cho phép tạo **một record duy nhất cho mỗi tên**, nhưng record đó có thể chứa **nhiều giá trị IP** (multiple values).
- Khi DNS query đến, Route 53 trả về **tất cả giá trị theo thứ tự ngẫu nhiên**, client tự chọn một giá trị để kết nối.
- Phù hợp khi muốn route traffic đến **một dịch vụ duy nhất** như web server.
- **Hạn chế lớn**: Không hỗ trợ **health checks**, nghĩa là không kiểm tra xem server có hoạt động hay không. Các routing policy khác đều hỗ trợ health checking và routing thông minh dựa trên kết quả kiểm tra.

**Summary (Tóm tắt ngắn):** Video giới thiệu **Simple Routing** — chính sách định tuyến cơ bản nhất của Route 53. Nó cho phép gán nhiều giá trị IP cho một record và trả về ngẫu nhiên, rất dễ dùng cho dịch vụ đơn giản nhưng **không hỗ trợ health checks**, khác biệt so với các chính sách routing nâng cao khác.

![[Pasted image 20260702111925.png]]