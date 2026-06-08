Chào mừng các bạn đã quay trở lại. Đây là phần hai của bài học này. Chúng ta sẽ tiếp tục ngay từ điểm kết thúc của phần một, vì vậy hãy cùng bắt đầu nào.

Giờ đây, ngoài việc sử dụng tính năng **user data (dữ liệu người dùng)** và khởi chạy các thực thể (instances) một cách thủ công, tự nhập dữ liệu người dùng này rồi dùng nó để **bootstrap (khởi động/cấu hình tự động)** quá trình khởi chạy của một thực thể EC2, chúng ta còn có thể tận dụng chức năng này từ ngay bên trong **CloudFormation**. Đính kèm với bài học này là một liên kết triển khai một-click (one-click deployment) bổ sung, liên kết này sẽ triển khai một thực thể EC2 WordPress khác. Và tôi muốn đi từng bước để giải thích chính xác những gì mà việc triển khai một-click này thực hiện.

Vì vậy, đây là một **mẫu CloudFormation (CloudFormation template)** khác. Chúng ta sẽ thảo luận nhiều hơn về CloudFormation khi tiếp tục lộ trình khóa học, hiện tại bạn chưa cần hiểu rõ nó làm gì. Chỉ cần nghĩ về nó như một dạng **tự động hóa (automation)** mà bạn có thể sử dụng để triển khai các tài nguyên AWS. Bây giờ, có một điểm cụ thể mà tôi muốn rút ra: nếu chúng ta cuộn xuống tệp này, tôi muốn bạn chú ý đến **nơi chúng ta tạo một thực thể EC2**. Tôi sẽ tiếp tục cuộn cho đến khi tìm thấy nó. Đây rồi, chúng ta có **public EC2**.

Những gì bạn có thể làm bên trong một mẫu CloudFormation, ngoài việc tự động khởi chạy một thực thể EC2, là bạn cũng có thể **cung cấp user data**. Vì vậy, trong trường hợp cụ thể này, hãy chú ý cách tôi sử dụng hàm này, bởi vì hãy nhớ rằng, **user data cần được cung cấp dưới dạng mã hóa base64**. Trong khi giao diện bảng điều khiển (console UI) cho phép bạn cung cấp một tệp văn bản thô và tự động chuyển đổi nó như một phần của quá trình khởi chạy, thì **khi sử dụng CloudFormation, bạn cần lấy văn bản thô và chạy nó qua một hàm mã hóa base64** – đó chính là những gì đoạn này thực hiện. Ngoài điểm đó ra thì mọi thứ hoàn toàn giống nhau. Điều này có nghĩa là thực thể này, khi được khởi chạy bởi CloudFormation, sẽ có user data được chạy như một phần của quá trình khởi chạy, do đó nó sẽ thực hiện chính xác cùng một tập hợp các hành động. Nó sẽ cấp phát một thực thể và **tự động cấu hình nó trong quá trình khởi chạy**. Vì vậy, bạn sẽ nhận ra các câu lệnh tương tự, cùng một quá trình cài đặt, cấu hình và biểu ngữ đăng nhập tùy chỉnh (custom login banner).

Do đó, liên kết này được đính kèm thêm vào bài học này như một tùy chọn triển khai một-click bổ sung. Nếu bạn nhấp vào liên kết bổ sung đính kèm với bài học này, nó sẽ đưa bạn đến màn hình tạo nhanh stack (quick create stack). Lần này, stack này sẽ được đặt tên là **bootstrap-CFN**. Bởi vì chúng ta đang sử dụng CloudFormation để bootstrap thực thể này, hệ thống sẽ hỏi bạn tất cả các giá trị quan trọng — tên cơ sở dữ liệu, mật khẩu cơ sở dữ liệu, mật khẩu root của cơ sở dữ liệu, người dùng cơ sở dữ liệu. Tất cả chúng đều được cài đặt sẵn bằng các giá trị mặc định. Tất cả những gì bạn cần làm là **tích vào hộp thoại xác nhận quyền hạn (capabilities box) ở dưới cùng và nhấp vào tạo stack (create stack)**. Bây giờ, hãy đảm bảo rằng **bootstrap-CFN** đã được chọn, và nếu bạn chọn "template" (mẫu), đây chính là mẫu đang được sử dụng để tạo stack này. Đây là mẫu mà nếu chúng ta cuộn xuống, bạn sẽ thấy rằng **user data đang được cung cấp như một phần của mẫu CloudFormation này**. Vì vậy, chúng ta đang sử dụng hàm base64 để mã hóa văn bản thô này thành base64. Chúng ta truyền dữ liệu đó vào thực thể như một phần của quá trình khởi chạy của nó, và do đó, việc này sẽ sử dụng cùng một dữ liệu người dùng mà tôi đã minh họa trước đó trong bài học này. **Kết quả cuối cùng là một thực thể EC2 khác đã được cấp phát bằng user data (được bootstrap), nhưng lần này là sử dụng CloudFormation.**

Chúng ta sẽ cần đợi vài phút để quá trình tạo hoàn tất, vì vậy hãy tạm dừng video và tiếp tục lại khi stack **bootstrap-CFN** chuyển sang trạng thái **create complete (tạo hoàn tất)**.

