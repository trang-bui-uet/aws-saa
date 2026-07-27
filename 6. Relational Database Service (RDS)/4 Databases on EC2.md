Chào mừng trở lại! Trong bài học này, chúng ta sẽ nói về một chủ đề có thể được coi là **thực tiễn xấu** trong AWS — đó là **chạy cơ sở dữ liệu trực tiếp trên EC2**. Như bạn sẽ thấy trong phần này của khóa học, AWS có rất nhiều sản phẩm cung cấp dịch vụ cơ sở dữ liệu được quản lý. Vì vậy, việc chạy bất kỳ cơ sở dữ liệu nào trên EC2, tốt nhất là cần có lý do chính đáng.

Tôi sẽ đi qua **tại sao bạn có thể muốn chạy database trên EC2** và **tại sao đó thực sự là một ý tưởng xấu**. Thực tế thì **luôn luôn là ý tưởng xấu** khi chạy database trên EC2. Vấn đề thực sự chỉ là: liệu lợi ích mang lại cho bạn hoặc doanh nghiệp có lớn hơn những nhược điểm nghiêm trọng hay không?

### Kiến trúc khi chạy cơ sở dữ liệu trên EC2

Khi mọi người nghĩ đến việc chạy database trên EC2, họ thường hình dung một trong hai mô hình:

- **Mô hình 1 (đang dùng hiện tại)**: Một **instance duy nhất** chứa toàn bộ stack — database platform + ứng dụng + web server (ví dụ Apache). Đây chính là cách chúng ta đang chạy WordPress Animals for Life cho đến nay: **một EC2 instance duy nhất** trong **một Availability Zone duy nhất**.
- **Mô hình 2 (kiến trúc tách biệt)**: Tách database ra instance riêng. Bạn sẽ có **hai EC2 instance** — một chạy application/web server, một chạy database server. Hai instance này có thể nằm cùng một AZ hoặc tách ra hai AZ khác nhau (AZ-A và AZ-B).

Khi bạn tách thành hai instance, bạn đã **giới thiệu một sự phụ thuộc mới** vào kiến trúc: cần có **giao tiếp đáng tin cậy** giữa application instance và database instance. Nếu không, ứng dụng sẽ không hoạt động.

Ngoài ra, nếu đặt hai instance ở hai AZ khác nhau, bạn sẽ phải chịu **chi phí truyền dữ liệu** giữa các AZ (dù nhỏ). Ngược lại, giao tiếp giữa các instance dùng private IP trong **cùng một AZ là miễn phí**.

### Lý do bạn có thể muốn chạy database trên EC2

Dưới đây là những lý do thường được đưa ra (một số cần được **đặt câu hỏi nghiêm túc**):

- **Cần truy cập cấp độ hệ điều hành (OS-level access)**: Chỉ EC2 mới cho phép bạn truy cập OS. Các sản phẩm managed database của AWS không cung cấp quyền này. Tuy nhiên, bạn nên **hỏi lại khách hàng/doanh nghiệp**: Họ có thực sự cần không, hay chỉ nghĩ là cần? Rất ít trường hợp thực sự yêu cầu quyền OS.
- **Cần tuning database ở mức root**: Một số tham số cấu hình chỉ thay đổi được với quyền root. Nhưng AWS đã cho phép bạn kiểm soát rất nhiều tham số quan trọng mà **không cần quyền root**. Nên xác minh kỹ trước khi chấp nhận lý do này.
- **Cần chạy database hoặc phiên bản database mà AWS không cung cấp**: Đây là lý do **thường được chấp nhận**. Đặc biệt với các loại database mới nổi, database niche, hoặc khi bạn cần một **kết hợp rất cụ thể** giữa phiên bản OS + phiên bản DB mà managed services không hỗ trợ.
- **Cần kiến trúc replication đặc biệt**: Một số cách sao chép (replication) hoặc timing mà AWS managed services chưa hỗ trợ.
- **Doanh nghiệp / người ra quyết định muốn dùng EC2**: Đôi khi họ chỉ đơn giản muốn database chạy trên EC2. Dù có thể không hợp lý, nhưng trong thực tế bạn đôi khi không có lựa chọn.

**Lưu ý quan trọng**: Nhiều yêu cầu về quyền root hay OS-level access thường đến từ **nhà cung cấp ứng dụng**, chứ không phải từ doanh nghiệp. Hãy luôn xác minh và yêu cầu biện minh rõ ràng.

### Tại sao bạn THỰC SỰ KHÔNG NÊN chạy database trên EC2

Dù có những lý do trên, bạn cần nhận thức rõ những **nhược điểm nghiêm trọng** sau:

