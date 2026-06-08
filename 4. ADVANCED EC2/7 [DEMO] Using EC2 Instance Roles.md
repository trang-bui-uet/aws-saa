Chào mừng bạn đến với bài demo này, nơi bạn sẽ có trải nghiệm thực hành làm việc với **EC2** và **EC2 instance roles**. Như bạn đã học ở bài lý thuyết trước, **instance role** về cơ bản là một loại **IAM role** đặc biệt được thiết kế để có thể được assume bởi một EC2 instance. Khi instance assume role (quá trình này xảy ra tự động khi hai bên được liên kết), instance đó và bất kỳ ứng dụng nào chạy trên instance sẽ nhận được temporary security credentials mà role cung cấp. Trong bài demo này, bạn sẽ thực hành toàn bộ quy trình.

Để bắt đầu, bạn cần một số hạ tầng. Hãy đảm bảo bạn đã đăng nhập vào **general AWS account** (tức là management account của organization), và như mọi khi, bạn cần ở trong region **Northern Virginia**. Giả sử bạn đã sẵn sàng, có một **one-click deployment link** được đính kèm trong bài học. Hãy click vào link đó. Nó sẽ dẫn bạn đến trang quick create stack. Tên stack sẽ được điền sẵn là **"iam-role-demo"**, bạn chỉ cần cuộn xuống dưới, tick vào ô capabilities và click **Create stack**. Việc triển khai one-click này sẽ tạo **Animals for Life VPC**, một **EC2 instance**, và một **S3 bucket**.

Để tiếp tục bài demo, chúng ta cần stack ở trạng thái **"Create complete"**. Hãy tạm dừng video, chờ đến khi stack chuyển sang trạng thái Create complete thì chúng ta tiếp tục. Okay, stack giờ đã ở trạng thái Create complete, chúng ta có thể tiếp tục. Hãy click vào menu Services, gõ **"EC2"**, tìm và right-click mở trong tab mới. Tại EC2 console, click vào **"Instances running"**, bạn sẽ thấy chỉ có duy nhất một EC2 instance.

Bây giờ chúng ta muốn kết nối vào instance để thực hiện các tác vụ trong demo. Right-click vào instance, chọn **"Connect"**. Chúng ta sẽ dùng **EC2 Instance Connect**. Xác nhận username là **"ec2-user"** rồi click Connect. AMI dùng để launch instance là Amazon Linux 2 AMI tiêu chuẩn, nên nếu gõ lệnh **"aws"** và Enter, nó sẽ có AWS CLI version 2 được cài sẵn.

Điều quan trọng cần hiểu là hiện tại instance này **không có instance role nào được gắn**, và chưa được cấu hình gì thêm. Đây là AMI Amazon Linux 2 nguyên bản. Nếu cố gắng tương tác với AWS qua command line — ví dụ chạy **aws s3 ls** — thì CLI tools sẽ báo không có credentials nào được cấu hình trên instance, và yêu cầu cung cấp long-term credentials qua lệnh **aws configure**. Đây chính là cách bạn đã dùng để cung cấp credentials cho bản CLI cài trên máy local. Bạn dùng **aws configure** để thiết lập các profile và cung cấp authentication qua **access keys**.

Instance này hiện không có access keys nào, nên không có cách nào tương tác với AWS. Chúng ta có thể dùng **aws configure** để cung cấp credentials, nhưng đó **không phải best practice** cho EC2 instance. Thay vào đó, chúng ta sẽ dùng **instance role**. Để làm vậy, hãy quay lại AWS console. Click Services, gõ **"IAM"** trong ô tìm kiếm. Mở IAM console trong tab mới. Như đã nói ở bài lý thuyết, instance role chỉ là một loại IAM role đặc biệt. Chúng ta sẽ tạo một IAM role để instance assume. Click **"Roles"** → **"Create role"**.

Quy trình create role đưa ra một số scenario phổ biến: role cho AWS service, AWS account khác, web identity, hoặc SAML 2.0 federation. Trong trường hợp của chúng ta, chọn role có thể được assume bởi **AWS service**, cụ thể là **EC2**. Chọn **"AWS service"** → **"EC2"** → **Next**.

Ở phần permissions, gõ **"S3"** vào ô tìm kiếm, chọn policy **"AmazonS3ReadOnlyAccess"**. Tick vào ô bên cạnh và click **Next**. Đặt **Role name** là **"A4L-instance-role"** để dễ phân biệt. Nhập tên và click **"Create role"**.

Như đã giải thích ở bài lý thuyết về instance roles, khi tạo từ giao diện UI, nó sẽ tự động tạo cả **IAM role** và **instance profile** cùng tên. Chính **instance profile** là thứ chúng ta sẽ gắn vào EC2 instance. Từ góc nhìn UI, cả hai trông giống nhau, bạn không thấy chúng là hai thực thể riêng biệt, nhưng chúng tồn tại. Bây giờ quay lại **EC2 console**. Hiện tại instance chưa có instance role nào, nên không thể tương tác với AWS.

Để gắn instance role qua console UI: right-click instance → **Security** → **Modify IAM role**. Chọn role mới từ dropdown — chọn **A4L-instance-role** mà bạn vừa tạo → click **"Save"**.

Bây giờ chọn instance và click **"Security"**, bạn sẽ xác nhận nó đã có IAM role được gắn. Đây là instance role mà EC2 instance có thể sử dụng. Tiếp tục tương tác với instance qua hệ điều hành. Nếu đã vài phút kể từ lần kết nối Instance Connect trước, màn hình có thể bị treo. Lúc đó chỉ cần đóng tab đang kết nối, right-click instance → Connect lại với username **ec2-user**.

