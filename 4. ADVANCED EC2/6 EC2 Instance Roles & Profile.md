Chào mừng bạn quay lại. Trong khóa học, tôi đã đề cập vài lần rằng **IAM roles** là cách thực hành tốt nhất để các dịch vụ AWS được cấp quyền truy cập vào các dịch vụ AWS khác thay mặt bạn. Việc cho phép một dịch vụ assume một role sẽ cấp cho dịch vụ đó những quyền mà role đó sở hữu.

**EC2 instance roles** là các role mà một instance có thể assume, và bất kỳ thứ gì đang chạy trong instance đó sẽ có quyền hạn mà role cấp. Có một số chi tiết quan trọng cần lưu ý, vì vậy hãy cùng xem cách hoạt động kiến trúc của tính năng này trong EC2.

Kiến trúc của **instance role** không thực sự phức tạp. Nó bắt đầu từ một **IAM role**, và role đó có **permissions policy** được gắn vào. Ai assume role sẽ nhận được temporary credentials được tạo ra, và những temporary credentials này sẽ cấp quyền theo những gì permissions policy cho phép.

**EC2 instance role** cho phép dịch vụ EC2 assume role đó, nghĩa là chính EC2 instance có thể assume role và nhận được quyền truy cập vào credentials. Tuy nhiên, chúng ta cần một cách để đưa credentials vào bên trong EC2 instance để các ứng dụng chạy bên trong có thể sử dụng quyền từ role. Do đó, có một lớp kiến trúc trung gian: **instance profile**. Đây là một wrapper bao quanh IAM role, và instance profile chính là thứ cho phép permissions đi vào bên trong instance.

Khi bạn tạo instance role trong console, một instance profile sẽ được tạo tự động với cùng tên. Nhưng nếu dùng command line hoặc CloudFormation, bạn phải tạo hai thứ này riêng biệt. Khi dùng giao diện UI, bạn tưởng rằng đang gắn instance role trực tiếp vào instance, nhưng thực tế không phải vậy. Bạn đang gắn **instance profile** có cùng tên. Chính **instance profile** mới được gắn vào EC2 instance.

Chúng ta đã biết rằng khi IAM roles được assume, bạn sẽ nhận temporary security credentials có thời hạn. Những credentials này cấp quyền dựa trên permissions policy của role. Bên trong EC2 instance, credentials được cung cấp qua **instance metadata**. Các ứng dụng chạy bên trong instance có thể truy cập credentials này để sử dụng khi truy cập tài nguyên AWS như S3.

Một điểm tuyệt vời của kiến trúc này là credentials có sẵn trong metadata luôn hợp lệ. EC2 và security token service phối hợp với nhau để đảm bảo credentials luôn được renew trước khi hết hạn. Miễn là ứng dụng bên trong EC2 instance tiếp tục kiểm tra metadata, nó sẽ không bao giờ rơi vào tình trạng credentials hết hạn.

Tóm lại, khi sử dụng **EC2 instance roles**, credentials được cung cấp qua instance metadata. Cụ thể, trong metadata có một cây IAM. Bên trong đó có phần security credentials, rồi bên trong nữa là tên role. Nếu truy cập vào đây, bạn sẽ nhận temporary security credentials. Chúng luôn được rotate và luôn hợp lệ. Miễn là instance role vẫn được gắn vào instance, bất kỳ thứ gì chạy bên trong instance sẽ luôn có quyền truy cập vào các credentials hợp lệ này.

Tất nhiên, các ứng dụng chạy trong instance cần cẩn thận khi cache credentials, và nên kiểm tra metadata trước khi credentials hết hạn, hoặc kiểm tra định kỳ.

Bạn nên **luôn sử dụng roles** bất cứ khi nào có thể. Tôi sẽ tiếp tục nhấn mạnh điều này xuyên suốt khóa học. Đây là nội dung quan trọng cho kỳ thi. **Roles** luôn tốt hơn việc lưu trữ long-term credentials (như access keys) vào EC2 instance. Không bao giờ là ý hay khi lưu long-term credentials (access keys) ở bất kỳ nơi nào không được bảo mật an toàn, ví dụ như trên máy local của bạn. Thực tế, các công cụ AWS (như CLI tools) sẽ tự động sử dụng instance role credentials. Vì vậy, miễn là instance role được gắn vào EC2 instance, bất kỳ lệnh command-line tool nào chạy bên trong instance đều có thể tự động sử dụng credentials đó.

Đến đây là tất cả những gì tôi muốn chia sẻ. Cảm ơn bạn đã theo dõi. Hãy hoàn thành video này, và khi sẵn sàng, hãy cùng tôi sang bài học tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- IAM roles là best practice cho AWS services
- EC2 instance roles & Instance Profile
- Temporary credentials qua Instance Metadata
- Tự động renew credentials
- Ưu tiên roles thay vì access keys
- AWS CLI tự động sử dụng role credentials

**Notes (Chi tiết):**

- **IAM Role** có permissions policy → khi assume sẽ nhận temporary credentials.
- **Instance Profile** là wrapper quanh IAM Role, được gắn vào EC2 instance (console tự tạo, CLI/CFN phải tạo riêng).
- Credentials được cung cấp qua **instance metadata** (IAM tree → security credentials → role name).
- EC2 và STS phối hợp renew credentials trước khi hết hạn → luôn valid nếu ứng dụng kiểm tra metadata định kỳ.
- Không nên lưu access keys (long-term credentials) vào instance hoặc local machine.
- Công cụ AWS CLI tự động dùng instance role khi có sẵn.

**Summary (Tóm tắt ngắn):** Bài học giải thích **kiến trúc EC2 Instance Roles**: IAM Role + Instance Profile → credentials được đưa vào instance qua metadata, luôn tự động renew và valid. Khuyến khích **sử dụng roles** thay vì lưu access keys để tăng bảo mật, đây là best practice quan trọng cho cả thực tế và kỳ thi AWS.