Chào mừng bạn trở lại. Trong video này, tôi muốn nói về **RDS Read Replicas**. **Read replicas** mang lại một số lợi ích chính cho chúng ta với tư cách là **Solutions Architect** hoặc kỹ sư vận hành:

- Cải thiện **hiệu năng** cho các thao tác **đọc** (read operations)
- Giúp tạo khả năng **failover liên vùng** (cross-region failover)
- Cung cấp cách để RDS đạt được **RTO rất thấp** (Recovery Time Objective), miễn là **không liên quan đến dữ liệu bị hỏng** trong kịch bản thảm họa.

Bây giờ chúng ta hãy đi qua các khái niệm và kiến trúc chính, vì chúng sẽ hữu ích cả cho kỳ thi lẫn thực tế.

**Read replicas**, đúng như tên gọi, là các **bản sao chỉ đọc** (read-only) của một instance RDS. Khác với **Multi-AZ**, nơi bạn **không thể** sử dụng standby replica cho bất kỳ việc gì theo mặc định, bạn **có thể** sử dụng read replicas, nhưng **chỉ cho các thao tác đọc**.

**Multi-AZ** chạy ở chế độ **cluster** (phiên bản mới hơn của Multi-AZ) giống như sự kết hợp giữa Multi-AZ instance mode cũ với read replicas. Nhưng điều **rất quan trọng** cần nhớ: bạn phải coi **read replicas là những thứ riêng biệt**. Chúng **không phải** là một phần của database instance chính theo bất kỳ cách nào. Chúng có **endpoint riêng** của mình, vì vậy ứng dụng cần được điều chỉnh để sử dụng chúng. Một ứng dụng (ví dụ WordPress) đang dùng RDS instance sẽ **hoàn toàn không biết** đến bất kỳ read replica nào theo mặc định. Nếu ứng dụng không hỗ trợ, read replicas **không có tác dụng gì** từ góc độ sử dụng. Không có failover tự động; chúng chỉ tồn tại ở một bên.

Chúng được giữ đồng bộ bằng **replication bất đồng bộ** (asynchronous replication). Hãy nhớ: **Multi-AZ** dùng **replication đồng bộ** (synchronous). Điều này có nghĩa là khi dữ liệu được ghi vào primary instance, cùng lúc dữ liệu đó được lưu trên đĩa của primary, nó cũng được replicate sang standby. Về mặt khái niệm, hãy nghĩ đây là **một thao tác ghi duy nhất** trên cả primary và standby.

Với **asynchronous**, dữ liệu được ghi vào **primary trước**, tại thời điểm đó nó được coi là **đã commit**; sau đó mới được replicate sang read replicas. Do đó, về lý thuyết có thể có một độ trễ nhỏ (có thể là vài giây), phụ thuộc vào điều kiện mạng và số lượng thao tác ghi trên database.

**Cho kỳ thi**, với bất kỳ câu hỏi nào về RDS (và **tạm thời loại trừ Aurora**), hãy nhớ:

- **Synchronous** = Multi-AZ
- **Asynchronous** = Read Replicas

Read replicas có thể được tạo **trong cùng region** với primary database instance, hoặc tạo **ở các region AWS khác** (gọi là **cross-region read replicas**). Nếu bạn tạo cross-region read replica, AWS sẽ xử lý **toàn bộ networking** giữa các region, quá trình này diễn ra **trong suốt** đối với bạn và được **mã hóa hoàn toàn khi truyền** (encrypted in transit).

**Tại sao read replicas quan trọng?** Có **hai lĩnh vực chính** bạn cần nghĩ đến:

### 1. Hiệu năng đọc và khả năng scale đọc

Bạn có thể tạo **tối đa 5 read replicas trực tiếp** cho mỗi database instance, và mỗi cái cung cấp thêm một instance hiệu năng đọc. Đây là cách đơn giản để **scale-out** hiệu năng đọc của database.

Bản thân read replicas cũng có thể có **read replicas của riêng chúng**, nhưng điều này khiến **độ trễ (lag)** trở thành vấn đề. Vì dùng asynchronous replication, có thể có lag giữa main database và các read replicas. Nếu bạn tạo read replicas của read replicas thì lag càng lớn hơn. Vì vậy, dù bạn có thể dùng nhiều cấp read replicas để scale hiệu năng đọc nhiều hơn, lag sẽ ngày càng nghiêm trọng, bạn cần cân nhắc điều đó.

