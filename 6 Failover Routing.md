**Chào mừng trở lại**, và trong video này, tôi muốn nói về **chính sách định tuyến Route 53 thứ hai** mà tôi sẽ đề cập trong loạt video này, đó là **định tuyến failover**. Bây giờ hãy cùng bắt đầu ngay nhé.

Với **định tuyến failover**, chúng ta bắt đầu với một **hosted zone**, và bên trong hosted zone này là một bản ghi **www**. Tuy nhiên, với định tuyến failover, chúng ta có thể thêm **nhiều bản ghi cùng tên**: một bản ghi **primary** (chính) và một bản ghi **secondary** (phụ). Mỗi bản ghi này trỏ đến một tài nguyên, và một ví dụ phổ biến là **kiến trúc failover out-of-band** — nơi bạn có điểm cuối ứng dụng chính (primary application endpoint), chẳng hạn như một **EC2 instance**, và một tài nguyên dự phòng (backup/failover resource) sử dụng dịch vụ khác, chẳng hạn như **S3 bucket**.

**Yếu tố then chốt** của định tuyến failover là việc bao gồm **health check**. Health check thường được thực hiện trên bản ghi **primary**. Nếu bản ghi primary khỏe mạnh, thì mọi truy vấn đến www (trong ví dụ này) sẽ được phân giải đến giá trị của bản ghi primary — tức là **EC2 instance đang chạy Catagram**. Nếu bản ghi primary **thất bại health check**, thì giá trị của bản ghi **secondary** cùng tên sẽ được trả về — trong trường hợp này là **S3 bucket**.

**Mục đích sử dụng** của định tuyến failover rất đơn giản: Sử dụng nó khi bạn cần cấu hình **active-passive failover**, nghĩa là bạn muốn định tuyến lưu lượng đến một tài nguyên khi tài nguyên đó khỏe mạnh, hoặc chuyển sang một tài nguyên khác khi tài nguyên gốc đang thất bại health check.

Đây là một khái niệm khá đơn giản mà bạn sẽ tự trải nghiệm trong **video demo sắp tới**. Tuy nhiên, tại thời điểm này, đó là tất cả những gì tôi muốn đề cập trong video này. Vì vậy, hãy hoàn thành video, và khi bạn sẵn sàng, tôi mong chờ được gặp bạn trong video tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- **Failover routing** policy trong Route 53
- **Primary record** và **Secondary record** cùng tên (www)
- **Health check** (thường áp dụng trên primary)
- **Active-passive failover** architecture
- Ví dụ thực tế: Primary = EC2 instance (Catagram), Secondary = S3 bucket
- **Out-of-band failure** architecture
- Chuyển hướng tự động khi primary fail health check

**Notes (Chi tiết):**

- **Hosted zone** chứa bản ghi www; với failover routing có thể tạo **nhiều record cùng tên** (primary + secondary) thay vì chỉ một record.
- **Health check** là yếu tố quyết định: Nếu primary **healthy** → Route 53 trả về giá trị primary (EC2). Nếu primary **fail health check** → Tự động trả về giá trị secondary (S3 bucket).
- **Use case chính**: Cấu hình **active-passive failover** — định tuyến lưu lượng đến tài nguyên khỏe mạnh, hoặc chuyển sang tài nguyên dự phòng khi tài nguyên chính gặp sự cố.
- Đây là chính sách định tuyến đơn giản nhưng rất mạnh mẽ để tăng tính sẵn sàng cao (high availability) mà không cần cấu hình phức tạp ở phía ứng dụng.
- Video nhấn mạnh đây là khái niệm cơ bản, sẽ được thực hành ngay trong **video demo tiếp theo**.

**Summary (Tóm tắt ngắn):** Video giới thiệu **chính sách định tuyến Failover** của Amazon Route 53, cho phép cấu hình **active-passive failover** bằng cách sử dụng **primary** và **secondary records** cùng tên, kết hợp **health check** để tự động chuyển hướng lưu lượng truy vấn khi primary resource gặp sự cố. Ví dụ minh họa rõ ràng với EC2 instance làm primary và S3 bucket làm secondary, rất phù hợp cho các kiến trúc cần dự phòng ngoài băng tần (out-of-band).