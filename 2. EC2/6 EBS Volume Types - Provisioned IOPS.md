![[Pasted image 20260518103810.png]]
Provisioned IOPS SSD — Cung cấp hiệu suất cao cho các workload quan trọng then chốt, có độ trễ thấp hoặc thông lượng cao
Tên **Provisioned IOPS SSD** = **Ổ cứng SSD mà bạn được phép cấp phát (provision) trước số IOPS cụ thể.**
Nói một cách nghiêm ngặt thì, hiện nay có ba loại Provisioned IOPS SSD: hai loại đã ở trạng thái general release là io1 và thế hệ kế nhiệm của nó là io2, và một loại đang ở giai đoạn preview là io2 Block Express. Hiện tại cả ba loại đều cung cấp các đặc tính hiệu suất hơi khác nhau và có giá khác nhau. Nhưng yếu tố chung quan trọng là IOPS có thể được cấu hình độc lập với dung lượng của volume, và chúng được thiết kế dành cho các tình huống hiệu suất cực cao, trong đó độ trễ thấp và tính nhất quán của độ trễ thấp là hai đặc tính quan trọng.

Nói một cách nghiêm ngặt thì, hiện nay có **ba loại Provisioned IOPS SSD**: hai loại đã ở trạng thái **general release** là **io1** và thế hệ kế nhiệm của nó là **io2**, và một loại đang ở giai đoạn **preview** là **io2 Block Express**. Hiện tại cả ba loại đều cung cấp các đặc tính hiệu suất hơi khác nhau và có **giá khác nhau**. Nhưng **yếu tố chung quan trọng** là **IOPS** có thể được **cấu hình độc lập** với dung lượng của volume, và chúng được thiết kế dành cho các tình huống **hiệu suất cực cao**, trong đó **độ trễ thấp** và **tính nhất quán của độ trễ thấp** là hai đặc tính quan trọng.

Với **io1** và **io2**, bạn có thể đạt **tối đa 64.000 IOPS** cho mỗi volume, và con số này **gấp 4 lần** mức tối đa của GP2 và GP3. Đồng thời với io1 và io2, bạn có thể đạt **1.000 MB/giây** throughput. Mức này **bằng** GP3 và **cao hơn đáng kể** so với GP2.

Hiện nay **io2 Block Express** đưa mọi thứ lên một tầm cao mới. Với **Block Express**, bạn có thể đạt tới **256.000 IOPS** mỗi volume và **4.000 MB/giây** throughput mỗi volume.

Về dung lượng volume có thể sử dụng với Provisioned IOPS SSD: io1 và io2 cho phép từ **4 GB đến 16 TB**, còn với **io2 Block Express** bạn có thể dùng volume lớn hơn, lên tới **64 TB**.

Như đã đề cập, với các volume này bạn có thể **cấp phát giá trị hiệu suất IOPS hoàn toàn độc lập** với dung lượng của volume. Hiện nay điều này rất hữu ích khi bạn cần **hiệu suất cực cao** cho các volume nhỏ, hoặc khi bạn chỉ cần **hiệu suất cực cao** nói chung.

Tuy nhiên, **có giới hạn tỷ lệ giữa dung lượng và hiệu suất**. Với **io1**, tối đa là **50 IOPS mỗi GB** dung lượng. Con số này cao hơn mức **3 IOPS/GB** của GP2. Với **io2**, con số tăng lên **500 IOPS mỗi GB** dung lượng volume. Còn với **Block Express**, đạt tới **1.000 IOPS mỗi GB** dung lượng volume.

**Lưu ý**: Đây đều là **mức tối đa**. Bạn sẽ phải **trả tiền cho cả dung lượng volume lẫn IOPS đã provisioned** mà bạn cần.

Vì đây là các loại volume có **mức hiệu suất cực cao**, nên còn có **một hạn chế khác** bạn cần biết, đó là **hiệu suất theo từng instance** (per instance performance). Có một **giới hạn tối đa** về hiệu suất giữa dịch vụ EBS và **một EC2 instance** duy nhất.

Giới hạn này bị ảnh hưởng bởi một số yếu tố:
- **Loại volume** (các loại volume khác nhau có mức hiệu suất tối đa theo instance khác nhau)
- **Loại instance**
- Và cuối cùng là **kích thước của instance**.

Bạn sẽ thấy **chỉ những instance hiện đại nhất và lớn nhất** mới hỗ trợ **mức hiệu suất cao nhất**.

Và các giới hạn **per instance maximum** này cũng cao hơn những gì **một volume đơn lẻ** có thể cung cấp. Do đó bạn sẽ **cần nhiều volume** cùng hoạt động để đạt được giới hạn per instance này.

Với **io1 volumes**, bạn có thể đạt tối đa **260.000 IOPS** per instance và **7.500 MB/giây** throughput. Nghĩa là bạn cần **hơn 4 volume** hoạt động ở mức tối đa để đạt được giới hạn per instance.

Thật thú vị là **io2** lại thấp hơn một chút, ở mức **160.000 IOPS** và **4.750 MB/giây** cho toàn bộ instance. Đó là vì AWS đã tách thế hệ volume mới này ra.

Họ đã thêm **Block Express**, loại này có thể đạt **260.000 IOPS** và **7.500 MB/giây** cho giới hạn per instance.

Vì vậy **rất quan trọng** là bạn phải hiểu rằng bạn **cần nhiều volume hoạt động cùng nhau**, và coi đây là **giới hạn hiệu suất (performance cap)** cho **một EC2 instance** riêng lẻ.

Đây là **mức tối đa** cho từng loại volume, nhưng bạn cũng cần xem xét **giới hạn theo loại và kích thước instance**. Tất cả những yếu tố này phải **phù hợp với nhau** để đạt được hiệu suất tối đa.

Hãy **ghi nhớ các con số này**. Không cần nhớ chính xác tuyệt đối, nhưng cần **có cảm nhận** về mức hiệu suất mà GP2 hay GP3 có thể đạt được, và io1, io2, io2 Block Express sẽ giúp bạn rất nhiều trong tình huống thực tế cũng như trong kỳ thi.

**Instant store volumes** (sẽ được đề cập ở phần khác) có thể đạt hiệu suất còn cao hơn nữa, nhưng chúng **không bền vững (not persistent)** — đây là hạn chế nghiêm trọng.

Để so sánh, giới hạn per instance của **GP2 và GP3** là **260.000 IOPS** và **7.000 MB/giây**.

Tóm lại, bạn sẽ dùng **Provisioned IOPS SSD** cho những workload cần **độ trễ cực thấp (sub-millisecond latency)**, **độ trễ ổn định**, và **hiệu suất cao hơn**. Một use case phổ biến là khi volume nhỏ nhưng cần **siêu cao hiệu suất** — và chỉ **io1, io2, io2 Block Express** mới làm được điều đó.