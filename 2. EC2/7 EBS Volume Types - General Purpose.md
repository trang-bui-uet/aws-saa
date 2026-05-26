#### General Purpose
General Purpose SSD: Cân bằng giữa giá thành và hiệu suất, chúng tôi khuyến nghị sử dụng loại volume này cho hầu hết các workload.

Hiện nay GP2 là loại lưu trữ SSD mục đích chung (General Purpose SSD) mặc định được cung cấp bởi EBS.

GP3 là loại lưu trữ mới hơn, và tôi muốn bổ sung vào vì tôi kỳ vọng nó sẽ xuất hiện trong tất cả các bài thi rất sớm.

Lưu trữ SSD mục đích chung được cung cấp bởi EBS là một bước thay đổi cuộc chơi khi nó mới ra mắt. Đây là loại lưu trữ có hiệu suất cao với mức giá khá thấp.

Hiện nay **GP2** là phiên bản đầu tiên, và đây là nội dung mình sẽ nói trước tiên vì nó có kiến trúc đơn giản nhưng ban đầu khá khó hiểu. Vì vậy mình muốn giải quyết nó ngay từ đầu, bởi nó sẽ giúp bạn hiểu rõ hơn về **các loại lưu trữ khác nhau**.

Khi bạn lần đầu tạo một **GP2 volume**, nó có thể nhỏ chỉ **1 GB** hoặc lớn tới **16 TB**. Khi tạo, volume sẽ được cấp sẵn một lượng **IO credit**.

Hãy nghĩ IO credit giống như một **bucket**. Một **IO** là một thao tác Input/Output. Một **IO credit** tương đương với **16 KB** dữ liệu. Do đó, **1 IOPS** nghĩa là bạn có thể xử lý **16 KB** dữ liệu trong **1 giây**.

**Ví dụ**: Khi chuyển một file **160 KB**, nó sẽ chiếm **10 IO blocks** (10 × 16 KB).
Và nếu bạn làm tất cả điều đó trong **1 giây**, thì đó là **10 credits** trong 1 giây, tức là **10 IOPS**.
Khi bạn không sử dụng volume nhiều, bạn cũng không dùng nhiều **IOPS** và không tiêu hao nhiều **IO credits**.
Nhưng trong những lúc tải đĩa cao (high disk load), bạn sẽ đẩy volume hoạt động mạnh, và vì vậy nó sẽ **tiêu thụ nhiều credits hơn**.
Ví dụ: trong lúc **system boot**, chạy **backup**, hoặc làm việc với **database nặng**.
**Bây giờ, nếu bucket IO credit của bạn hết sạch (không còn credit nào),**
bạn sẽ **không thể thực hiện bất kỳ IO nào** trên ổ đĩa.
Bucket IO có dung lượng tối đa **5.4 triệu IO credits**, và nó được nạp lại liên tục theo **tốc độ baseline** của volume.
**Vậy điều này nghĩa là gì?**
Mỗi volume đều có **hiệu suất baseline** (tốc độ cơ bản) dựa trên dung lượng của nó, với một mức tối thiểu.
Nghĩa là, **liên tục** có **100 IO credits** được nạp vào bucket mỗi giây.
**Tức là:** Dù trong bất kỳ tình huống nào, bạn cũng luôn có thể sử dụng ít nhất **100 IO credits mỗi giây**, tương đương **100 IOPS**.
**Tốc độ baseline thực tế** mà bạn nhận được với **GP2** được tính dựa trên **dung lượng volume**.
Bạn được **3 IO credits mỗi giây, cho mỗi 1 GB** dung lượng.
Ví dụ: Một volume **100 GB** sẽ nhận **300 IO credits mỗi giây** (refilling vào bucket).
- Bất kỳ volume nào **dưới 33.33 GB** đều được hưởng mức **tối thiểu 100 IOPS**.
- Volume **trên 33.33 GB** sẽ được baseline = **3 × dung lượng GB**.
**Bạn không bị giới hạn chỉ ở mức baseline.** Mặc định, **GP2 có thể burst lên tới 3.000 IOPS** — tức là thực hiện **3.000 thao tác IO** (mỗi IO = 16 KB) trong **1 giây**. Đây gọi là **burst rate**.

Nghĩa là: Nếu workload của bạn **không liên tục nặng**, bạn **không bị giới hạn** bởi baseline (3 IOPS/GB). Bạn vẫn có thể dùng nhỏ volume nhưng thỉnh thoảng có workload nặng — điều này hoàn toàn ổn.

**Điều còn tuyệt hơn** là bucket credit **bắt đầu đầy** khi volume mới được tạo.
Vậy bucket có **5.4 triệu IO credits**.

Điều này có nghĩa là bạn có thể chạy ở mức **3.000 IOPS** (3.000 IO mỗi giây) trong **đầy 30 phút**.

_(Tính toán trên giả sử bucket không được nạp thêm credit mới — nhưng thực tế nó luôn được nạp)._

**Thực tế thì** bạn có thể burst ở mức tối đa **lâu hơn rất nhiều**.

Điều này cực kỳ tốt nếu volume của bạn ban đầu được dùng cho các workload nặng, vì **lượng credit khởi tạo đầy** là một bộ đệm rất mạnh.

**Takeaway quan trọng nhất ở đây là:**

- Nếu bạn **tiêu thụ IO credits nhanh hơn tốc độ bucket được nạp** → bạn đang **làm cạn kiệt bucket**.
- Ví dụ: Khi burst lên **3.000 IOPS** mà baseline của bạn thấp hơn → theo thời gian bạn sẽ **giảm dần credit** trong bucket.
- Ngược lại, nếu bạn **tiêu thụ ít hơn baseline** → bucket sẽ **được nạp đầy lại**.

