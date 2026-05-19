**Instance Store** cung cấp **lưu trữ tạm thời ở mức block** cho instance của bạn. Loại lưu trữ này nằm trên các **đĩa vật lý gắn trực tiếp** vào máy chủ (host computer) đang chạy instance.

**Instance Store** rất lý tưởng cho việc lưu trữ **tạm thời** những dữ liệu thay đổi thường xuyên, chẳng hạn như **buffers, caches, scratch data** và các nội dung tạm khác, hoặc cho dữ liệu được **nhân bản** trên nhiều instance (ví dụ: cụm web server cân bằng tải).

**Instance Store** bao gồm một hoặc nhiều **instance store volumes** được expose dưới dạng **block devices**. Kích thước cũng như **số lượng volume** có sẵn sẽ **khác nhau tùy theo instance type**.

Các thiết bị ảo cho instance store volumes được đặt tên là **ephemeral[0-23]**.
- Instance type hỗ trợ 1 volume sẽ có **ephemeral0**.
- Instance type hỗ trợ 2 volumes sẽ có **ephemeral0** và **ephemeral1**, và cứ thế.

**Lần này là Instance Store Volumes.**
Rất quan trọng đối với **tất cả các kỳ thi AWS** và sử dụng thực tế rằng bạn phải hiểu rõ **ưu và nhược điểm** của loại lưu trữ này.

Nó có thể **tiết kiệm tiền**, **tăng hiệu suất**, hoặc cũng có thể gây ra **những rắc rối lớn**, vì vậy bạn cần nắm rõ tất cả các yếu tố liên quan.

Hãy đi thẳng vào nội dung vì chúng ta có rất nhiều thứ cần tìm hiểu.

**Instance Store Volumes** cung cấp các **block storage devices** (thiết bị lưu trữ khối), tức là các raw volume có thể được gắn vào một instance, được hệ điều hành trên instance đó nhận diện và sử dụng làm nền tảng để tạo file system, sau đó các ứng dụng sẽ sử dụng.

Cho đến nay chúng **giống EBS**, chỉ khác là **local** (địa phương), thay vì được trình bày qua mạng.

**Những volume này được kết nối vật lý trực tiếp** vào **một máy chủ EC2**, và đây là điểm **rất quan trọng**.

Mỗi máy chủ EC2 có **instance store volumes riêng** của nó và chúng **chỉ giới hạn trong máy chủ đó**

Các instance chạy trên máy chủ đó có thể truy cập những volume này. Vì chúng được **gắn cục bộ (locally attached)**, nên chúng mang lại **hiệu suất lưu trữ cao nhất hiện có trong AWS**, cao hơn rất nhiều so với những gì EBS có thể cung cấp.

(Mình sẽ giải thích thêm tại sao điều này quan trọng ngay sau đây). Chúng cũng **được bao gồm trong giá của instance** mà nó đi kèm.

Các **instance type** khác nhau sẽ đi kèm với **số lượng và cấu hình instance store volumes** khác nhau. Với bất kỳ instance nào có instance store volumes, chúng đã **được tính vào giá instance**, nên hoặc **dùng hoặc mất** (use it or lose it).

**Một điều cực kỳ quan trọng** về instance store volumes là **bạn chỉ có thể gắn chúng lúc launch instance**. Khác với EBS, **bạn không thể gắn sau khi instance đã chạy**.

Bạn chỉ có thể làm điều này **tại thời điểm launch**.

Tùy thuộc vào **instance type**, bạn sẽ được cấp một số lượng instance store volumes nhất định. Bạn có thể chọn dùng hoặc không dùng. Nhưng nếu không dùng thì **sau này không thể điều chỉnh** được nữa.

![[Pasted image 20260518113034.png]]

**Đây là cách kiến trúc Instance Store trông như thế nào.** Mỗi instance có thể có một **tập hợp các volume**, được hỗ trợ bởi các thiết bị vật lý trên **EC2 host** mà instance đang chạy.

Trong ví dụ này, **host A** có ba thiết bị vật lý và chúng được trình bày thành ba **instance store volumes**. **Host B** cũng có đúng ba thiết bị vật lý như vậy. Trong thực tế, các EC2 host sẽ có nhiều thiết bị hơn, nhưng đây chỉ là sơ đồ đơn giản hóa.

Trên **host A**, instance 1 đang dùng một volume, instance 2 đang dùng hai volume còn lại. Các volume được đặt tên **ephemeral0, ephemeral1, ephemeral2**. Kiến trúc tương tự cũng có trên **host B**, nhưng instance 3 là instance duy nhất đang chạy và nó đang sử dụng **ephemeral1** và **ephemeral2**.

**Những volume này là ephemeral volumes** — chúng là **lưu trữ tạm thời**. Là solutions architect, developer hay engineer, bạn cần coi chúng như **dữ liệu tạm thời**.

Nếu instance 1 lưu một bức ảnh mèo lên **ephemeral volume 0** trên **host A**, sau đó instance bị migrate sang **host B**, thì nó vẫn có thể truy cập **ephemeral0**, nhưng đó sẽ là một **physical volume mới hoàn toàn trống**. Do đó, **toàn bộ dữ liệu cũ sẽ bị mất**.