Được rồi, bây giờ stack đã chuyển sang trạng thái tạo hoàn tất. Nếu chúng ta nhấp vào **resources (tài nguyên)**, cuộn xuống, tìm **public EC2** bên trong stack **bootstrap-CFN**, và nhấp vào **physical ID (ID vật lý)** của nó, thao tác này sẽ chuyển chúng ta đến bảng điều khiển EC2 và chỉ hiển thị thực thể này — đây là thực thể được tạo bởi stack thứ hai. Hãy tiếp tục và chọn thực thể này. Chúng ta sẽ cần cho nó vài phút để hoàn thành quá trình cấp phát, nhưng sau khi hoàn tất, nếu chúng ta **sao chép địa chỉ IP công cộng phiên bản 4 (public IP version 4)** của nó vào khay nhớ tạm (clipboard) — lưu ý không sử dụng liên kết "open address" (mở địa chỉ) trực tiếp, mà hãy sao chép địa chỉ đó rồi mở nó trong một tab mới. **Những gì bạn sẽ thấy, giống như ví dụ trước về việc bootstrap bằng cách nhập user data thủ công, là hộp thoại cài đặt WordPress.** Và nếu chúng ta nhấp chuột phải vào thực thể này, đi tới **connect (kết nối)**, chọn **instance connect**, chúng ta sẽ thấy **biểu ngữ đăng nhập "Animals for Life" tùy chỉnh** của chúng ta.

Như vậy, chúng ta đã thực hiện cùng một quy trình bootstrap thực thể này, chỉ khác là lần này chúng ta lưu trữ dữ liệu người dùng bên trong mẫu CloudFormation. Và khi bạn tiếp tục lộ trình khóa học, bạn sẽ thấy ngày càng nhiều ví dụ phức tạp hơn về việc sử dụng CloudFormation tận dụng quy trình bootstrap này. Bạn cũng sẽ có quyền truy cập vào một bài học chuyên sâu ngắn về CloudFormation (CloudFormation mini deep dive), nơi tôi sẽ đi vào chi tiết hơn về cách CloudFormation hoạt động và cách bạn có thể tự tạo các mẫu CloudFormation của riêng mình. Tại thời điểm này, đó là tất cả những gì tôi muốn minh họa trong video này.

Vì vậy, hãy **bắt đầu quá trình dọn dẹp tài khoản**. Chúng ta cần đưa tài khoản trở lại trạng thái giống như lúc bắt đầu bài học demo này.

1. Hãy quay lại bảng điều khiển EC2 và đi tới mục **instances (các thực thể)**, và chúng ta cần **chấm dứt (terminate) một trong các thực thể này một cách thủ công**.
2. Chúng ta cần chấm dứt thực thể **a4l-manual-wordpress**, bởi vì thực thể này không được tạo bởi CloudFormation nên chúng ta phải tự tay chấm dứt nó. Vì vậy, nhấp chuột phải vào **a4l-manual-wordpress** và chọn **terminate instance**, sau đó xác nhận. Bạn sẽ cần đợi quá trình này hoàn tất, đợi thực thể chuyển hẳn sang trạng thái **terminated (đã chấm dứt)** trước khi có thể tiếp tục bước tiếp theo. Hãy tạm dừng video, đợi thực thể chuyển sang trạng thái terminated rồi tiếp tục.

Bây giờ thực thể đó đã được chấm dứt, chúng ta có thể chuyển quay lại **bảng điều khiển CloudFormation**, và chúng ta cần **xóa (delete) cả hai stack này**.

- Hãy chọn **bootstrap-CFN**, rồi nhấp vào **delete**, sau đó xác nhận việc xóa đó.
- Đồng thời, bạn có thể chọn **bootstrap**, nhấp vào **delete** một lần nữa, xác nhận việc xóa.

Và sau khi cả hai stack đó đã được xóa, **tài khoản sẽ trở về trạng thái y hệt như lúc bắt đầu bài học demo này**.

Tôi hy vọng bài demo này hữu ích với các bạn. Bạn đã có được một số kinh nghiệm về việc bootstrap các thực thể EC2 cả bằng cách thủ công lẫn sử dụng CloudFormation. Đến đây thì mọi thứ đã hoàn thành, vì vậy hãy kết thúc video này, và khi bạn đã sẵn sàng, tôi rất mong được gặp lại bạn trong bài học tiếp theo.

--- 

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- User Data trong CloudFormation (Base64)
- One-click deployment với template bootstrap-CFN
- So sánh bootstrap thủ công vs CloudFormation
- Custom login banner & WordPress install
- Account cleanup: Terminate instance + Delete stacks

**Notes (Chi tiết):**

- **User Data** được nhúng vào CloudFormation template bằng hàm mã hóa **Base64**.
- Template tự động launch EC2 + chạy script User Data giống hệt cách thủ công.
- Sau khi stack **Create Complete** → kiểm tra Public IP → truy cập WordPress và Instance Connect để xem banner tùy chỉnh.
- **Cleanup**: Terminate instance thủ công (**a4l-manual-wordpress**) trước, sau đó xóa cả 2 CloudFormation stack (**bootstrap-CFN** và **bootstrap**).

**Summary (Tóm tắt ngắn):** Bài học hướng dẫn cách **bootstrap EC2 instance** bằng **User Data** theo hai cách: **thủ công** và **qua CloudFormation template**. Sử dụng Base64 để nhúng script tự động cài WordPress và custom banner. Kết thúc bằng quy trình dọn dẹp tài khoản để trả về trạng thái ban đầu.