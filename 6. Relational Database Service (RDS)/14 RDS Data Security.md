Chào mừng trở lại. Trong bài học này, chúng ta sẽ nói về **bảo mật dữ liệu** trong sản phẩm RDS. Tôi sẽ tập trung vào **bốn khía cạnh chính**:

- **Authentication**: Cách người dùng đăng nhập vào RDS
- **Authorization**: Cách kiểm soát quyền truy cập
- **Encryption in Transit**: Mã hóa dữ liệu khi truyền giữa client và RDS
- **Encryption at Rest**: Mã hóa dữ liệu khi lưu trên đĩa

Chúng ta có khá nhiều nội dung cần cover, hãy bắt đầu ngay.

### Encryption in Transit & Encryption at Rest

Với **tất cả các engine** trong RDS, bạn đều có thể sử dụng **encryption in transit**. Điều này có nghĩa là dữ liệu giữa client và RDS instance được mã hóa bằng **SSL hoặc TLS**. Bạn thậm chí có thể thiết lập bắt buộc (mandatory) trên từng user.

**Encryption at rest** được hỗ trợ theo một số cách khác nhau tùy thuộc vào database engine:

- **KMS & EBS Encryption (mặc định)**: Sử dụng **AWS KMS** và **EBS encryption**. Quá trình này được xử lý bởi **RDS host** và storage EBS phía dưới. Đối với database engine, nó chỉ thấy mình đang ghi dữ liệu chưa mã hóa; việc mã hóa thực tế do host đảm nhiệm.
- **CMKs & DEKs**: Bạn chọn một **Customer Master Key (CMK)** — có thể là customer-managed hoặc AWS-managed. CMK này được dùng để tạo ra các **Data Encryption Keys (DEKs)**. DEKs mới là thứ thực hiện việc mã hóa/giải mã dữ liệu.
- **Phạm vi mã hóa**: Khi bật loại mã hóa này, **storage, logs, snapshots và mọi replicas** đều được mã hóa bằng cùng một CMK. **Quan trọng**: Một khi đã bật encryption, bạn **không thể tắt** nó đi.

### Các tùy chọn mã hóa riêng theo Engine

Ngoài KMS và EBS, **Microsoft SQL Server** và **Oracle** còn hỗ trợ **TDE (Transparent Data Encryption)**.

- **Transparent Data Encryption (TDE)**: Đây là mã hóa được hỗ trợ và xử lý **trực tiếp bên trong database engine**. Dữ liệu được mã hóa/giải mã ngay trong engine, không phụ thuộc vào host → mức độ tin cậy cao hơn vì dữ liệu đã được bảo vệ ngay từ lúc engine ghi xuống đĩa.
- **CloudHSM Support**: RDS Oracle hỗ trợ TDE sử dụng **AWS CloudHSM**. Với kiến trúc này, quá trình mã hóa còn an toàn hơn vì bạn tự quản lý key, **AWS không hề tiếp xúc với key**. Điều này cực kỳ quan trọng trong các môi trường có yêu cầu tuân thủ nghiêm ngặt (không có trust chain với AWS).

### Tổng quan kiến trúc mã hóa

Hình dung như sau:

1. Trong một VPC có một số RDS instance chạy trên các host, sử dụng EBS làm storage.
2. **Oracle với TDE + CloudHSM** (bên trái): CloudHSM cung cấp dịch vụ key. Vì TDE được xử lý ngay trong engine, dữ liệu được mã hóa từ engine đến tận storage. AWS không tiếp xúc với key bên ngoài instance.
3. **KMS-Based Encryption**: KMS tạo CMK → sinh ra DEKs. Các DEK này được load lên RDS host khi cần và được host dùng để mã hóa/giải mã. Database engine hoàn toàn không biết đến việc mã hóa — nó ghi dữ liệu bình thường, host mới mã hóa trước khi gửi xuống EBS.
4. **Replicas & Snapshots**: Dữ liệu truyền giữa các replicas (ví dụ MySQL) cũng được mã hóa, và mọi snapshot của volume EBS đều dùng chung key mã hóa.

![[Pasted image 20260729011212.png]]

### IAM Authentication cho RDS

Thông thường, việc đăng nhập vào RDS được kiểm soát bằng **local database users** (username + password). Những user này **không phải IAM user** và nằm ngoài sự kiểm soát của AWS (chỉ có một user được tạo khi provision RDS).

Tuy nhiên, bạn có thể cấu hình RDS để cho phép **IAM user authentication**:

1. Tạo một **local database user** trên RDS và cấu hình nó cho phép xác thực bằng **AWS authentication token**.
2. Gắn **IAM policy** cho IAM user hoặc role (ví dụ instance role). Policy này chứa mapping giữa IAM entity và local database user.
3. Identity chạy lệnh generate-db-auth-token → tạo ra một token có hiệu lực **15 phút**.
4. Dùng token này để đăng nhập vào database **mà không cần password**.

**Mẹo quan trọng cho kỳ thi**: IAM authentication **chỉ xử lý authentication**, **không xử lý authorization**. Quyền hạn bên trong database vẫn do **local database user** quyết định. IAM chỉ tham gia vào bước xác thực nếu bạn bật tính năng này trên RDS instance.

![[Pasted image 20260729011327.png]]

Đó là toàn bộ nội dung về encryption in transit, encryption at rest và IAM-based authentication trên RDS. Cảm ơn bạn đã theo dõi! Hãy hoàn thành video này và khi sẵn sàng, chúng ta sẽ gặp nhau ở bài học tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Encryption in Transit (SSL/TLS)
- Encryption at Rest (KMS + EBS)
- CMK → DEK
- TDE (SQL Server & Oracle)
- CloudHSM với Oracle
- IAM Authentication (token 15 phút)
- Authentication ≠ Authorization

**Notes (Chi tiết):**

- **Encryption in Transit**: Hỗ trợ SSL/TLS, có thể bắt buộc theo từng user.
- **Encryption at Rest mặc định**: Dùng KMS + EBS. Host mã hóa, engine không biết.
    - CMK tạo DEKs để thực hiện mã hóa.
    - Mã hóa áp dụng cho storage, logs, snapshots, replicas.
    - **Không thể tắt** sau khi đã bật.
- **TDE**: Mã hóa ngay trong engine (SQL Server & Oracle) → tin cậy cao hơn.
- **Oracle + CloudHSM**: Key do bạn quản lý, AWS không tiếp xúc key → phù hợp yêu cầu tuân thủ cao.
- **IAM Authentication**:
    - Tạo local DB user cho phép dùng token.
    - IAM policy mapping IAM entity ↔ local user.
    - Token có hiệu lực 15 phút.
    - **Chỉ authentication**, authorization vẫn do local DB user kiểm soát.

**Summary (Tóm tắt ngắn):** Bài học trình bày bốn trụ cột bảo mật dữ liệu trên RDS: Authentication, Authorization, Encryption in Transit và Encryption at Rest. Encryption at rest mặc định dùng KMS + EBS (host mã hóa), trong khi SQL Server/Oracle hỗ trợ TDE (engine mã hóa). Oracle còn dùng được CloudHSM để loại bỏ trust với AWS. IAM Authentication cho phép đăng nhập bằng token 15 phút nhưng chỉ xử lý authentication, không thay thế authorization của database.