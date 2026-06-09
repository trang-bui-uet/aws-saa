**Parameter Store** cho phép bạn tạo các **tham số (parameters)**, mỗi tham số có **tên tham số** và **giá trị tham số**. Phần giá trị là nơi lưu trữ cấu hình thực tế. Nhiều dịch vụ AWS tích hợp sẵn với **Parameter Store**. **CloudFormation** cung cấp các tích hợp mà bạn đã sử dụng, tôi sẽ giải thích ngay sau đây và trong các bài demo sắp tới. Bạn cũng có thể sử dụng công cụ CLI trên instance EC2 để truy cập dịch vụ này.

Khi nói về **cấu hình và bí mật (secrets)**, **Parameter Store** cung cấp khả năng lưu trữ **ba loại tham số khác nhau**: **String**, **String List**, và **Secure String**. Với ba loại này, bạn có thể lưu trữ các thông tin như **mã license**, **chuỗi kết nối cơ sở dữ liệu** (hostname, port), thậm chí là toàn bộ cấu hình và mật khẩu.

**Parameter Store** cũng cho phép lưu trữ tham số theo **cấu trúc phân cấp (hierarchical structure)**. Ngoài ra, nó còn lưu trữ **các phiên bản khác nhau của tham số**. Tương tự như **object versioning** trong S3, trong **Parameter Store** bạn cũng có thể có nhiều phiên bản của một tham số.

**Parameter Store** có thể lưu trữ tham số dạng **plain text** (văn bản thuần), phù hợp cho chuỗi kết nối DB hoặc user DB. Đồng thời, nó cũng hỗ trợ **cipher text** (văn bản mã hóa), tích hợp với **KMS** để mã hóa tham số. Điều này rất hữu ích khi lưu **mật khẩu** hoặc thông tin nhạy cảm khác. Khi sử dụng mã hóa cipher text, bạn cần quyền trên **KMS**, tạo thêm một lớp bảo mật.

**Parameter Store** còn có khái niệm **public parameters** (tham số công khai). Đây là các tham số do AWS tạo và công khai. Bạn đã sử dụng chúng trước đây trong khóa học; ví dụ khi dùng **CloudFormation** tạo EC2 instance, bạn không cần chỉ định AMI cụ thể vì đã tham khảo **public parameter** của AWS – đó là **AMI ID mới nhất** cho hệ điều hành nhất định trong vùng (region) đó. Tôi sẽ demo cách hoạt động ngay trong các bài học sắp tới.

**Kiến trúc của Parameter Store** khá đơn giản. Đây là dịch vụ công khai, nên bất kỳ thứ gì sử dụng nó đều cần là dịch vụ AWS hoặc có quyền truy cập đến **public endpoints** của AWS. Nhiều thực thể có thể sử dụng **Parameter Store**, như **ứng dụng**, **EC2 instances**, các tiến trình chạy trên instance, và cả **Lambda functions**. Chúng đều có thể yêu cầu truy cập các tham số trong **Parameter Store**.

Vì **Parameter Store** là dịch vụ AWS, nó tích hợp chặt chẽ với **IAM** để quản lý quyền. Mọi truy cập đều cần **xác thực và ủy quyền**, có thể dùng **long-term credentials** (access keys) hoặc credentials được truyền qua **IAM role**. Nếu tham số được mã hóa, **KMS** sẽ tham gia và cần quyền phù hợp với **CMK** trong KMS.

**Parameter Store** cho phép tạo tập tham số đơn giản hoặc phức tạp. Ví dụ: tham số đơn giản như **/myDBpassword** lưu mật khẩu cơ sở dữ liệu dưới dạng mã hóa. Bạn cũng có thể tạo cấu trúc phân cấp, ví dụ **/wordpress/** bên trong có **dbuser** (có thể truy cập bằng tên đầy đủ hoặc theo nhánh wordpress). Tương tự có **dbpassword**. Bạn có thể lấy toàn bộ nhánh wordpress hoặc truy cập từng tham số riêng lẻ (ví dụ **/wordpress/dbpassword**). Ngoài ra, bạn có thể có nhánh cho ứng dụng khác như **/my-cat-app**, hoặc chia theo chức năng tổ chức (ví dụ nhánh cho đội ngũ dev).

**Quyền truy cập** rất linh hoạt, có thể thiết lập cho từng tham số riêng lẻ hoặc toàn bộ cây phân cấp. Mọi thứ đều hỗ trợ **versioning**. Bất kỳ thay đổi nào đối với tham số cũng có thể sinh ra **events**, kích hoạt các quy trình trong các dịch vụ AWS khác (sẽ giới thiệu sau).

**Parameter Store** không phải là sản phẩm quá phức tạp. Tại thời điểm này, tôi đã trình bày đầy đủ **lý thuyết** cần thiết cho kỳ thi **Associate level**. Bây giờ tôi sẽ kết thúc bài lý thuyết này, và ngay sau là **demo** hướng dẫn tương tác với **Parameter Store** qua giao diện console và công cụ dòng lệnh AWS. Demo sẽ khá ngắn gọn, bạn có thể chỉ xem tôi thực hiện hoặc tự làm theo. Tất cả tài nguyên cần thiết đều có trong thư mục của bài demo trên GitHub repository của khóa học.

Vậy nên, hãy xem hết video này, và khi sẵn sàng, hãy cùng tôi sang bài demo tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Parameter Store: String, String List, Secure String
- Hierarchical structure & Versioning
- Plain text vs Cipher text (tích hợp KMS)
- Public parameters (AMI ID)
- Kiến trúc: IAM, IAM Role, KMS
- Ứng dụng: EC2, Lambda, Apps
- Demo sắp tới: Console + CLI

**Notes (Chi tiết):**

- **Parameter Store** lưu cấu hình và secrets với 3 loại: **String**, **String List**, **Secure String** (mã hóa qua KMS).
- Hỗ trợ **cấu trúc phân cấp** (/wordpress/dbpassword) và **versioning** giống S3.
- **Public parameters** do AWS cung cấp (ví dụ AMI ID mới nhất theo region).
- Truy cập qua **IAM** (access keys hoặc IAM Role), cần quyền KMS nếu mã hóa.
- Có thể trigger **events** khi tham số thay đổi.
- Dùng cho EC2 User Data, CloudFormation, Lambda, ứng dụng chạy trên instance.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu **AWS Parameter Store** – dịch vụ lưu trữ cấu hình và secrets an toàn với hỗ trợ phân cấp, versioning, mã hóa KMS và public parameters. Tích hợp chặt chẽ với IAM, CloudFormation, EC2 và các dịch vụ khác. Lý thuyết đầy đủ cho kỳ thi Associate, tiếp theo là demo thực hành qua Console và CLI.