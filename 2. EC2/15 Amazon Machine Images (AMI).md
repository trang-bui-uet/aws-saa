**Phần Gợi nhớ (Cues / Keywords)**

- AMI là gì?
- AMI dùng để làm gì trong EC2?
- Các nguồn AMI nào có sẵn trong AWS?
- AMI có tính chất gì về Region?
- Quyền truy cập của AMI mặc định là gì? Có những tùy chọn nào?
- Vòng đời của AMI gồm những giai đoạn nào?
- Khi tạo AMI từ một instance đang chạy thì sẽ xảy ra gì?
- EBS Snapshots đóng vai trò gì khi tạo AMI?
- Block Device Mapping là gì và có tác dụng gì?
- AMI Baking nghĩa là gì?
- AMI có thể chỉnh sửa trực tiếp được không?
- Nếu muốn thay đổi cấu hình AMI thì phải làm sao?
- Có thể copy AMI sang Region khác không?
- Chi phí của AMI được tính như thế nào?
- AMI chứa những thông tin gì chính?

Trong bài học này, tôi muốn đi sâu hơn một chút về **Amazon Machine Images**, hay còn gọi là **AMI**.

Nhiều người trong cộng đồng AWS có thể khiến bạn nghĩ rằng có một số cuộc tranh luận hoặc bất đồng về cách phát âm từ "**AMI**", với một số người phát âm nó hơi khác một chút mà tôi sẽ không lặp lại ở đây. Theo quan điểm của tôi, không có gì phải tranh luận cả, những người đó chỉ đang bị nhầm lẫn mà thôi.

**AMI** là các **hình ảnh (images)** của **EC2**. Chúng là một cách để bạn có thể tạo ra một **cấu hình mẫu (template)** cho một **thực thể (instance)**, và sau đó sử dụng mẫu đó để tạo ra nhiều thực thể khác từ cấu hình này. Thực tế, **AMI** được sử dụng khi bạn khởi chạy các thực thể **EC2**. Bạn đang khởi chạy các thực thể đó bằng cách sử dụng các **AMI** do AWS cung cấp. Tuy nhiên, bạn cũng có thể tự tạo **AMI** của riêng mình, và đó là điều tôi muốn tập trung vào trong bài học này: ý nghĩa của việc đó, điều gì xảy ra khi bạn tự tạo **AMI** và cách thực hiện nó một cách hiệu quả.

### Một vài điểm mấu chốt trước khi tôi nói về vòng đời và quy trình xung quanh AMI:

- **AMI** có thể được sử dụng để khởi chạy các thực thể **EC2**.
- Khi bạn chọn sử dụng Amazon Linux 2, bạn thực chất đang sử dụng một **AMI Amazon Linux 2** để khởi chạy thực thể đó.
- Các **AMI** mà bạn thường dùng để khởi chạy thực thể có thể do **AWS cung cấp** hoặc do **cộng đồng cung cấp**.
- Một số nhà cung cấp tự tạo bản phân phối Linux riêng của họ, họ sẽ sản xuất các **AMI cộng đồng** để phục vụ cho việc khởi chạy bản phân phối Linux đó bên trong **EC2**. Ví dụ: Red Hat, CentOS, Ubuntu đều có sẵn bên trong AWS.
- Bạn cũng có thể khởi chạy các thực thể từ các **AMI do Marketplace cung cấp**, và những **AMI** này có thể bao gồm cả phần mềm thương mại (có chi phí bổ sung).
- **AMI** mang tính chất **vùng miền (regional)**: Mỗi vùng AWS sẽ có các **AMI** khác nhau cho cùng một thứ. Một **AMI** chỉ có thể được sử dụng trong chính vùng mà nó tồn tại.
- Mỗi **AMI** có một **ID duy nhất** với định dạng: ami- theo sau là một chuỗi ngẫu nhiên gồm các chữ số và chữ cái.
- **AMI** cũng kiểm tra và kiểm soát **quyền truy cập (permissions)**:
    - Mặc định: Chỉ tài khoản của bạn mới có thể sử dụng (private).
    - Có thể đặt chế độ **công khai (public)**.
    - Hoặc chia sẻ cho các tài khoản AWS cụ thể khác.

Quy trình mà bạn đã trải nghiệm cho đến nay là **lấy một AMI và sử dụng nó để tạo ra một thực thể**. Nhưng bạn cũng có thể làm ngược lại: **tạo một AMI từ một thực thể EC2 đang tồn tại** để ghi lại cấu hình hiện tại của thực thể đó.

### Vòng đời của một AMI (chu trình 4 giai đoạn)

