**Chào mừng bạn quay trở lại.** Trong bài demo này, tôi muốn cung cấp cho bạn một số kinh nghiệm thực tế khi tương tác với **Parameter Store** trong AWS. Để thực hiện, hãy đảm bảo bạn đã đăng nhập bằng **IAM admin user** của **management account** trong Organization và chọn vùng **Northern Virginia (us-east-1)**. Ngoài ra, có tài liệu **lesson commands** được đính kèm bài học, chứa tất cả các lệnh cần dùng cho demo lần này.

Trước khi tương tác với **Parameter Store** từ dòng lệnh, chúng ta cần tạo một số tham số. Cách thực hiện là chuyển đến **Systems Manager**. **Parameter Store** thực chất là một sub-product của **Systems Manager**. Hãy chuyển sang console **Systems Manager**, sau đó chọn **Parameter Store** từ menu bên trái (khoảng giữa, nằm dưới **Application Management**).

Khi vào **Parameter Store**, để bỏ màn hình chào mừng mặc định, bạn cần **tạo tham số** đầu tiên bằng cách click **Create parameter**.

Khi tạo tham số, bạn có thể chọn **Standard** hoặc **Advanced**. **Standard** là mặc định và đáp ứng hầu hết nhu cầu. Bạn có thể tạo tối đa **10.000 tham số** với tier Standard. Tier **Advanced** cho phép tạo nhiều tham số hơn, giá trị dài hơn (**8 KB** so với **4 KB**), và có thêm một số tính năng. Tuy nhiên, hầu hết trường hợp dùng **Standard** là đủ. Với **Standard tier**, không mất thêm chi phí đến giới hạn 10.000 tham số. Chỉ khi dùng tùy chọn throughput cao hơn hoặc tier Advanced mới phát sinh chi phí. Trong khóa học chúng ta chỉ dùng **Standard**, nên sẽ không có chi phí thêm liên quan đến Parameter Store.

Một tham số gồm **tên tham số** và **giá trị tham số**, đồng thời có thể thêm **mô tả tùy chọn** và chọn **kiểu tham số**: **String**, **StringList** (danh sách chuỗi cách nhau bởi dấu phẩy), hoặc **SecureString** (sử dụng mã hóa).

Chúng ta sẽ tạo một số tham số để sau này tương tác qua dòng lệnh.

Tham số đầu tiên: **/my-cat-app/dbstring**. Tên này cũng thiết lập **cấu trúc phân cấp** (hierarchy) nhờ dấu gạch chéo (/). Hãy tưởng tượng đây là cấu trúc thư mục: root → my-cat-app → dbstring. Giữ kiểu **String**, giá trị là chuỗi kết nối DB cho my-cat-app: **db.allthecats.com:3306** (3306 là port mặc định của MySQL). Thêm mô tả: **"Connection string for cat application"**. Sau đó click **Create parameter**.

Tiếp theo, tạo **/my-cat-app/dbuser**. Giữ Standard, kiểu String, giá trị: **bosscat**. Không cần mô tả. Click **Create parameter**.

Tiếp tục tạo **/my-cat-app/dbpassword**. Lần này chọn kiểu **SecureString** để mã hóa. Sử dụng **KMS** để mã hóa, chọn key mặc định **alias/aws/ssm**. Giá trị: **amazingsiscretpassword1337**. Click **Create parameter**.

Tạo thêm tham số **/my-dog-app/dbstring**: Standard, String, giá trị **db.ifwereallymusthavedogs.com:3306**.

Cuối cùng tạo **/rate-my-lizard/dbstring**: Standard, String, giá trị **db.thisisprettyrandom.com:3306**.

Tổng cộng chúng ta đã tạo **5 tham số**: 3 cái cho my-cat-app (trong đó dbpassword được mã hóa), 1 cho my-dog-app và 1 cho rate-my-lizard.

Bây giờ chuyển sang dòng lệnh. Để đơn giản, dùng **CloudShell** (tính năng mới của AWS). Click icon **CloudShell** ở thanh menu trên cùng. Chờ provisioning hoàn tất (sẽ hiển thị "Preparing your terminal").

Để tương tác với Parameter Store qua CLI, dùng lệnh: **aws ssm get-parameters**.

Ví dụ lấy tham số **/rate-my-lizard/dbstring** → trả về JSON chứa thông tin name, type, value, version, last modified date, ARN…

Tương tự có thể lấy từng tham số khác: /my-dog-app/dbstring, /my-cat-app/dbstring.

Thay vì lấy từng cái, chúng ta có thể dùng **get-parameters-by-path** để lấy theo đường dẫn phân cấp.

Ví dụ: **aws ssm get-parameters-by-path --path /my-cat-app** → trả về 3 tham số (dbpassword, dbstring, dbuser). Lưu ý dbpassword mặc định trả về dạng **ciphertext** (mã hóa).

Để giải mã (decrypt), thêm tùy chọn **--with-decryption**. Lệnh đầy đủ: **aws ssm get-parameters-by-path --path /my-cat-app --with-decryption**

**Lưu ý quan trọng**: Quyền truy cập **Parameter Store** và quyền **KMS** (để decrypt) là tách biệt. IAM admin user có quyền admin nên thực hiện được cả hai.

**Parameter Store** sẽ được sử dụng rất nhiều trong khóa học này và các khóa khác. Đây là cách tuyệt vời để cung cấp cấu hình cho ứng dụng (AWS-native hoặc custom). Tốt hơn nhiều so với việc hardcode hoặc truyền bằng cách khác.

Cuối cùng, dọn dẹp: Đóng CloudShell, quay lại console Parameter Store, chọn tất cả 5 tham số vừa tạo (bỏ chọn nếu có tham số khác), click **Delete** và xác nhận.

Tài khoản đã được đưa về trạng thái ban đầu. Hoàn tất video này và sẵn sàng sang bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Tạo tham số: Standard vs Advanced, String / StringList / SecureString
- Hierarchical structure (/my-cat-app/dbstring)
- SecureString + KMS encryption
- Public parameters & CLI: get-parameters, get-parameters-by-path
- --with-decryption và quyền IAM + KMS
- CloudShell demo + Cleanup

**Notes (Chi tiết):**

- **Parameter Store** nằm trong **Systems Manager**. Tạo tham số với tên phân cấp bằng dấu /.
- Tạo 5 tham số demo: 3 cho **my-cat-app** (dbstring, dbuser, dbpassword SecureString), 1 cho **my-dog-app**, 1 cho **rate-my-lizard**.
- Lấy tham số qua CLI: aws ssm get-parameters (từng cái) hoặc aws ssm get-parameters-by-path --path /my-cat-app.
- Thêm --with-decryption để giải mã SecureString (cần quyền KMS riêng).
- Không mất chi phí với **Standard tier** dưới 10.000 tham số.
- **Cleanup**: Chọn và xóa tất cả tham số demo để khôi phục tài khoản.

**Summary (Tóm tắt ngắn):** Bài demo thực hành **AWS Parameter Store** qua Console và **CloudShell CLI**: tạo tham số phân cấp, SecureString với KMS, truy xuất bằng lệnh get-parameters và get-parameters-by-path (có decrypt). Nhấn mạnh sự tách biệt quyền IAM và KMS. Kết thúc bằng quy trình **dọn dẹp** để trả tài khoản về trạng thái ban đầu.