Ngoài ra, read replicas giúp cải thiện **hiệu năng toàn cầu** cho các workload đọc. Nếu bạn có workload đọc ở các region AWS khác, các workload đó có thể kết nối trực tiếp đến read replicas mà **không ảnh hưởng** đến hiệu năng của primary instance.

### 2. RPO và RTO

- **Snapshots và backups** giúp cải thiện **RPO** (Recovery Point Objective). Snapshot càng thường xuyên và backup càng tốt thì RPO càng tốt vì giới hạn lượng dữ liệu có thể bị mất. Nhưng chúng **không giúp nhiều** cho **RTO**, vì restore snapshot mất rất nhiều thời gian, đặc biệt với database lớn.
- **Read replicas** mang lại **RPO gần bằng 0**. Vì dữ liệu trên read replica được đồng bộ từ main database, nên tiềm năng mất dữ liệu rất thấp (giả sử không có data corruption).
- Read replicas có thể được **promote rất nhanh**, mang lại **RTO gần bằng 0**. Trong kịch bản thảm họa khi RDS instance gặp vấn đề lớn, bạn có thể promote một read replica và đây là quá trình rất nhanh.

**Nhưng điều cực kỳ quan trọng**: bạn **chỉ nên** xem xét dùng read replicas trong các kịch bản disaster recovery khi **đang phục hồi từ sự cố (failure)**. Nếu bạn đang phục hồi từ **data corruption**, thì về logic, read replica **có thể cũng chứa bản sao của dữ liệu bị hỏng đó**. Vì vậy, read replicas rất tốt để đạt **RTO thấp**, nhưng **chỉ cho failure**, **không phải cho data corruption**.

Read replicas là **chỉ đọc** cho đến khi được **promote**. Khi được promote, bạn có thể sử dụng chúng như một **RDS instance bình thường**.

Chúng cũng là cách **rất đơn giản** để đạt được cải thiện **tính sẵn sàng toàn cầu** và **khả năng phục hồi toàn cầu**, vì bạn có thể tạo một **cross-region read replica** ở region khác và dùng nó như một **failover region** nếu AWS gặp sự cố lớn ở một region.

Đến đây là tất cả những gì tôi muốn đề cập về read replicas. Nếu phù hợp với kỳ thi bạn đang học, tôi có thể có bài học khác đi sâu hơn về kỹ thuật hoặc bài demo để bạn trải nghiệm thực tế. Nếu bạn không thấy những bài đó thì đừng lo, chúng không bắt buộc cho kỳ thi bạn đang học.

Đến đây là hết nội dung tôi sẽ đề cập. Hãy hoàn thành video, và khi sẵn sàng, tôi mong được gặp bạn ở bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Lợi ích chính của Read Replicas
- Khác biệt với Multi-AZ
- Asynchronous vs Synchronous replication
- Scale đọc (tối đa 5 + cascaded)
- Cross-region Read Replicas
- RPO / RTO với Read Replicas
- Promote Read Replica
- Giới hạn khi dùng cho Disaster Recovery

**Notes (Chi tiết):**

- **Read Replicas** = bản sao **chỉ đọc** của RDS instance. Có endpoint riêng → ứng dụng phải chủ động kết nối.
- Khác Multi-AZ: standby **không dùng được** theo mặc định; read replica **dùng được cho đọc**.
- **Multi-AZ Cluster** ≈ Multi-AZ cũ + Read Replicas, nhưng vẫn phải coi Read Replicas là **riêng biệt**.
- **Replication**:
    - Multi-AZ → **Synchronous**
    - Read Replicas → **Asynchronous** (có thể có lag vài giây)
- Tối đa **5 read replicas trực tiếp**/instance. Có thể tạo read replica của read replica (cascaded) nhưng **lag tăng mạnh**.
- Cross-region: AWS tự lo networking + **encrypted in transit**.
- **RPO gần 0** (dữ liệu được đồng bộ) + **RTO gần 0** (promote rất nhanh).
- **Chỉ dùng cho failure**, **không dùng khi data corruption** (vì replica cũng chứa dữ liệu hỏng).
- Sau khi **Promote** → trở thành RDS instance bình thường (đọc/ghi).

**Summary (Tóm tắt ngắn):** **Read Replicas** là bản sao **chỉ đọc** dùng **asynchronous replication**, giúp **scale hiệu năng đọc** (tối đa 5 trực tiếp), hỗ trợ **cross-region**, và đạt **RPO/RTO gần 0** khi disaster recovery do **failure**. Chúng có endpoint riêng, ứng dụng phải tự kết nối, và **không phù hợp** khi phục hồi từ **data corruption**.