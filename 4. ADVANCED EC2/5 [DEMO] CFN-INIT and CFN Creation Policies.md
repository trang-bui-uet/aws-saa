Chào mừng bạn quay trở lại và trong bản demo ngắn gọn này, bạn sẽ có cơ hội **tạo một EC2 instance** với **WordPress được bootstrap sẵn**, sẵn sàng để cấu hình. Nhưng lần này, bạn sẽ sử dụng **template CloudFormation nâng cao** sử dụng **cfn-init** và **creation policies** thay vì user data đơn giản mà bạn đã dùng ở bản demo trước.

Để bắt đầu, hãy chắc chắn bạn đã đăng nhập vào **AWS account chung** với vai trò **IAM admin user** và như mọi khi, hãy chọn region **Northern Virginia**.

Bây giờ, bài học đính kèm hai đường link **one-click deployment**. Hãy sử dụng link đầu tiên là **VPC link**. Mọi thứ đã được điền sẵn. Bạn chỉ cần cuộn xuống dưới cùng, đánh dấu vào ô xác nhận và click **Create stack**. Sau khi stack chuyển sang trạng thái **CREATE_COMPLETE**, bạn có thể tiếp tục và chúng ta sẽ tiến hành demo.

Okay, giả sử stack đã ở trạng thái **CREATE_COMPLETE**. Bây giờ chúng ta sẽ áp dụng một template CloudFormation khác. Đây là template chúng ta sẽ dùng. Nó chỉ là phiên bản nâng cao của template bạn dùng ở bài trước. Lần này, thay vì dùng một tập hợp các chỉ dẫn thủ tục (script) truyền vào **user data**, nó sử dụng hệ thống **cfn-init** và **creation policies**.

Hãy xem cụ thể điều đó nghĩa là gì.

Nếu tôi cuộn xuống và tìm **logical resource EC2 instance**, thì ở đây chúng ta có **creation policy**. Điều này có nghĩa là CloudFormation sẽ tạo một **điểm giữ (hold point)**. Nó sẽ không cho phép resource này chuyển sang trạng thái **CREATE_COMPLETE** cho đến khi nhận được **signal**. Và nó sẽ chờ signal trong **15 phút** (timeout 15 phút).

Tiếp tục cuộn xuống xem **user data**, những thứ chúng ta làm theo cách thủ tục chỉ là dùng lệnh **cfn-init** để bắt đầu **desired state configuration**. Nó sẽ thành công hoặc thất bại, và dựa trên đó chúng ta dùng lệnh **cfn-signal** để gửi trạng thái thành công hoặc thất bại trở lại CloudFormation stack – đây chính là thứ sử dụng **creation policy**. Creation policy sẽ chờ signal, và chính lệnh này cung cấp signal đó (success hoặc failure).

Điều chúng ta quan tâm cụ thể trong demo bài này là lệnh **cfn-init**. Đây là thứ kéo **desired state configuration** từ **metadata** của logical resource. Tôi sẽ nói chi tiết về điều đó ngay sau. Nó kéo thông tin đó xuống bằng cách được cung cấp **stack ID**, và sử dụng lệnh thay thế này. Thay vì truyền tên biến này vào instance, cái thực sự được truyền thay cho tên biến **stack ID** là **stack ID thực tế**. Tương tự, thay vì tên biến **AWS::Region**, thì **region thực tế** mà template đang được áp dụng sẽ được truyền vào. Đó là chức năng của **substitution function** – nó thay thế bất kỳ tên biến hoặc parameter nào bằng giá trị thực của chúng.

Quá trình **cfn-init** sau đó có thể tham khảo CloudFormation stack và lấy thông tin cấu hình. Tất cả được lưu trong phần **metadata** của logical resource này.

Tôi muốn bạn chú ý đến dòng này: **--configsets WordPress_Install**. Điều này cho biết chúng ta muốn **cfn-init** chạy bộ chỉ dẫn nào. Nếu tôi mở rộng phần metadata, chúng ta có một hoặc nhiều **config sets**. Trong trường hợp này chỉ có một: **WordPress_Install**, và config set này chạy **năm item riêng lẻ** theo thứ tự, chúng được gọi là **config keys**: InstallCFN, SoftwareInstall, ConfigureInstance, InstallWordPress, và ConfigureWordPress.

