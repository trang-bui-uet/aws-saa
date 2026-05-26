In this lesson I talk about EBS Volume encryption. We look at the architecture, the flow of the encryption process and security of the data encryption keys.

Bây giờ, các **EBS volumes**, như bạn đã biết, là các **thiết bị lưu trữ khối (block storage)** được trình bày qua mạng. Những volume này được lưu trữ theo cách **resilient** và **highly available** bên trong một **Availability Zone**, nhưng ở mức **infrastructure**, chúng được lưu trên **một hoặc nhiều thiết bị lưu trữ vật lý**.

**Mặc định không có mã hóa nào được áp dụng**, nên dữ liệu được ghi xuống đĩa **chính xác** như hệ điều hành viết ra. Nếu bạn ghi một bức ảnh mèo vào ổ đĩa hoặc mount point trong instance, thì **nội dung plaintext** của bức ảnh đó sẽ được ghi thẳng xuống **các raw disks**.

Điều này **thêm rủi ro** và tạo ra **potential physical attack vector** cho hoạt động kinh doanh của bạn.

**EBS encryption** giúp giảm thiểu rủi ro này bằng cách cung cấp **encryption at rest** cho volumes và cả **snapshots**.

Về mặt kiến trúc thì **EBS encryption không phức tạp lắm**, đặc biệt khi bạn đã hiểu về **KMS** (Key Management Service).

Without encryption, kiến trúc trông sẽ **cơ bản** như sau. Chúng ta có một **EC2 host** đang chạy trong một **Availability Zone** cụ thể, và trên host đó là một **EC2 instance** đang sử dụng một **EBS volume** làm **boot volume**.

Không có encryption, instance tạo ra dữ liệu và dữ liệu này được lưu trữ trên volume dưới dạng **plain text**.

Vì vậy, nếu bạn lưu bất kỳ bức ảnh mèo hay gà nào trên ổ đĩa hoặc mount point bên trong EC2 instance, thì **mặc định plaintext** đó sẽ được lưu **at rest** trên các **EBS volumes**.

Bây giờ, khi bạn tạo một **encrypted EBS volume** ngay từ đầu, **EBS** sẽ sử dụng **KMS** và một **KMS key**. Key này có thể là **AWS managed key mặc định** (gọi là aws/ebs) hoặc **customer managed KMS key** do bạn tự tạo và quản lý.

Key này được EBS sử dụng khi tạo encrypted volume, cụ thể là để **tạo ra một encrypted data encryption key (DEK)** thông qua lệnh **GenerateDataKey** mà **không có plain text API call**.

Bạn chỉ nhận được **encrypted data encryption key** và key này được lưu cùng với volume trên **raw storage**.

DEK này **chỉ có thể được giải mã** bằng **KMS**, với điều kiện thực thể thực hiện có đủ quyền (permissions) để giải mã data encryption key bằng **corresponding KMS key**.

Nhớ rằng, ban đầu khi tạo volume, nó **hoàn toàn trống rỗng**, chỉ là một vùng không gian được cấp phát, nên **không có gì để mã hóa** cả.

Khi volume được sử dụng lần đầu tiên — hoặc khi bạn **mount** nó vào EC2 instance, hoặc khi instance được launch — thì **EBS** sẽ yêu cầu **KMS** giải mã data encryption key (DEK) dành riêng cho volume đó.

Key đã giải mã sau đó được **load vào bộ nhớ (memory)** của **EC2 host** đang sử dụng volume.

**Key này chỉ tồn tại ở dạng decrypted** trong memory của EC2 host đang dùng volume.

Từ đó, host sẽ sử dụng key này để **mã hóa và giải mã dữ liệu** giữa instance và EBS volume, cụ thể là raw storage mà volume đang được lưu trên đó.

Điều này có nghĩa là **toàn bộ dữ liệu** được ghi xuống raw storage đều là **ciphertext** (dữ liệu đã mã hóa) — tức là **encrypted at rest**.

**Dữ liệu chỉ tồn tại ở dạng unencrypted** (plaintext) **bên trong memory của EC2 host**. Những gì nằm trên raw storage chính là **phiên bản mã hóa** của mọi dữ liệu mà hệ điều hành của instance ghi ra.

Khi **EC2 instance** di chuyển từ host này sang host khác, **decrypted key** sẽ bị **discard** (xóa bỏ), chỉ còn lại **phiên bản encrypted** trên đĩa.

Để instance có thể sử dụng volume trở lại, **encrypted data encryption key** cần được **KMS giải mã** lần nữa và load vào **EC2 host mới**.

Nếu tạo **snapshot** của một encrypted volume, **cùng một data encryption key** sẽ được sử dụng cho snapshot đó, nghĩa là **snapshot cũng được mã hóa**.

