Welcome back! Trong video này, tôi muốn nói về các cách RDS cung cấp **High Availability**. Trong quá khứ, chỉ có một cách duy nhất: **Multi-AZ**. Theo thời gian, RDS đã được cải tiến và hiện nay có **Multi-AZ instance deployments** và **Multi-AZ cluster deployments**. Hai kiến trúc này mang lại lợi ích và trade-off khác nhau, vì vậy trong video này tôi sẽ đi qua kiến trúc và chức năng của cả hai. Chúng ta có khá nhiều nội dung cần cover, nên hãy bắt đầu ngay.

### Multi-AZ Instance Deployment (kiến trúc cũ)

Trong lịch sử, phương pháp duy nhất để cung cấp **High Availability** của RDS là **Multi-AZ**. Hiện nay nó được gọi là **Multi-AZ instance deployment**.

Với kiến trúc này, RDS có một **primary database instance** chứa tất cả các database bạn tạo. Khi bạn bật chế độ Multi-AZ, primary instance sẽ được cấu hình để **replicate dữ liệu đồng bộ (synchronous)** sang một **standby replica** đang chạy ở một **Availability Zone** khác. Điều này có nghĩa là standby cũng có bản sao của database của bạn.

Trong **Multi-AZ instance mode**, quá trình replication diễn ra ở **mức storage**. Cách này thực tế kém hiệu quả hơn so với kiến trúc Multi-AZ cluster, nhưng chúng ta sẽ nói về điều đó sau. Phương pháp chính xác mà RDS dùng để replicate phụ thuộc vào database engine bạn chọn:

- **MariaDB, MySQL, Oracle và PostgreSQL** sử dụng công nghệ failover của Amazon.
- **Microsoft SQL Server** sử dụng SQL Server database mirroring hoặc Always On availability groups.

Dù vậy, tất cả đều được ẩn đi. Bạn chỉ cần hiểu rằng đây là **synchronous replica**.

Về mặt kiến trúc, mọi truy cập đến database đều đi qua **database CNAME**. Đây là một DNS name mặc định trỏ đến primary database instance. Với kiến trúc Multi-AZ instance, bạn **luôn chỉ truy cập primary**. Không có quyền truy cập standby, kể cả cho thao tác **read**. Nhiệm vụ của standby chỉ là ngồi đó chờ khi primary gặp sự cố.

Tuy nhiên, các thao tác khác như **backup** có thể thực hiện từ standby. Dữ liệu được đưa lên S3 rồi replicate sang nhiều Availability Zone trong region. Cách này **không tạo thêm tải** cho primary vì backup diễn ra từ standby. Điều này rất quan trọng vì tất cả truy cập (cả read và write) trong kiến trúc Multi-AZ instance đều diễn ra trên primary.

Khi primary gặp sự cố, RDS sẽ phát hiện và thực hiện **failover**. Bạn có thể kích hoạt thủ công để test hoặc bảo trì, nhưng thông thường đây là quá trình tự động. Trong trường hợp này, **database CNAME** sẽ thay đổi: thay vì trỏ đến primary cũ, nó sẽ trỏ đến standby (giờ trở thành primary mới). Vì đây là thay đổi DNS, thời gian failover thường mất **60 đến 120 giây**, có thể gây ra outage ngắn. Bạn có thể giảm thời gian này bằng cách **loại bỏ DNS caching** trong application đối với CNAME này. Nếu bỏ caching, ngay khi RDS hoàn tất failover và cập nhật DNS, application của bạn sẽ dùng ngay tên mới trỏ đến primary mới.

#### Các điểm quan trọng của Multi-AZ Instance:

- Replication giữa primary và standby là **synchronous**: dữ liệu được ghi vào primary rồi ngay lập tức replicate sang standby trước khi được coi là committed.
- **Multi-AZ không nằm trong Free Tier** vì có chi phí thêm cho standby replica.
- Chỉ có **một standby replica duy nhất**, và standby **không thể dùng cho read hay write**. Nhiệm vụ của nó chỉ là chờ failover.
- Thời gian failover: **60–120 giây**.
- Multi-AZ chỉ hoạt động **trong cùng một region** (các Availability Zone khác nhau trong cùng AWS region).
- Backup có thể lấy từ standby để cải thiện hiệu năng.
- Failover xảy ra vì nhiều lý do: **Availability Zone outage**, primary instance failure, manual failover, thay đổi instance type, và thậm chí khi **patching software**. Bạn có thể dùng failover để chuyển consumer sang instance khác, patch instance không còn consumer, rồi flip lại. Đây là tính năng rất hữu ích để duy trì availability của application.

### Multi-AZ Cluster Deployment

Tiếp theo là **Multi-AZ sử dụng kiến trúc cluster**. Khi bạn xem video về Aurora, bạn có thể bị nhầm lẫn giữa kiến trúc này và Amazon Aurora. Tôi sẽ nhấn mạnh sự khác biệt giữa **Multi-AZ cluster cho RDS** và **Aurora** để bạn chuẩn bị tốt hơn khi xem video Aurora. Việc hiểu rõ sự khác biệt này là cực kỳ quan trọng.

Chúng ta vẫn bắt đầu với kiến trúc VPC tương tự, nhưng giờ ngoài một client, tôi thêm hai client nữa. Trong chế độ này, RDS có thể có **một writer** replicate sang **hai reader instances**. Đây là điểm khác biệt then chốt so với Aurora: với Multi-AZ cluster của RDS, bạn **chỉ có tối đa hai readers**. Các reader nằm ở Availability Zone khác với writer, nhưng chỉ có hai cái, trong khi Aurora có thể có nhiều hơn.