Những config key này tham chiếu đến các config keys được định nghĩa bên dưới, bạn sẽ thấy chúng có tên giống nhau. Bạn sẽ nhận ra nhiều lệnh vì chúng giống hệt các lệnh dùng để cài đặt và cấu hình WordPress.

Trong config key **SoftwareInstall**, chúng ta dùng **DNF package manager** để cài các gói phần mềm cần thiết như wget, MariaDB, Apache web server và nhiều tiện ích khác. Sau đó là phần **services**, chúng ta chỉ định muốn các service này được **enabled** và **running**. Nghĩa là service sẽ tự khởi động khi instance boot và đảm bảo nó đang chạy ngay lúc này.

Config key tiếp theo là **ConfigureInstance**. Phần **files** có thể tạo file với nội dung nhất định. Chúng ta đang tạo file **/etc/update-motd.d/40-cow** – đây là phần trước đây phải làm thủ công, dùng để thêm **cow-say banner**. Sau đó chạy thêm một số lệnh thủ tục để đặt mật khẩu root database và cập nhật banner.

Tiếp theo là **InstallWordPress**, sử dụng tùy chọn **sources** để giải nén nội dung được chỉ định vào thư mục này. Nó tự động xử lý download, un-gzip, un-tar archive vào thư mục, thậm chí có thể làm với authentication nếu cần. Chúng ta tạo thêm một file khác để cấu hình WordPress, và một file nữa để tạo database cho WordPress. Cuối cùng là **ConfigureWordPress** sửa quyền và tạo các database.

Đây đang làm **giống hệt** những gì ví dụ thủ tục ở demo trước. Thay vì chạy từng lệnh một, giờ chỉ dùng **desired state**.

Còn một điều nữa tôi muốn chỉ ra ngay ở đầu template. Đây là phần cấu hình **cfn-hup** để liên tục theo dõi cấu hình logical resource trong CloudFormation stack. Nếu nó phát hiện metadata của **EC2Instance** trong stack thay đổi, nó sẽ chạy **cfn-init** lại. Nhớ ở bài lý thuyết tôi đã nói quá trình này có thể xử lý stack updates chứ? Nó không chỉ chạy một lần như user data. Đây chính là cách nó làm. Nó cấu hình cập nhật tự động theo dõi CloudFormation stack và chạy lại cfn-init bất cứ khi nào có thay đổi. Điều này vượt xa những gì bạn cần cho kỳ thi liên quan, tôi chỉ muốn bạn biết nó là gì và cách nó hoạt động. Về cơ bản, chúng ta thiết lập process **cfn-hup** và cho phép nó theo dõi stack liên tục.

Đó là toàn bộ nội dung template. Bây giờ chúng ta sẽ áp dụng nó. Hãy click vào **one-click deployment link thứ hai** đính kèm bài học, tên là **A4L-EC2-CFN-INIT**. Click link đó, cuộn xuống dưới và click **Create stack**.

Lần này nhớ rằng chúng ta đang dùng **creation policy**, nên CloudFormation sẽ không chuyển logical ID sang **CREATE_COMPLETE** khi EC2 báo launch hoàn tất. Thay vào đó, nó sẽ chờ chính instance gửi signal thành công của quá trình **cfn-init**.

Vì dùng creation policy, nó sẽ giữ cho đến khi hệ điều hành instance, thông qua **cfn-signal**, gửi signal cho CloudFormation rằng “Mọi thứ ổn rồi”. Lúc đó logical resource mới chuyển sang CREATE_COMPLETE. Quá trình này mất vài phút. EC2 instance cần launch, pass status checks, sau đó cfn-init chạy toàn bộ cấu hình, rồi cfn-signal gửi trạng thái thành công, CloudFormation mới đánh dấu resource và stack hoàn tất. Hãy refresh liên tục, sau 2-3 phút trạng thái sẽ cập nhật. Bạn có thể pause video và resume khi stack đã CREATE_COMPLETE.