Instances có thể di chuyển giữa các host vì nhiều lý do: bị stop & start, host đang bảo trì, hoặc thay đổi instance type… Khi di chuyển, chúng sẽ được cấp **ephemeral volumes mới và trống**, dữ liệu trên volume cũ sẽ **bị xóa sạch** (wiped).

**Đây là rủi ro quan trọng cần ghi nhớ.**

Một nguy cơ khác là **hỏng phần cứng**. Nếu một volume vật lý hỏng (ví dụ ephemeral0 trên host A), thì instance đang dùng nó sẽ mất toàn bộ dữ liệu trên volume đó.

**Tóm lại:** Đây là **ephemeral volumes** — hãy đối xử với chúng như **dữ liệu tạm thời**. Chúng **không nên dùng** cho bất kỳ thứ gì yêu cầu **persistence** (bền vững).

![[Pasted image 20260518113607.png]]

**Kích thước và số lượng instance store volumes** có sẵn cho một instance **phụ thuộc vào instance type** và kích thước instance. Một số instance type **không hỗ trợ** instance store volumes. Các instance type lớn hơn thường được cấp **nhiều volume hơn**.

**Một trong những lợi ích chính** của instance store volumes là **hiệu suất**. Bạn có thể đạt được mức **throughput và IOPS cao hơn rất nhiều** so với EBS.

Ví dụ: Với **D3 instance** (storage optimized), bạn có thể đạt **4.6 GB/giây throughput** và dung lượng lưu trữ lớn bằng ổ cứng truyền thống — rất đáng giá cho dữ liệu lớn.

**I3 series** (cũng là storage optimized) sử dụng **NVMe SSDs**, cung cấp tới **16 GB/giây throughput** và **hàng triệu IOPS** (2 triệu read IOPS & 1.6 triệu write IOPS khi cấu hình tối ưu).

Đây là mức **cao hơn rất nhiều** so với ngay cả các EBS volume hiệu năng cao nhất.

**Tóm tắt ý quan trọng:**

- **Instance Store Volumes** là **ephemeral** (tạm thời) → **dữ liệu sẽ mất** khi instance di chuyển host hoặc host hỏng.
- Chỉ gắn được **tại lúc launch**.
- **Hiệu suất cực cao** (throughput & IOPS) — cao hơn hẳn EBS.
- Phù hợp cho **cache, temporary data, scratch data**, hoặc dữ liệu có thể **replicate**.

**Trước khi kết thúc bài học này**, mình xin đưa ra một số **exam powerups** (kiến thức trọng tâm cho kỳ thi).

- **Instance Store Volumes** là **local** (cục bộ) trên một **EC2 host**.
- Nếu instance **di chuyển giữa các host**, bạn sẽ **mất quyền truy cập** vào dữ liệu trên những volume đó.
- Bạn **chỉ có thể thêm Instance Store Volumes** vào instance **tại thời điểm launch**.
- Nếu không thêm lúc launch, **bạn không thể quay lại sau để thêm** chúng.
- Dữ liệu trên **Instance Store Volumes** sẽ **bị mất** nếu instance di chuyển giữa các host, thay đổi kích thước (resize), hoặc gặp sự cố host failure / volume hardware failure.

**Đổi lại**, Instance Store Volumes mang lại **hiệu suất rất cao** — đây là mức **hiệu suất lưu trữ cao nhất** bạn có thể đạt được trong AWS.

Bạn chỉ cần **chấp nhận tất cả các hạn chế** về rủi ro mất dữ liệu, tính tạm thời, và việc không thể sống sót qua restart, move hay resize.

**Về bản chất**, đây là một **performance trade-off**: bạn nhận được **lưu trữ nhanh hơn rất nhiều** miễn là bạn chịu được tất cả những hạn chế đó.

**Với Instance Store Volumes**, bạn **đã trả tiền cho chúng rồi** (vì chúng nằm trong giá của instance). Do đó, khi provisioning instance có Instance Store Volumes, **không có lợi ích gì khi không sử dụng chúng**.

Bạn có thể quyết định **không mount** chúng trong hệ điều hành, nhưng **không thể thêm về mặt vật lý** sau này.

**Nhắc lại một lần nữa** (mình sẽ lặp lại nhiều lần trong phần này của khóa học): **Instance Store Volumes là temporary** (tạm thời).

**Bạn không được dùng** chúng cho bất kỳ dữ liệu nào mà bạn **phụ thuộc** hoặc dữ liệu **không thể tái tạo lại được**.

**Hãy luôn ghi nhớ điều này.**

Nó mang lại **hiệu suất tuyệt vời**, nhưng **không phải để lưu trữ dữ liệu bền vững** (persistent storage).

**Và đó là toàn bộ lý thuyết** mình muốn truyền tải.

Đó chính là **kiến trúc** cùng với một số **trade-off về hiệu suất** và **lợi ích** mà bạn nhận được khi dùng **Instance Store Volumes**.

Bạn có thể hoàn thành video này đi, và khi sẵn sàng thì gặp mình ở video tiếp theo — đó sẽ là **so sánh kiến trúc giữa EBS và Instance Store**, giúp bạn dễ dàng chọn loại nào phù hợp trong tình huống thi.