Bất kỳ volume nào được tạo từ snapshot đó cũng sẽ **tự động encrypted** bằng **cùng data encryption key**, nên chúng cũng được mã hóa.

**Đó là toàn bộ kiến trúc** của EBS encryption.

Nó **không tốn thêm chi phí** gì cả, nên đây là một trong những tính năng **bạn nên bật mặc định** luôn.

Bây giờ mình đã giải thích kiến trúc một cách khá chi tiết, và giờ mình muốn đi qua **một số điểm tóm tắt quan trọng** sẽ giúp bạn rất nhiều trong kỳ thi.

Kỳ thi thường hay đưa ra **những câu hỏi khó (curve ball)** xoay quanh chủ đề encryption, nên mình sẽ đưa ra một số **hints** để bạn biết cách diễn giải và trả lời chúng.

**AWS accounts có thể được cấu hình** để **mã hóa EBS volumes by default**. Bạn có thể thiết lập **default KMS key** dùng cho việc mã hóa này, hoặc mỗi lần tạo volume bạn có thể **chọn KMS key thủ công**.

**KMS key không được dùng để trực tiếp mã hóa hoặc giải mã** volume, mà nó được dùng để **tạo ra một data encryption key (DEK) riêng biệt cho từng volume**.

Nếu bạn tạo **snapshot** hoặc tạo volume mới từ snapshot đó, thì **cùng một data encryption key** sẽ được sử dụng.

Tuy nhiên, **mỗi lần bạn tạo một volume hoàn toàn mới từ đầu (from scratch)**, EBS sẽ tạo ra **một data encryption key hoàn toàn khác** (unique per volume).

Nhấn mạnh lại một lần nữa vì điều này **rất quan trọng**: **Data encryption key** chỉ dùng cho **một volume cụ thể**, và cho **tất cả các snapshot** được tạo từ volume đó (những snapshot này cũng được mã hóa), cũng như **tất cả các volume mới** được tạo từ snapshot đó.

Tôi sẽ nhấn mạnh lại lần nữa. Tôi biết bạn đang bắt đầu chán khi nghe tôi nói điều này. Mỗi khi bạn tạo một EBS volume mới từ đầu, nó sẽ sử dụng một **data encryption key (DEK) riêng biệt**. Nếu bạn tạo thêm một volume mới khác từ đầu, nó sẽ dùng một **DEK khác**. Nhưng nếu bạn tạo snapshot từ một **encrypted volume** đã tồn tại, snapshot đó sẽ sử dụng **cùng một DEK**, và nếu bạn tiếp tục tạo thêm các EBS volume khác từ snapshot đó, chúng cũng sẽ sử dụng **cùng DEK đó**. Hiện tại **không có cách nào để gỡ encryption khỏi một volume hoặc snapshot**. Một khi đã được mã hóa thì nó sẽ luôn được mã hóa. Có một vài cách workaround thủ công bằng cách clone dữ liệu thực tế bên trong hệ điều hành sang một **unencrypted volume** khác, nhưng đây **không phải là tính năng được hỗ trợ trực tiếp từ AWS Console, CLI hay APIs**.

Hệ điều hành chỉ nhìn thấy **plain text**, nên đây là cách duy nhất để bạn có thể truy cập dữ liệu dạng plain text và clone nó sang một **unencrypted volume** khác. Và đây là một điểm cực kỳ quan trọng cần hiểu: **bản thân hệ điều hành không hề biết có encryption đang diễn ra**. Đối với operating system, nó chỉ thấy **plain text**, bởi vì encryption đang xảy ra **giữa EC2 host và volume**, cụ thể là giữa **EC2 host và hệ thống EBS**, sử dụng **AES-256**. Nếu bạn gặp trường hợp cần chính operating system thực hiện việc encryption, thì đó là thứ bạn phải tự cấu hình **bên trong operating system**. Nếu bạn muốn **tự giữ keys**, thay vì để **EC2, EBS và KMS** quản lý, thì bạn cần cấu hình **volume encryption bên trong operating system**. Điều này thường được gọi là **software disk encryption**, điều này đơn giản có nghĩa là **operating system tự thực hiện encryption và tự lưu giữ keys**. Bạn có thể sử dụng **software disk encryption** bên trong operating system đồng thời với **EBS encryption**, nhưng điều này thật ra không hợp lý trong hầu hết các use case, dù hoàn toàn có thể làm được. Tuy nhiên, **EBS encryption rất hiệu quả**: bạn không cần phải lo quản lý keys, nó **không tốn thêm chi phí**, và cũng **không gây performance loss** khi sử dụng.