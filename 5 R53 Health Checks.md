**Chào mừng trở lại.** Trong video này, tôi muốn trình bày về **tính năng Health Check** trong **Route 53**. Health checks hỗ trợ nhiều kiến trúc nâng cao của Route 53, vì vậy việc hiểu cách chúng hoạt động là rất cần thiết với tư cách là kiến trúc sư, nhà phát triển hoặc kỹ sư. Hãy cùng bắt đầu ngay nhé.

Đầu tiên, hãy nhanh chóng xem qua một số khái niệm cấp cao về health checks. **Health checks** tách biệt nhưng được sử dụng bởi các **records** bên trong Route 53. Bạn không tạo health check bên trong record; chúng tồn tại độc lập. Bạn cấu hình chúng riêng biệt. Chúng đánh giá tình trạng sức khỏe của một thứ gì đó và có thể được sử dụng bởi các record trong Route 53.

Health checks được thực hiện bởi một đội ngũ **health checkers** được phân phối toàn cầu. Điều này có nghĩa là nếu bạn kiểm tra sức khỏe của các hệ thống được lưu trữ trên internet công cộng, bạn cần cho phép các kiểm tra này đến từ health checkers. Nếu bạn coi chúng là bot hoặc cố gắng khai thác và chặn chúng, điều đó sẽ gây ra cảnh báo giả.

Như tôi vừa đề cập, health checks không chỉ giới hạn ở các mục tiêu AWS. Bạn có thể kiểm tra bất kỳ thứ gì có thể truy cập qua internet công cộng. Chỉ cần một địa chỉ IP là đủ.

Các kiểm tra diễn ra **mỗi 30 giây** theo mặc định, hoặc có thể tăng lên **mỗi 10 giây** với chi phí bổ sung. Các kiểm tra có thể là **TCP checks** — Route 53 cố gắng thiết lập kết nối TCP với endpoint và cần thành công trong vòng 10 giây. Bạn có thể có **HTTP checks** — Route 53 phải thiết lập kết nối TCP với endpoint trong vòng 4 giây, và endpoint phải trả về mã trạng thái HTTP trong khoảng 200 hoặc 300 trong vòng 2 giây sau khi kết nối. Loại này chính xác hơn cho các ứng dụng web so với TCP check đơn giản.

Cuối cùng, với các kiểm tra **HTTP và HTTPS**, bạn cũng có thể thực hiện **string matching**. Route 53 phải thiết lập kết nối TCP trong vòng 4 giây, endpoint phải trả về mã trạng thái 200 hoặc 300 trong vòng 2 giây, và health checker phải nhận được phần thân phản hồi trong vòng 2 giây tiếp theo. Route 53 tìm kiếm chuỗi bạn chỉ định trong phần thân phản hồi. Chuỗi phải xuất hiện hoàn toàn trong **5.120 byte đầu tiên** của phần thân phản hồi, nếu không endpoint sẽ thất bại health check. Đây là cách chính xác nhất vì không chỉ kiểm tra ứng dụng có phản hồi qua HTTP/HTTPS mà còn kiểm tra nội dung phản hồi so với những gì ứng dụng nên làm trong điều kiện bình thường.

Dựa trên các health checks này, một endpoint sẽ ở trạng thái **healthy** hoặc **unhealthy**. Nó chuyển đổi giữa hai trạng thái này dựa trên kết quả kiểm tra.

Cuối cùng, chính các health checks có thể thuộc một trong ba loại:

- **Endpoint checks**: Kiểm tra sức khỏe của một endpoint thực tế mà bạn chỉ định.
- **CloudWatch alarm checks**: Phản ứng với các CloudWatch alarms, có thể được cấu hình riêng và có thể bao gồm các kiểm tra chi tiết trong hệ điều hành hoặc ứng dụng nếu bạn sử dụng CloudWatch agent.
- **Calculated checks**: Kiểm tra của các health checks khác. Bạn có thể tạo health check để kiểm tra sức khỏe toàn bộ ứng dụng với nhiều thành phần riêng lẻ.

Bạn sẽ có cơ hội thực hiện health check trong bài học demo sắp tới trong phần này của khóa học. Nhưng trước đó, tôi muốn đưa ra tổng quan về giao diện console khi tạo health check. Hãy chuyển sang console.

Chúng ta đang ở AWS Console, đăng nhập vào tài khoản chung tại vùng Northern Virginia. Để tạo health check, cần chuyển đến **Route 53 console**. Tôi sẽ thực hiện điều đó ngay. Như đã đề cập trong phần lý thuyết, health checks được tạo độc lập với records. Thay vì vào hosted zone, chọn record rồi cấu hình health check, bạn vào menu bên trái và nhấp **Health checks**. Sau đó nhấp **Create health check** để nhập thông tin cần thiết.

Đầu tiên, bạn cần đặt **tên** cho health check. Ví dụ: “test health check”. Có ba loại health check khác nhau:

- **Endpoint health check**: Kiểm tra sức khỏe của một endpoint cụ thể.
- **Status of other health checks**: Đây là **calculated health check**, cho phép bạn tạo health check giám sát toàn bộ ứng dụng dựa trên trạng thái sức khỏe của các thành phần riêng lẻ.
- **Status of a CloudWatch alarm**: Sử dụng trạng thái của CloudWatch alarm làm cơ sở.

Nếu chọn **endpoint**, bạn có thể chọn **IP address** hoặc **domain name**. Nếu chọn domain name, tất cả health checkers của Route 53 sẽ phân giải domain name trước rồi thực hiện kiểm tra trên địa chỉ IP thu được.