![[Pasted image 20260523234023.png]]

Tôi muốn bạn nghĩ về vòng đời của một **AMI** như một chu trình gồm bốn giai đoạn:

1. **Khởi chạy (Launch)**
2. **Cấu hình (Configure)**
3. **Tạo hình ảnh (Create Image)**
4. **Khởi chạy lại (Launch again)**

Việc lặp lại giai đoạn khởi chạy lần thứ hai là có chủ đích.

Phần lớn những người tương tác với AWS chỉ trải nghiệm giai đoạn đầu tiên (sử dụng **AMI** để khởi chạy thực thể **EC2**).

### Kiến trúc chi tiết khi tạo AMI

- Các ổ đĩa **EBS** được gắn vào thực thể **EC2** bằng **ID thiết bị khối (block device IDs)**.
    - Ổ đĩa khởi động (boot volume) thường là /dev/xvda.
    - Ổ đĩa bổ sung ví dụ: /dev/xvdf.

Khi bạn tạo **AMI** từ một thực thể đang tồn tại:

- Bạn áp dụng tùy chỉnh (cài ứng dụng, cấu hình hệ thống, gắn thêm ổ đĩa…).
- Khi tạo **AMI**, AWS sẽ:
    - Tạo **EBS snapshots** từ tất cả các ổ đĩa EBS được gắn vào thực thể.
    - **Snapshots** là bản sao nhanh (incremental), bản đầu tiên là full copy.
- **AMI** chứa:
    - **Quyền truy cập** (ai được dùng AMI).
    - **Block Device Mapping** (sơ đồ thiết bị khối): Bảng liên kết **Snapshot ID** với **Device ID** gốc.

**Quan trọng**: Bản thân **AMI** không chứa dữ liệu thực tế. Nó chỉ là một **vùng chứa logic** tham chiếu đến các **EBS snapshots** và block device mapping.

Khi khởi chạy thực thể mới từ **AMI**:

- Các snapshot được dùng để tạo ra **EBS volumes mới**.
- Volumes được gắn đúng với **Device ID** gốc → thực thể mới có đúng cấu hình ổ đĩa và dữ liệu như thực thể nguồn.

### Điểm lưu ý quan trọng cho kỳ thi (Solutions Architect Associate)

- **AMI** nằm trong **một vùng duy nhất (one region)**: Chỉ sử dụng được trong vùng đã tạo, nhưng có thể triển khai vào bất kỳ AZ nào trong vùng đó.
- **“Nướng” AMI (AMI Baking)**: Lấy một thực thể, cài đặt toàn bộ phần mềm + cấu hình, sau đó tạo **AMI** từ đó để “dập khuôn” hàng loạt thực thể giống hệt.
- **AMI không thể chỉnh sửa (cannot be edited)**: Muốn thay đổi → phải launch instance từ AMI cũ → chỉnh sửa → tạo **AMI mới**.
- **AMI có thể được sao chép (can be copied)** giữa các vùng AWS.
- **Quyền truy cập mặc định**: Private (chỉ tài khoản của bạn).
- **Vấn đề chi phí (Billing)**: Bạn bị tính phí cho **dung lượng lưu trữ của các EBS snapshots** mà AMI tham chiếu đến (không tính theo kích thước ổ đĩa được cấp, mà theo lượng dữ liệu thực tế được sử dụng).

Tôi nghĩ phần lý thuyết như vậy là đã đủ. Trong bài học tiếp theo (bài thực hành), bạn sẽ có trải nghiệm thực tế về việc khởi chạy một thực thể, thực hiện cấu hình tùy chỉnh, tạo **AMI** của riêng mình và dùng nó để tạo ra các thực thể mới.

Hy vọng bài học lý thuyết này đã giúp bạn nắm rõ về mặt kiến trúc để áp dụng tốt vào bài thực hành cũng như chuẩn bị cho kỳ thi.

**Hẹn gặp lại bạn ở bài học tiếp theo!**

**Phần Tóm tắt (Summary)** Amazon Machine Image (AMI) là bản mẫu để khởi chạy EC2 instances. Khi tạo AMI, hệ thống sẽ tạo EBS Snapshots và Block Device Mapping để lưu cấu hình ổ đĩa. AMI có tính regional, không thể chỉnh sửa trực tiếp, phù hợp cho **AMI Baking** – chuẩn hóa môi trường và triển khai nhanh hàng loạt. Chi phí chủ yếu đến từ việc lưu trữ snapshots. Đây là công cụ quan trọng giúp tiết kiệm thời gian và đảm bảo tính nhất quán trong hạ tầng AWS.