Lần trước chúng ta chạy **aws s3 ls** và nhận thông báo không có credentials. Bây giờ thử lại lệnh **aws s3 ls**. Nhờ có instance role được gắn, CLI tools sẽ tự động sử dụng temporary credentials mà role tạo ra.

Cách hoạt động là (tôi sẽ demo bằng lệnh curl): credentials được cung cấp cho CLI tools qua **instance metadata**. Đây là đường dẫn metadata mà CLI tools dùng để lấy security credentials (temporary credentials khi role được assume). Chạy lệnh curl tương ứng, bạn sẽ thấy nó trả về tên role được gắn với instance.

Nếu dùng curl với đường dẫn đầy đủ đến **security-credentials** + tên role, bạn sẽ thấy chi tiết credentials: **AccessKeyId**, **SecretAccessKey**, và **Token**. Tất cả đều được sinh ra khi EC2 instance assume role. Vì là temporary credentials nên chúng có **expiry date** (ví dụ: hết hạn ngày 7/5/2022 lúc 5:52:47 UTC).

Đó là tất cả những gì tôi muốn trình bày trong demo về instance roles. Về cơ bản, bạn chỉ cần tạo instance role rồi gắn vào instance. Khi đó, instance có khả năng assume role, nhận temporary credentials, và mọi ứng dụng (kể cả CLI tools) đều có thể tương tác với AWS.

Quá trình renew credentials diễn ra **tự động**. Miễn là ứng dụng trên instance định kỳ kiểm tra metadata service, nó sẽ luôn có credentials hợp lệ và mới nhất. Khi expiry date đến gần hoặc qua đi, EC2 service sẽ renew và cung cấp bộ credentials mới qua metadata.

Trước khi kết thúc, tôi muốn cho bạn xem một link (đã đính kèm) về thứ tự ưu tiên (precedence) mà CLI tools dùng để lấy credentials:

- Đầu tiên: command line options.
- Tiếp theo: environment variables.
- Sau đó: CLI credentials file (~/.aws/credentials).
- Tiếp: CLI configuration file.
- Tiếp: container credentials.
- Cuối cùng: **instance profile credentials** (chính là những gì chúng ta vừa demo).

Điều này có nghĩa nếu bạn cấu hình long-term credentials thủ công qua **aws configure**, chúng sẽ được ưu tiên hơn instance profile. Nhưng best practice là dùng **instance profile** và gắn cho nhiều instance để cung cấp quyền truy cập AWS an toàn.

Đến đây là toàn bộ nội dung demo. Bây giờ chúng ta dọn dẹp hạ tầng. Quay lại **IAM console** → **Roles** → chọn **A4L-instance-role** → **Delete role**.

Sau đó quay lại **EC2 console** → chọn instance → **Security** → **Modify IAM role** → chọn **No IAM role** → **Save** (xác nhận bằng cách gõ **"detach"**). Dù role đã xóa, instance profile vẫn hiển thị — đây là cách minh họa sự khác biệt.

Cuối cùng, vào **CloudFormation console**, chọn stack **iam-role-demo** → **Delete** để đưa account về trạng thái ban đầu.

Đây là một demo ngắn gọn để bạn có trải nghiệm thực tế với instance roles. EC2 instances kết hợp với IAM roles giúp instance và các ứng dụng trên đó có khả năng tương tác an toàn với các dịch vụ AWS khác. Bạn sẽ sử dụng khá thường xuyên trong khóa học, đặc biệt khi cấu hình AWS service tương tác với service khác. Đó là use case phổ biến của IAM roles.

Hãy hoàn thành video và khi sẵn sàng, hẹn gặp bạn ở bài học tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- One-click CloudFormation stack (iam-role-demo)
- Tạo IAM Role + Instance Profile (A4L-instance-role)
- Attach Instance Role qua EC2 console
- Credentials qua Instance Metadata (curl demo)
- Precedence của AWS CLI credentials
- Cleanup: Delete role, detach profile, delete stack

**Notes (Chi tiết):**

- Stack tạo VPC + EC2 (Amazon Linux 2) + S3 bucket. Kết nối bằng **EC2 Instance Connect**.
- Ban đầu instance không có role → aws s3 ls báo lỗi, yêu cầu **aws configure** (access keys) — **không phải best practice**.
- Tạo role cho EC2 service với policy **AmazonS3ReadOnlyAccess** → UI tự tạo cả **Instance Profile**.
- Attach role → CLI tự động dùng temporary credentials từ metadata (luôn renew tự động).
- Thứ tự CLI kiểm tra credentials: command line → env vars → credentials file → config file → container → **instance profile** (thấp nhất nhưng best practice).
- Cleanup cẩn thận: xóa IAM role trước, detach instance profile, sau đó xóa CloudFormation stack.

**Summary (Tóm tắt ngắn):** Bài demo thực hành **EC2 Instance Roles**: Tạo IAM Role + Instance Profile, attach vào instance để CLI và ứng dụng tự động sử dụng **temporary credentials** an toàn qua metadata (không cần access keys). Hướng dẫn đầy đủ từ triển khai one-click, demo curl metadata, thứ tự precedence CLI, đến **cleanup** hoàn chỉnh. Đây là best practice quan trọng khi EC2 tương tác với các dịch vụ AWS khác.