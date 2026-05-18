![[Pasted image 20260518110641.png]]
Bài này sẽ nói về
- **Throughput Optimized HDD** — Ổ cứng HDD giá rẻ được thiết kế cho các workload thường xuyên truy cập, đòi hỏi throughput cao.
- **Cold HDD** — Ổ cứng HDD có chi phí thấp nhất, được thiết kế cho các workload ít được truy cập.

**HDD-based** nghĩa là chúng có các bộ phận chuyển động. Đó là những **platter quay** (đĩa quay), những cánh tay robot nhỏ gọi là **heads** di chuyển qua lại trên những platter đang quay đó.

**Có bộ phận chuyển động nên sẽ chậm hơn**, đó là lý do bạn chỉ nên sử dụng các loại volume này trong **những tình huống rất cụ thể**.

Bây giờ, chúng ta đi thẳng vào vấn đề và xem xét các tình huống mà bạn nên sử dụng **lưu trữ dựa trên HDD**. Hiện tại có hai loại **HDD-based storage** trong EBS. Thực ra nói vậy không đúng, **thực tế có ba loại**, nhưng một loại đã là legacy nên mình sẽ chỉ trình bày hai loại đang được sử dụng phổ biến.

Đó là **st1** — **Throughput Optimized HDD** và **sc1** — **Cold HDD**.

Hãy nghĩ **st1** như một ổ cứng nhanh, không quá linh hoạt nhưng **khá nhanh**. Còn **sc1** thì coi như kiểu **lạnh** (cold). **st1 khá rẻ**, nó rẻ hơn so với các volume SSD, điều này khiến nó trở nên **lý tưởng cho các dung lượng dữ liệu lớn**. **sc1 còn rẻ hơn nữa**, nhưng đi kèm với **một số đánh đổi đáng kể**.

**st1 được thiết kế cho dữ liệu được truy cập tuần tự (sequentially accessed)**. Vì nó dựa trên HDD nên **không giỏi xử lý random access**. Nó phù hợp hơn với dữ liệu cần được ghi hoặc đọc **theo cách khá tuần tự**.

**Các ứng dụng mà throughput và chi phí (economy) quan trọng hơn IOPS hoặc mức hiệu năng cực cao**.

**st1 volumes** có dung lượng từ **125 GB đến 16 TB**, và bạn có **tối đa 500 IOPS**. Nhưng, và đây là điểm quan trọng, **I/O trên các volume HDD-based được tính theo khối 1MB**.

Vì vậy **500 IOPS nghĩa là 500 MB mỗi giây**.

Bây giờ là thông số tối đa của chúng.

**Lưu trữ HDD-based** hoạt động tương tự như **gp2 volumes** với cơ chế credit bucket, chỉ khác là với **HDD-based volumes**, hiệu năng được đo bằng **MB mỗi giây** thay vì IOPS.

Cụ thể với **st1**, bạn có **baseline performance** là **40 MB mỗi giây cho mỗi 1TB dung lượng volume**, và có thể **burst lên tối đa 250 MB mỗi giây cho mỗi TB dung lượng**. Tất nhiên, không vượt quá giới hạn **500 IOPS và 500 MB mỗi giây**.

**st1** được thiết kế cho trường hợp **chi phí là mối quan tâm**, nhưng bạn vẫn cần **lưu trữ truy cập thường xuyên** cho các **workload sequential throughput-intensive**.

Ví dụ như **big data, data warehouse và log processing**.

**Ngược lại, sc1** được thiết kế cho các **workload ít truy cập** (infrequent workloads).

Nó hướng đến **mức kinh tế tối đa** khi bạn chỉ muốn **lưu trữ lượng dữ liệu lớn** và **không quan tâm đến hiệu năng**.

Vì vậy nó chỉ cung cấp **tối đa 250 IOPS**. Cũng với kích thước I/O 1MB, điều này nghĩa là **tối đa 250 MB mỗi giây** throughput.

Và giống như **st1**, nó cũng sử dụng **cùng kiến trúc credit pool**, nên có **baseline 12 MB mỗi giây cho mỗi 1TB dung lượng**, và có thể **burst lên 80 MB mỗi giây cho mỗi TB dung lượng**.

Bạn có thể thấy **sc1 cho hiệu năng thấp hơn khá nhiều so với st1**, nhưng **nó cũng rẻ hơn rất nhiều**.

**sc1 volumes** cũng có dung lượng từ **125 GB đến 16 TB**.

Đây là **loại lưu trữ EBS có chi phí thấp nhất hiện có**. Nó được thiết kế cho các **workload ít được truy cập**.

Vậy nếu bạn có **dữ liệu lạnh (colder data)**, **archive**, hoặc bất kỳ thứ gì chỉ cần **vài lần đọc/ghi hoặc quét mỗi ngày**, thì đây chính là loại volume nên chọn.

**Và đó là tất cả về lưu trữ HDD-based.**

Cả hai loại đều có **chi phí thấp hơn và hiệu năng thấp hơn** so với **SSD**.

Việc chọn giữa chúng rất đơn giản.

**Nếu bạn có thể chấp nhận những đánh đổi của sc1**, thì hãy dùng nó. Nó **siêu rẻ** và **hoàn hảo** cho mọi thứ **không được truy cập hàng ngày**.

Ngược lại, hãy chọn **st1**.

Còn **nếu bạn cần hiệu năng dựa trên IOPS**, thì nên tránh cả hai và chuyển sang **lưu trữ dựa trên SSD**.

**Với tất cả những điều trên**, đó là toàn bộ nội dung mình muốn trình bày trong bài học này.