Và đây rồi. Lúc này stack đã chuyển sang **CREATE_COMPLETE**, tôi muốn bạn chú ý dòng này (bạn chưa thấy trước đây). Đây là dòng mà EC2 instance đã chạy **cfn-init** thành công, rồi lệnh **cfn-signal** đã gửi success signal đến CloudFormation stack. Đây chính là signal mà CloudFormation đang chờ trước khi chuyển resource sang CREATE_COMPLETE, và đó là điều cần thiết để stack hoàn tất. Bây giờ chúng ta **rõ ràng biết** việc cấu hình instance đã hoàn tất. Chúng ta không chỉ dựa vào EC2 báo instance running 2/2 checks, mà chính hệ điều hành qua cfn-init đã hoàn thành và cfn-signal đã báo tường minh.

Nếu chuyển sang **EC2 console**, chúng ta có thể connect vào instance như trước. Tìm instance đang running, copy **Public IPv4 address** và mở tab mới. Nếu mọi thứ ổn, bạn sẽ thấy màn hình cài đặt WordPress quen thuộc.

Nếu right-click instance → Connect → Instance Connect → Connect, bạn sẽ vào instance và thấy **cow-themed login banner**. Lần này nếu dùng curl xem nội dung user data, nó chỉ còn rất ít dòng vì chỉ chạy cfn-init và cfn-signal. Chú ý cách tất cả tên biến đã được thay bằng giá trị thực (stack ID và region). Đây là cách nó biết giao tiếp với stack đúng và region đúng trong CloudFormation.

Nếu cd /var/log rồi ls, chúng ta vẫn có hai file cũ: **cloud-init.log** và **cloud-init-output.log** (liên quan đến user data). Nhưng giờ có thêm các file mới: **cfn-init-cmd.log** – đây là output của quá trình cfn-init. Dùng sudo cat file đó để xem, bạn sẽ thấy từng config key chạy và các hoạt động bên trong.

Đến đây là tất cả những gì tôi muốn trình bày. Chỉ để bạn có trải nghiệm thực tế về một giải pháp thay thế cho raw user data, đó là **cfn-init**. Đây là hệ thống mạnh mẽ hơn nhiều, đặc biệt khi kết hợp với **CloudFormation creation policies**, cho phép tạm dừng tiến trình stack, chờ chính resource báo “Tôi đã hoàn tất bootstrapping, bạn có thể tiếp tục”, và điều này được thực hiện qua lệnh **cfn-signal**.

Bây giờ hãy dọn dẹp account. Quay lại CloudFormation, xóa stack **EC2-CFN-INIT**. Chờ hoàn tất, sau đó xóa stack **A4L-VPC**. Điều này sẽ đưa AWS account về trạng thái ban đầu của demo. Cảm ơn bạn đã thực hành demo. Hy vọng hữu ích. Bạn có thể hoàn tất video này, và khi sẵn sàng thì gặp tôi ở bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Demo CloudFormation nâng cao với **cfn-init** & **creation policies**
- One-click deployment: VPC stack + A4L-EC2-CFN-INIT stack
- Creation Policy + cfn-signal (timeout 15 phút)
- Configsets: WordPress_Install với 5 config keys (InstallCFN, SoftwareInstall, ConfigureInstance, InstallWordPress, ConfigureWordPress)
- cfn-hup theo dõi thay đổi metadata
- Kiểm tra WordPress, banner cow, log files (cfn-init-cmd.log)
- Cleanup: Xóa 2 stacks

**Notes (Chi tiết):**

- **Creation Policy** tạo hold point, chờ signal từ instance qua **cfn-signal** sau khi **cfn-init** chạy xong thay vì chỉ dựa vào EC2 launch.
- **User Data** giờ chỉ chạy cfn-init (kéo metadata) và cfn-signal; substitution thay thế biến stack ID & region.
- **Metadata** định nghĩa config sets & config keys: cài gói phần mềm (DNF), services, tạo file banner, download & config WordPress, sửa quyền.
- **cfn-hup** cho phép rerun cfn-init khi stack update (khác với user data chỉ chạy 1 lần).
- Sau CREATE_COMPLETE: truy cập Public IP xem WordPress, Instance Connect xem banner, kiểm tra log cfn-init.
- Cleanup thủ công để trả account về trạng thái sạch.

**Summary (Tóm tắt ngắn):** Bài học hướng dẫn cách bootstrap EC2 với **WordPress** bằng **CloudFormation template** sử dụng **cfn-init** và **creation policies** thay thế user data thủ công. Template tự động cài đặt, cấu hình qua metadata và config keys, gửi signal thành công. Kết thúc bằng kiểm tra thực tế và quy trình dọn dẹp 2 stacks.