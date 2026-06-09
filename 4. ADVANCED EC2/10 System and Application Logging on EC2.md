**Chào mừng bạn quay trở lại.** Cho đến nay trong khóa học, bạn đã được tiếp xúc ngắn gọn với **CloudWatch** và **CloudWatch Logs**. Bạn đã biết **CloudWatch** giám sát một số khía cạnh hiệu suất và độ tin cậy của EC2, nhưng chỉ những **metrics** có sẵn ở mặt ngoài của instance EC2. Có những tình huống bạn cần bật giám sát **bên trong instance** để có quyền truy cập các **performance counters** của hệ điều hành. Ví dụ: xem các tiến trình đang chạy, mức tiêu thụ bộ nhớ của các tiến trình, hay các chỉ số hiệu suất cấp hệ điều hành mà bạn không thể thấy từ bên ngoài instance. Bạn cũng có thể muốn thu thập **system logs** và **application logs** từ bên trong hệ điều hành của EC2 instance.

Trong bài học này, tôi sẽ hướng dẫn chi tiết cách hoạt động và những gì bạn cần để đạt được điều đó.

Tóm tắt nhanh những gì chúng ta đã học liên quan đến chủ đề này: **CloudWatch** là dịch vụ chịu trách nhiệm lưu trữ và quản lý **metrics** trong AWS. **CloudWatch Logs** là một phần của CloudWatch, chuyên lưu trữ, quản lý và trực quan hóa dữ liệu log. Tuy nhiên, cả hai sản phẩm này **không thể thu thập dữ liệu hoặc log bên trong EC2 instance** một cách tự nhiên. Bên trong instance là khu vực “opaque” (không nhìn thấy) đối với CloudWatch và CloudWatch Logs theo mặc định.

Để cung cấp khả năng nhìn thấy bên trong, chúng ta cần sử dụng **CloudWatch Agent** — một phần mềm chạy **bên trong EC2 instance** (trên hệ điều hành). Agent thu thập dữ liệu có thể nhìn thấy từ OS và gửi vào **CloudWatch** hoặc **CloudWatch Logs**, sau đó bạn có thể sử dụng và trực quan hóa trong console.

Để **CloudWatch Agent** hoạt động, nó cần **cấu hình** và **quyền** để gửi dữ liệu vào hai sản phẩm trên. Tóm lại, để CloudWatch và CloudWatch Logs có quyền truy cập bên trong EC2 instance, bạn cần thực hiện thêm công việc cấu hình và bảo mật, cùng với việc cài đặt **CloudWatch Agent**. Đó chính là nội dung tôi sẽ trình bày trong phần còn lại của bài học này và bài demo sắp tới.

Về mặt **kiến trúc**, **CloudWatch Agent** khá đơn giản. Chúng ta có một EC2 instance (ví dụ: Animals for Life WordPress instance từ các demo trước). Instance này không thể gửi log vào CloudWatch Logs nếu chưa cài agent. Để khắc phục, chúng ta cần **cài đặt CloudWatch Agent** bên trong instance và cung cấp **cấu hình** cho agent (cho agent biết cần thu thập và gửi thông tin gì).

Agent cũng cần quyền tương tác với AWS. Chúng ta biết rằng việc thêm **long-term credentials** (access keys) vào instance là **không tốt**, và cũng khó quản lý ở quy mô lớn. **Best practice** là tạo một **IAM Role** có quyền tương tác với **CloudWatch** và **CloudWatch Logs**, sau đó gắn role này vào EC2 instance. Như vậy, mọi thứ chạy trên instance đều có quyền truy cập vào các dịch vụ này.

Cấu hình của agent sẽ xác định các **metrics** và **logs** cần thu thập. Dữ liệu được gửi vào CloudWatch thông qua **log groups**. Chúng ta sẽ cấu hình **một log group** cho mỗi file log muốn thu thập. Bên trong mỗi log group sẽ có **một log stream** cho mỗi EC2 instance đang gửi log.

Để triển khai cho một instance đơn lẻ, bạn có thể làm thủ công: đăng nhập vào instance, cài agent, cấu hình, gắn role và bắt đầu gửi dữ liệu. Khi triển khai ở quy mô lớn, bạn cần tự động hóa, và có thể sử dụng **CloudFormation** để nhúng cấu hình agent vào mọi instance được provision.

**CloudWatch Agent** hỗ trợ nhiều cách lưu trữ cấu hình. Một trong những cách đó là sử dụng **Parameter Store** để lưu cấu hình của agent dưới dạng parameter. Vì chúng ta vừa học về Parameter Store, nên trong bài demo sắp tới, chúng ta sẽ vừa cài đặt và cấu hình **CloudWatch Agent**, vừa sử dụng **Parameter Store** để lưu cấu hình đó.

Cụ thể, chúng ta sẽ cấu hình agent thu thập log từ **3 file** sau:

- **/var/log/secure** — chứa các sự kiện liên quan đến đăng nhập an toàn vào EC2 instance.
- **Access log** và **Error log** — hai file log được tạo bởi **Apache web server** đang chạy trên EC2 instance.

Việc sử dụng 3 file log này sẽ giúp bạn có kinh nghiệm thực tế tốt về cách cấu hình **CloudWatch Agent** và cách dùng **Parameter Store** để lưu cấu hình ở quy mô lớn.

Đó là phần lý thuyết cho đến nay. Bạn có thể xem hết video này, và khi sẵn sàng, hãy cùng tôi sang bài demo tiếp theo — nơi chúng ta sẽ **cài đặt và cấu hình CloudWatch Agent**.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- CloudWatch & CloudWatch Logs chỉ thấy mặt ngoài EC2
- Cần **CloudWatch Agent** chạy bên trong instance
- Thu thập OS metrics + System/Application logs
- IAM Role (best practice, thay vì long-term credentials)
- Log Groups & Log Streams
- Lưu cấu hình Agent trong **Parameter Store**
- Demo: /var/log/secure + Apache access.log + error.log

**Notes (Chi tiết):**

- **CloudWatch Agent** là phần mềm chạy trên OS của EC2, thu thập dữ liệu bên trong và gửi về CloudWatch / CloudWatch Logs.
- Cần **cấu hình** (xác định metrics/logs cần thu thập) và **IAM Role** có quyền gửi dữ liệu.
- Kiến trúc: 1 **Log Group** cho mỗi file log → bên trong có **Log Stream** cho từng instance.
- Có thể lưu cấu hình Agent dưới dạng parameter trong **Parameter Store** để dễ quản lý ở quy mô lớn.
- Demo sắp tới sẽ cài agent và cấu hình thu thập 3 log file: /var/log/secure, Apache access log và error log.

**Summary (Tóm tắt ngắn):** Bài học giải thích lý do và cách sử dụng **CloudWatch Agent** để thu thập metrics và logs bên trong EC2 instance (vì CloudWatch mặc định chỉ thấy bên ngoài). Nhấn mạnh việc dùng **IAM Role** thay vì credentials, cấu hình qua Log Groups/Streams, và lưu cấu hình Agent trong **Parameter Store**. Bài demo tiếp theo sẽ thực hành cài đặt và cấu hình agent thu thập 3 file log cụ thể.