Trong cả hai trường hợp, bạn có thể chọn **TCP** (chỉ cần chỉ định IPv4 hoặc IPv6 cùng port). Hoặc chọn **HTTP/HTTPS** (cần chỉ định IP, port, domain name làm host header nếu có nhiều virtual host, và path). HTTPS sẽ sử dụng kết nối bảo mật.

Cuộn xuống và mở rộng **Advanced configuration**:

- **Request interval**: Mặc định 30 giây, hoặc chọn “fast” để kiểm tra mỗi 10 giây (từ mọi health checker).
- **Failure threshold**: Số lần kiểm tra liên tiếp phải pass hoặc fail trước khi Route 53 thay đổi trạng thái.
- **String matching**: Kiểm tra nâng cao hơn bằng cách tìm chuỗi cụ thể trong response body.
- Các tùy chọn nâng cao khác: **Latency graphs**, **Invert health check status**, **Disable health check** (hữu ích khi bảo trì tạm thời), và **Health checker regions** (khuyến nghị hoặc tùy chỉnh).

Sau khi nhập giá trị mẫu (ví dụ: IP 1.1.1.1, port 80), bạn có thể cấu hình thông báo khi health check fail (tùy chọn). Bạn có thể tạo alarm và gửi đến SNS topic để tích hợp với hệ thống khác.

Đây là tổng quan giao diện tạo health check trong console. Bạn sẽ thực hành trong bài demo sắp tới.

Bây giờ quay lại phần kiến trúc. Giả sử bạn có ứng dụng **Catagram** gần UK và trỏ record Route 53 (catagram.io) đến nó. Bạn có thể liên kết health check với record này. Các health checkers toàn cầu sẽ kiểm tra định kỳ. Nếu **hơn 18%** health checkers báo healthy thì health check tổng thể là healthy; ngược lại là unhealthy. Hầu hết các record unhealthy sẽ không được trả về khi query DNS.

Bạn sẽ thấy trong suốt phần này và khóa học cách health checks ảnh hưởng đến cách DNS phản hồi query và cách ứng dụng phản ứng khi thành phần gặp sự cố. **Route 53** là công cụ thiết kế và vận hành thiết yếu để kiểm soát resolution requests và routing đến các thành phần ứng dụng. Hiểu health checks là cần thiết để thiết kế hạ tầng Route 53, tích hợp với ứng dụng và vận hành hàng ngày.

Đó là tất cả những gì tôi muốn trình bày trong video này. Hãy hoàn thành video và khi sẵn sàng, tôi mong được gặp bạn trong bài học tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Health checks tách biệt với records trong Route 53
- Được thực hiện bởi fleet health checkers phân phối toàn cầu
- Ba loại: Endpoint check, CloudWatch alarm check, Calculated check
- Tần suất kiểm tra: 30 giây (mặc định) hoặc 10 giây (fast)
- Các loại kiểm tra: TCP, HTTP/HTTPS, HTTP/HTTPS + String matching (5120 bytes đầu)
- Tạo trong Console: Tên, loại, IP/domain, port, path, advanced config (failure threshold, invert, disable, regions)
- Kiến trúc: Kiểm tra từ nhiều region → ngưỡng >18% healthy mới coi là healthy
- Ứng dụng: Ảnh hưởng routing DNS, failover, và giám sát ứng dụng

**Notes (Chi tiết):**

- **Health checks** tồn tại độc lập, không tạo bên trong record. Chúng đánh giá tình trạng endpoint (có thể là bất kỳ IP nào trên internet công cộng).
- **Health checkers** toàn cầu thực hiện kiểm tra. Phải cho phép traffic từ chúng, nếu không sẽ gây false positive.
- **TCP check**: Thiết lập kết nối TCP thành công trong 10 giây.
- **HTTP/HTTPS check**: Kết nối TCP trong 4 giây + mã trạng thái 200/300 trong 2 giây.
- **String matching** (chính xác nhất): Nhận response body trong 2 giây và tìm chuỗi chỉ định trong 5120 byte đầu tiên.
- **Failure threshold**: Số lần fail liên tiếp trước khi chuyển trạng thái (để tránh nhiễu).
- **Calculated health check**: Kiểm tra sức khỏe tổng thể ứng dụng dựa trên nhiều health check thành phần.
- **CloudWatch alarm check**: Kết hợp kiểm tra sâu trong OS/app qua CloudWatch agent.
- Khi tạo trong console: Chọn endpoint → chỉ định IP/domain + protocol + port + path (và host header nếu cần). Advanced: latency graph, invert status, disable tạm thời, chọn region health checker.
- **Kiến trúc quan trọng**: Record liên kết health check → nếu <18% health checkers healthy thì record bị coi là unhealthy và không được trả về DNS query.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu toàn diện về **Health Check trong Route 53** — một tính năng độc lập được sử dụng bởi records để giám sát sức khỏe endpoint. Có ba loại chính (Endpoint, CloudWatch Alarm, Calculated). Kiểm tra được thực hiện bởi fleet toàn cầu với tần suất 30s/10s, hỗ trợ TCP, HTTP/HTTPS và string matching nâng cao. Việc tạo trong console cho phép cấu hình chi tiết (interval, threshold, regions, invert…). Về mặt kiến trúc, health check giúp Route 53 quyết định có trả về record hay không (dựa trên ngưỡng >18% healthy), từ đó hỗ trợ các mô hình failover, routing thông minh và giám sát ứng dụng phức tạp. Hiểu rõ health check là nền tảng để thiết kế và vận hành hệ thống Route 53 hiệu quả.