- **Gánh nặng quản trị (Admin overhead) rất lớn**: Bạn phải quản lý cả EC2 instance lẫn database server. Công việc vá lỗi (patching), nâng cấp phiên bản, đảm bảo tương thích với ứng dụng… tất cả đều tốn rất nhiều effort. Nâng cấp thường phải làm ngoài giờ hành chính → gây stress và chi phí nhân sự cao.
- **Backup và Disaster Recovery (DR) phức tạp**: Nếu doanh nghiệp có kế hoạch DR, việc chạy database trên EC2 sẽ tăng độ phức tạp rất nhiều. Ngược lại, các sản phẩm managed database của AWS có rất nhiều tính năng tự động hóa giúp giảm bớt gánh nặng này.
- **Vấn đề nghiêm trọng nhất: Single Availability Zone**: **EC2 + EBS volume đều nằm trong một AZ duy nhất**. Nếu AZ đó gặp sự cố, toàn bộ database sẽ không truy cập được. Bạn phải tự lo chụp EBS snapshot hoặc backup database lên S3. Đây là rủi ro và gánh nặng quản trị lớn mà doanh nghiệp cần hiểu rõ.
- **Bỏ lỡ các tính năng nâng cao**: AWS đã đầu tư rất nhiều thời gian, công sức và tiền bạc để phát triển các sản phẩm database managed với nhiều tính năng vượt trội so với việc bạn tự cài database trên EC2. Bạn sẽ **không thể sử dụng** những tính năng này.
- **Không có khả năng serverless scaling**: EC2 là “bật/tắt”. Bạn không thể scale down dễ dàng hay theo kịp nhu cầu bursty. Nhiều managed database service có thể scale up/down nhanh chóng theo tải. Khi dùng EC2, bạn bị ràng buộc với **chi phí tối thiểu cố định** theo kích thước instance.
- **Replication tự setup tốn kém**: Nếu cần replication, bạn phải tự thiết lập, giám sát và kiểm tra hiệu quả. Tất cả những công việc này thường đã được managed services xử lý sẵn.
- **Hiệu suất thấp hơn**: AWS đã tối ưu hóa rất mạnh các sản phẩm database của họ. Khi bạn chỉ cài database off-the-shelf lên EC2, bạn sẽ **không tận dụng được** các tính năng tối ưu hiệu suất mà AWS cung cấp.

### Kết luận của bài học

**Chạy database trên EC2 luôn là ý tưởng xấu về mặt kỹ thuật**. Chỉ nên làm khi lợi ích thực sự vượt trội và đã được biện minh rõ ràng. Trong hầu hết các trường hợp, bạn nên **đặt câu hỏi và yêu cầu giải thích** trước khi chấp nhận giải pháp này.

Trong **bài học tiếp theo (demo)**, chúng ta sẽ lấy stack WordPress single-instance hiện tại và **phát triển thành hai EC2 instance riêng biệt**:

- Một instance chạy Apache + WordPress (Application server)
- Một instance chạy MariaDB (Database server)

Đây là bước **best practice** (trong giới hạn của self-managed database) để tách monolithic stack, giúp sau này dễ dàng di chuyển database sang các sản phẩm managed database của AWS.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Chạy database trên EC2: Thực tiễn xấu nhưng đôi khi cần thiết
- Hai kiến trúc: Single instance (monolithic) vs Split (App + DB riêng biệt)
- Lý do chính đáng: OS-level access, root tuning, DB/version không hỗ trợ bởi AWS, yêu cầu dự án cụ thể, quyết định business
- Nhược điểm chính: Admin overhead cao, Backup/DR phức tạp, **Single AZ** rủi ro, thiếu advanced features, không serverless scaling, replication effort, performance kém
- Bước tiếp theo: Demo tách WordPress thành 2 EC2 instances (Apache/WordPress + MariaDB)

**Notes (Chi tiết):**

- **Lý do nên cân nhắc**:
    - Cần truy cập OS (nhưng **hiếm khi thực sự cần** — nên đặt câu hỏi)
    - Tuning DB (hay tối ưu hóa cơ sở dữ liệu) yêu cầu root access (AWS managed cho phép kiểm soát nhiều tham số mà không cần root)
    - Cần **DB hoặc version cụ thể** không có trên managed services
    - Yêu cầu kết hợp OS + DB version rất cụ thể
    - Kiến trúc replication đặc biệt không được AWS cung cấp
    - Business yêu cầu (dù không hợp lý)
- **Lý do KHÔNG NÊN** (rất quan trọng):
    - **Gánh nặng quản trị lớn**: Patch, upgrade, compatibility giữa EC2 và DB, làm ngoài giờ → stress + chi phí nhân sự cao
    - **Backup & Disaster Recovery** phức tạp hơn nhiều so với managed services (có automation sẵn)
    - **Single Availability Zone**: Toàn bộ DB nằm trong 1 AZ → rủi ro cao nếu AZ fail. Phải tự quản lý EBS snapshot / DB backup lên S3
    - Bỏ lỡ **nhiều tính năng nâng cao** mà AWS đã đầu tư mạnh vào managed DB products
    - **Không có serverless**: Không scale dễ dàng theo bursty demand, chi phí tối thiểu cố định theo instance type
    - Tự setup **replication** tốn kỹ năng, thời gian, monitoring
    - **Hiệu suất thấp hơn**: Thiếu các tối ưu hóa và tính năng performance mà AWS cung cấp sẵn
- **Kết luận & Hướng đi tiếp theo**:
    - Chạy DB trên EC2 **luôn là ý tưởng xấu** về mặt kỹ thuật, chỉ làm khi lợi ích vượt trội rõ rệt.
    - Demo tiếp theo: Tách monolithic WordPress stack thành **2 EC2 instances riêng biệt** (Application server + Database server MariaDB). Đây là bước chuẩn bị để sau này migrate DB sang managed service.

**Summary (Tóm tắt ngắn):** Bài học phân tích chi tiết **lý do nên và không nên chạy database trực tiếp trên EC2**. Mặc dù có một số trường hợp chính đáng (DB đặc biệt, yêu cầu access OS, quyết định business), nhưng nhược điểm về quản trị, availability (Single AZ), tính năng và hiệu suất là rất lớn. Khuyến khích **đặt câu hỏi và yêu cầu biện minh** trước khi chọn giải pháp này. Kết thúc bằng việc giới thiệu demo tách app và DB để tiến tới sử dụng AWS managed database services.