Khác với Multi-AZ instance mode, các **reader trong cluster mode là có thể sử dụng được**. Bạn có thể coi writer giống như primary trong instance mode: nó dùng được cho cả **write và read**. Các reader instance (khác với standby trong instance mode) có thể được dùng khi đang ở trạng thái này — **chỉ cho thao tác read**. Application của bạn cần hỗ trợ việc này vì không thể dùng chung một instance cho cả read và write. Tuy nhiên, điều này cho phép bạn **scale read workload**, điều mà Multi-AZ instance mode không làm được.

Về replication giữa writer và readers: dữ liệu được gửi đến writer và được coi là **committed** khi **ít nhất một reader** xác nhận đã ghi thành công. Lúc này dữ liệu đã có resilience trên nhiều Availability Zone trong region.

Cluster mà RDS tạo ra trong kiến trúc này có điểm giống và khác so với Aurora. Trong RDS Multi-AZ cluster mode, **mỗi instance vẫn có local storage riêng** (khác với Aurora). Giống Aurora, bạn truy cập cluster thông qua một số loại **endpoint**:

- **Cluster endpoint**: giống như database CNAME trong kiến trúc trước. Nó trỏ đến writer và dùng được cho read, write cũng như các thao tác administration.
- **Reader endpoint**: trỏ đến bất kỳ reader nào đang available trong cluster. Trong một số trường hợp, nó cũng có thể bao gồm writer (vì writer cũng dùng được cho read). Trong hoạt động bình thường, reader endpoint sẽ trỏ đến các dedicated reader instances. Đây là cách **scale read** trong cluster. Application có thể dùng reader endpoint để balance các thao tác read.
- **Instance endpoints**: mỗi instance trong cluster đều có một endpoint riêng. Nói chung **không khuyến khích** dùng trực tiếp vì nếu instance đó fail thì không có cơ chế chuyển tiếp. Bạn chỉ nên dùng để testing và fault-finding.

#### Các điểm quan trọng của Multi-AZ Cluster:

- RDS Multi-AZ cluster mode gồm **một writer + hai reader** DB instances nằm ở các Availability Zone khác nhau → mức availability cao hơn instance mode vì có thêm một reader so với chỉ một standby.
- Chạy trên **hardware nhanh hơn nhiều**: kiến trúc **Graviton** và sử dụng **local NVMe SSD**. Mọi write được ghi trước vào local storage siêu nhanh rồi mới flush ra EBS → vừa có lợi thế của local storage nhanh, vừa có resilience của EBS.
- Readers có thể dùng để **scale read operations**. Nếu application hỗ trợ, bạn có thể đưa các thao tác read sang reader endpoint → giải phóng capacity trên writer và cho phép RDS scale hiệu năng cao hơn các mode khác.
- Replication được thực hiện bằng **transaction logs** → hiệu quả hơn rất nhiều.
- Failover nhanh hơn: chỉ mất khoảng **35 giây** + thời gian apply transaction logs lên reader instances (vẫn nhanh hơn rất nhiều so với 60–120 giây của instance mode).
- Writes được coi là committed khi đã gửi đến writer, lưu trữ và được **ít nhất một reader** xác nhận đã ghi thành công.

Như bạn thấy, đây là hai kiến trúc hoàn toàn khác nhau. Theo quan điểm của tôi, **Multi-AZ cluster mode** mang lại nhiều lợi ích đáng kể so với instance mode. Bạn sẽ thấy chức năng này được mở rộng hơn nữa khi tôi nói về Amazon Aurora. Nhưng đến đây là toàn bộ nội dung tôi muốn cover trong video này. Cảm ơn bạn đã xem. Hãy hoàn thành video và khi sẵn sàng, tôi rất mong được gặp bạn ở video tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Multi-AZ Instance Deployment vs Multi-AZ Cluster Deployment
- Synchronous replication (storage-level vs transaction logs)
- Primary/Standby vs Writer + 2 Readers
- Failover time (60–120s vs ~35s)
- Endpoint types (Cluster / Reader / Instance)
- Local NVMe SSD + Graviton
- Read scaling capability
- Khác biệt với Aurora

**Notes (Chi tiết):**

- **Multi-AZ Instance**: Primary replicate đồng bộ (storage-level) sang 1 standby ở AZ khác. Chỉ truy cập primary qua CNAME. Standby **không dùng được** cho read/write. Failover DNS mất 60–120 giây. Backup lấy từ standby.
- **Multi-AZ Cluster**: 1 Writer + 2 Readers ở các AZ khác nhau. Readers **dùng được cho read**. Replication bằng transaction logs (hiệu quả hơn). Failover nhanh hơn (~35 giây). Chạy trên Graviton + local NVMe SSD.
- Endpoints quan trọng: Cluster endpoint (writer), Reader endpoint (scale read), Instance endpoints (chỉ dùng test).
- Writes committed khi ít nhất 1 reader xác nhận.
- Cluster mode của RDS chỉ có tối đa 2 readers (khác Aurora có thể scale nhiều hơn) và mỗi instance vẫn có local storage riêng.

**Summary (Tóm tắt ngắn):** Video giải thích chi tiết hai kiến trúc High Availability của RDS: **Multi-AZ Instance** (cũ, chỉ 1 standby không dùng được, failover 60–120s) và **Multi-AZ Cluster** (mới, 1 writer + 2 readers dùng được cho read, failover nhanh hơn, hardware mạnh hơn với Graviton + NVMe). Cluster mode mang lại nhiều lợi ích hơn về availability, performance và khả năng scale read.