**Một trong những điểm quan trọng** của loại lưu trữ này là **bạn phải quản lý credit bucket của tất cả các volume**.

Vì vậy bạn cần đảm bảo rằng **credit bucket luôn được nạp đầy** và **không bị cạn kiệt xuống mức zero**.

Mỗi volume đều được cấp **3 IO credits mỗi giây cho mỗi GB dung lượng** (áp dụng cho volume tối đa **1 TB**).

Với các volume **lớn hơn 1 TB**, chúng sẽ có **baseline performance bằng hoặc vượt mức burst rate 3.000 IOPS**. Do đó chúng **luôn đạt được baseline performance** như một tiêu chuẩn mặc định. Chúng **không sử dụng hệ thống credit**.

**Tốc độ IO tối đa** hiện tại của GP2 là **16.000 IOPS**. Vì vậy bất kỳ volume nào **trên 5.33 TB** sẽ đạt được tốc độ tối đa này **liên tục**.

**GP2 là loại lưu trữ rất linh hoạt**, phù hợp cho mục đích sử dụng chung.

Tại thời điểm làm bài học này, nó vẫn là **loại mặc định**, nhưng mình dự đoán nó sẽ dần được thay thế bởi **GP3** (mình sẽ nói chi tiết ở phần sau).

**GP2 rất phù hợp cho:**

- Boot volume
- Ứng dụng interactive cần độ trễ thấp (low latency)
- Môi trường dev & test
- Bất kỳ trường hợp nào bạn **không có lý do cụ thể** để chọn loại khác.

Bạn cũng có thể sử dụng **Elastic Volume feature** để thay đổi loại lưu trữ giữa GP2 và các loại khác. Mình sẽ hướng dẫn cách làm trong bài học sắp tới (đặc biệt nếu bạn học SysOps hoặc Developer Associate).

**Nếu bạn đang học theo architecture stream** thì phần lý thuyết về kiến trúc này là đủ rồi.

Tại thời điểm này, tôi muốn chuyển sang và giải thích **chính xác** cách **GP3** khác biệt như thế nào.

**GP3** cũng là loại dựa trên SSD, nhưng nó loại bỏ hoàn toàn **kiến trúc credit bucket** của GP2 để thay bằng một thứ **đơn giản hơn rất nhiều**.

Mọi volume **GP3**, bất kể kích thước bao nhiêu, đều **bắt đầu với mức chuẩn 3.000 IOPS**.

Tức là **3.000 * 16 KB operations mỗi giây** và nó có thể chuyển **125 MB mỗi giây**.

Mức này là **tiêu chuẩn**, áp dụng cho **tất cả các volume**, không phụ thuộc vào kích thước.

Và giống như GP2, volume GP3 có thể có dung lượng từ **1 GB** đến **16 TB**.

Hiện tại, **giá cơ bản** của GP3 tại thời điểm làm bài học này **rẻ hơn 20%** so với GP2.

Vì vậy, nếu bạn chỉ dự định sử dụng **tối đa 3.000 IOPS**, thì đây là lựa chọn **không cần suy nghĩ** (no brainer). **Bạn nên chọn GP3 thay vì GP2.**

Nếu bạn cần hiệu suất cao hơn, bạn có thể trả thêm tiền để đạt tối đa **16.000 IOPS** và **1.000 MB mỗi giây** throughput.

Và ngay cả khi thêm những tùy chọn này, **GP3 vẫn thường rẻ hơn GP2** về tổng thể.

**GP3** còn cung cấp **throughput tối đa cao hơn**, bạn có thể đạt tới **1.000 MB/giây**, so với mức tối đa chỉ **250 MB/giây** của GP2.

**Vậy GP3 đơn giản hơn rất nhiều để hiểu** so với GP2 đối với hầu hết mọi người.

Và tôi nghĩ theo thời gian, nó **sẽ trở thành loại mặc định**.

Hiện tại, tại thời điểm tạo bài học này, **GP2 vẫn là loại mặc định**.

**Tóm lại**, GP3 giống như **GP2 và IO1** (mà tôi sẽ nói sau) **sinh ra một đứa con**. Bạn nhận được **một số lợi ích của cả hai** trong một loại lưu trữ **SSD mục đích chung mới**.

Các **kịch bản sử dụng** của GP3 cũng **gần như giống hệt** với GP2.

Ví dụ:

- Virtual desktops
- Database quy mô trung bình
- Ứng dụng cần độ trễ thấp (low-latency applications)
- Môi trường dev và testing
- Boot volumes

Bạn có thể **an toàn chuyển từ GP2 sang GP3 bất kỳ lúc nào**, nhưng cần lưu ý rằng:

Với bất kỳ nhu cầu nào **trên 3.000 IOPS**, hiệu suất **không được tự động cộng thêm** như ở GP2 (nơi baseline tự động tăng theo kích thước volume).

Với GP3, bạn **phải chủ động mua thêm** các IOPS extra này (và trả thêm chi phí).

Điều này cũng tương tự với **throughput bổ sung**.

Ngoài mức chuẩn **125 MB/giây**, mọi thứ vượt quá đều là **extra** phải trả tiền.

Tuy nhiên, **ngay cả khi cộng thêm các extra đó**, thì GP3 **vẫn kinh tế hơn GP2** trong hầu hết các trường hợp.