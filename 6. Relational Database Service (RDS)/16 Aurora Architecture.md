**Chào mừng trở lại**, và trong bài học này, tôi sẽ trình bày về **kiến trúc của Amazon Aurora** – sản phẩm cơ sở dữ liệu được quản lý từ AWS. Tôi đã đề cập trước đó rằng Aurora chính thức là một phần của RDS, nhưng theo góc nhìn của tôi, tôi luôn coi nó như một **sản phẩm riêng biệt**. Các tính năng mà nó cung cấp và kiến trúc mà nó sử dụng để mang lại những tính năng đó khác biệt **rất lớn** so với RDS thông thường, nên cần được xem như một sản phẩm độc lập. Chúng ta có khá nhiều nội dung cần bao quát, vậy hãy bắt đầu ngay.

Như tôi vừa đề cập, **kiến trúc Aurora rất khác** so với RDS thông thường. Ở tầng nền tảng nhất, nó sử dụng thực thể cơ bản là một **cluster** – điều mà các engine khác trong RDS không có. Và một cluster được cấu thành từ một số thành phần quan trọng.

**Thứ nhất**, về mặt compute, nó bao gồm **một primary instance duy nhất**, và sau đó là **zero hoặc nhiều hơn các replica**. Điều này có vẻ giống với cách RDS hoạt động với primary và standby replica, nhưng thực tế **rất khác**. Các replica trong Aurora **có thể được sử dụng cho đọc (read)** trong quá trình hoạt động bình thường, không giống như standby replica trong RDS. Các replica trong Aurora thực sự mang lại lợi ích của **cả RDS Multi-AZ và RDS read replica**. Chúng nằm trong cluster và có thể dùng để **cải thiện availability**, đồng thời cũng có thể dùng cho **thao tác đọc** trong lúc cluster đang hoạt động bình thường. Chỉ riêng điều này đã đáng để chuyển sang Aurora, vì bạn **không phải chọn giữa scale đọc và availability**. Replica trong Aurora cung cấp **cả hai lợi ích** đó.

**Thứ hai**, sự khác biệt lớn về kiến trúc của Aurora nằm ở **storage**. Aurora **không sử dụng local storage** cho các compute instance. Thay vào đó, một Aurora cluster có một **shared cluster volume**. Đây là storage được **chia sẻ và sẵn có cho tất cả** các compute instance trong cluster. Điều này mang lại một số lợi ích như **provisioning nhanh hơn**, **availability tốt hơn**, và **performance cao hơn**.

Một Aurora cluster điển hình trông như thế này: nó hoạt động trên **nhiều Availability Zone** – trong ví dụ này là A, B và C. Bên trong cluster có một **primary instance**, và tùy chọn là một số **replica**. Những replica này vừa đóng vai trò **failover** nếu primary instance bị lỗi, vừa có thể được dùng trong hoạt động bình thường của cluster cho các **thao tác đọc** từ ứng dụng.

Cluster có **shared storage dựa trên SSD**, với kích thước tối đa **128 TiB**. Nó cũng có **sáu bản sao (replicas) storage** trải trên nhiều Availability Zone. Khi dữ liệu được ghi vào primary DB instance, Aurora **đồng bộ (synchronously) replicate** dữ liệu đó trên tất cả sáu storage node được phân bố trên các Availability Zone liên kết với cluster của bạn.

**Tất cả instance** trong cluster – primary và tất cả replica – đều có quyền truy cập vào **tất cả** các storage node này. Điểm quan trọng cần hiểu về storage là quá trình replication này diễn ra **ở tầng storage**, nên **không tiêu tốn thêm tài nguyên** trên instance hay replica trong quá trình replication. Mặc định, **chỉ primary instance** mới có thể **ghi** vào storage, còn replica và primary đều có thể thực hiện **thao tác đọc**.

Vì Aurora duy trì **nhiều bản sao dữ liệu trên ba Availability Zone**, khả năng mất dữ liệu do lỗi liên quan đến disk được **giảm thiểu rất nhiều**. Aurora **tự động phát hiện** lỗi trên các disk volume tạo nên shared storage của cluster. Khi một segment hoặc một phần của disk volume bị lỗi, Aurora **ngay lập tức sửa chữa** vùng disk đó. Khi sửa chữa, nó sử dụng dữ liệu từ các storage node khác trong cluster volume, **tự động tái tạo** dữ liệu và đảm bảo dữ liệu được đưa trở lại trạng thái hoạt động mà **không bị hỏng**. Kết quả là Aurora **tránh mất dữ liệu** và **giảm nhu cầu** phải thực hiện point-in-time restore hoặc snapshot restore để phục hồi từ lỗi disk. Vì vậy, **hệ thống storage của Aurora bền vững hơn nhiều** so với các engine RDS thông thường.

Một điểm khác biệt mạnh mẽ khác giữa Aurora và các engine RDS thông thường là với Aurora bạn có thể có **tối đa 15 replica**, và **bất kỳ replica nào** cũng có thể là **failover target**. Thay vì chỉ có một primary instance và một standby replica như các engine không phải Aurora, với Aurora bạn có **15 replica khác nhau** để lựa chọn failover. Và thao tác failover sẽ **nhanh hơn nhiều** vì **không cần phải thực hiện bất kỳ thay đổi storage nào**.

![[Pasted image 20260730204255.png]]

Ngoài độ bền vững mà cluster volume mang lại, còn một số yếu tố quan trọng khác bạn cần biết. **Shared volume của cluster** dựa trên **SSD storage** theo mặc định, nên cung cấp **IOPS cao và latency thấp**. Đây là **storage hiệu năng cao mặc định**; bạn **không có tùy chọn** sử dụng magnetic storage.

**Cách tính phí storage** cũng rất khác so với các engine RDS thông thường. Với Aurora, bạn **không cần allocate** lượng storage mà cluster sử dụng. Khi tạo Aurora cluster, bạn **không chỉ định** dung lượng storage cần thiết. Storage chỉ đơn giản dựa trên **những gì bạn tiêu thụ**. Khi bạn lưu trữ dữ liệu lên đến giới hạn **128 TiB**, bạn sẽ bị tính phí theo **mức tiêu thụ**.

Cách tính tiêu thụ này dựa trên **high-water mark**. Nếu bạn tiêu thụ **50 GiB** storage, bạn sẽ bị tính phí cho **50 GiB**. Nếu bạn giải phóng **10 GiB** dữ liệu – tức giảm xuống còn **40 GiB** dữ liệu đang dùng – bạn **vẫn bị tính phí** cho high-water mark là **50 GiB**, nhưng bạn **có thể tái sử dụng** bất kỳ storage nào đã giải phóng. Những gì bạn bị tính phí là **high-water mark** – mức storage tối đa mà bạn đã tiêu thụ trong cluster. Và nếu bạn trải qua quá trình giảm đáng kể storage và cần giảm chi phí storage, thì bạn cần **tạo một cluster hoàn toàn mới** và **migrate dữ liệu** từ cluster cũ sang cluster mới.

Cần lưu ý rằng kiến trúc high-water mark này **đang được AWS thay đổi** và **không còn áp dụng** cho các phiên bản Aurora mới hơn. Tôi sẽ cập nhật bài học này khi tính năng này trở nên phổ biến hơn, nhưng hiện tại, bạn **vẫn cần giả định** rằng kiến trúc high-water mark đang được sử dụng.

Vì storage thuộc về **cluster** chứ không phải instance, nên **replica có thể được thêm và xóa** mà **không cần provisioning hoặc xóa storage**, điều này **cải thiện rất lớn** tốc độ và hiệu quả của bất kỳ thay đổi replica nào trong cluster.

Kiến trúc cluster này cũng **thay đổi phương thức truy cập** so với RDS. Aurora cluster, giống như RDS, sử dụng **endpoint**. Đây là các địa chỉ DNS dùng để kết nối tới cluster. Khác với RDS, Aurora cluster có **nhiều endpoint** sẵn có cho ứng dụng. Tối thiểu, bạn có **cluster endpoint** và **reader endpoint**. **Cluster endpoint** luôn trỏ tới **primary instance**, và đây là endpoint có thể dùng cho **cả đọc và ghi**. **Reader endpoint** cũng sẽ trỏ tới primary instance nếu chỉ có mỗi nó, nhưng nếu có replica, thì reader endpoint sẽ **load balance** trên tất cả các replica sẵn có, và có thể dùng cho **thao tác đọc**. Điều này giúp **dễ dàng hơn nhiều** trong việc quản lý **read scaling** với Aurora so với RDS, vì khi bạn thêm các replica mới có thể dùng cho đọc, reader endpoint sẽ **tự động được cập nhật** để load balance trên các replica mới này. Bạn cũng có thể tạo **custom endpoint**, và ngoài ra, **mỗi instance** – primary và bất kỳ replica nào – đều có **endpoint riêng**. Vì vậy Aurora cho phép kiến trúc **tùy chỉnh và phức tạp hơn nhiều** so với RDS.

![[Pasted image 20260730204342.png]]

Bây giờ hãy chuyển sang nói về **chi phí**. Với Aurora, một trong những điểm bất lợi lớn nhất là **không có tùy chọn free tier**. Bạn **không thể dùng Aurora** trong free tier, vì Aurora **không hỗ trợ** các micro instance có sẵn trong free tier. Nhưng đối với bất kỳ instance nào lớn hơn RDS single-AZ micro-sized, Aurora mang lại **giá trị tốt hơn nhiều**.

Đối với mọi compute bạn sử dụng, có **phí theo giờ**, và bạn sẽ bị tính phí **theo giây** với **tối thiểu 10 phút**. Đối với storage, bạn sẽ bị tính phí dựa trên metric **gigabyte-month** tiêu thụ – tất nhiên có tính đến high-water mark, tức dựa trên lượng storage tối đa bạn đã tiêu thụ trong vòng đời của cluster đó. Ngoài ra còn có **chi phí I/O** cho mỗi request gửi tới shared storage của cluster.

Về **backup**, bạn được cấp **100%** lượng storage tiêu thụ của cluster dưới dạng **free backup allocation**. Nếu database cluster của bạn là **100 GiB**, thì bạn được cấp **100 GiB** storage cho backup như một phần của những gì bạn trả cho cluster đó. Vì vậy, trong hầu hết các tình huống sử dụng thấp hoặc trung bình, trừ khi bạn có **turnover dữ liệu cao** hoặc giữ dữ liệu với **thời gian retention dài**, trong hầu hết trường hợp, chi phí backup thường **đã được bao gồm** trong phí bạn trả cho database cluster.

Aurora còn cung cấp một số **tính năng rất thú vị** khác. Nói chung, backup trong Aurora hoạt động **gần như giống hệt** RDS. Các tính năng backup thông thường, automatic backup, manual snapshot backup đều hoạt động giống như bất kỳ engine RDS nào khác. Và restore sẽ **tạo một cluster hoàn toàn mới**, giống như bạn đã trải nghiệm trong bài demo trước khi tạo RDS instance mới từ snapshot, và kiến trúc này mặc định **không thay đổi** khi dùng Aurora.

Nhưng bạn cũng có một số **tính năng nâng cao** có thể thay đổi cách bạn làm việc. Một trong số đó là **Backtrack**. Tính năng này cần được **bật trên từng cluster**, và nó cho phép bạn **roll back** database về một thời điểm trước đó. Hãy tưởng tượng tình huống bạn gặp **corruption nghiêm trọng** bên trong Aurora cluster và xác định được thời điểm corruption xảy ra. Thay vì phải restore sang một database hoàn toàn mới tại thời điểm trước khi corruption, nếu bạn bật Backtrack, bạn có thể **đơn giản roll back tại chỗ** cluster Aurora hiện tại về thời điểm trước khi corruption xảy ra. Điều đó nghĩa là bạn **không cần cấu hình lại ứng dụng**; bạn chỉ cần để chúng tiếp tục dùng cùng cluster đó, chỉ khác là dữ liệu đã được roll back về trạng thái trước khi corruption. Bạn cần bật tính năng này trên từng cluster, và có thể điều chỉnh cửa sổ thời gian mà Backtrack hoạt động, nhưng đây là một **tính năng rất mạnh** và tại thời điểm tạo bài học này thì **chỉ có riêng trên Aurora**.

Bạn cũng có khả năng tạo cái gọi là **Fast Clone**. Fast Clone cho phép bạn tạo một database hoàn toàn mới từ database hiện có, nhưng điểm quan trọng là nó **không tạo bản sao 1-1** của storage. Thay vào đó, nó **tham chiếu đến storage gốc** và chỉ lưu trữ **những khác biệt** giữa hai bên. Khác biệt có thể là bạn cập nhật storage trong database clone, hoặc dữ liệu được cập nhật trên database gốc, nghĩa là clone cần một bản sao của dữ liệu đó trước khi nó bị thay đổi trên source. Về cơ bản, database clone của bạn **chỉ sử dụng một lượng storage rất nhỏ**. Nó chỉ lưu trữ dữ liệu đã thay đổi trong clone hoặc đã thay đổi trên original sau khi bạn tạo clone. Điều đó nghĩa là bạn có thể tạo clone **nhanh hơn nhiều** so với việc phải copy toàn bộ dữ liệu từng bit, và các clone này cũng **không tiêu thụ gần bằng** toàn bộ lượng dữ liệu – chúng chỉ lưu trữ những thay đổi giữa source và clone.

Tôi biết đây là khá nhiều kiến trúc cần ghi nhớ. Tôi đã cố gắng nhanh chóng đi qua tất cả những khác biệt giữa Aurora và các engine RDS khác. Bạn sẽ có các bài học sắp tới trong phần này đi sâu hơn một chút vào các tính năng cụ thể của Aurora mà tôi nghĩ bạn sẽ cần cho kỳ thi, nhưng trong bài học này, tôi chỉ muốn cung cấp **tổng quan ở mức rộng** về những khác biệt giữa Aurora và các engine RDS khác. Vì vậy trong bài demo tiếp theo, bạn sẽ có cơ hội **migrate dữ liệu** từ stack ứng dụng WordPress của chúng ta từ engine RDS MariaDB sang engine Aurora. Bạn sẽ có trải nghiệm tạo một Aurora cluster và tương tác với nó cùng dữ liệu đã migrate. Nhưng đến đây, đó là toàn bộ lý thuyết mà tôi muốn đề cập. Hãy hoàn thành video này, và khi bạn sẵn sàng, tôi rất mong được gặp lại bạn ở bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Aurora là sản phẩm riêng biệt dù thuộc RDS
- Cluster architecture: 1 Primary + 0–15 Replicas
- Shared Cluster Volume (SSD, max 128 TiB)
- Storage replication ở tầng storage (6 bản sao trên 3 AZ)
- Endpoint: Cluster, Reader, Custom, Instance
- Billing: Compute theo giây, Storage theo high-water mark + I/O
- Backtrack & Fast Clone
- Không có Free Tier

**Notes (Chi tiết):**

- **Aurora** dùng kiến trúc **cluster** (không có ở RDS thông thường): 1 Primary instance + tối đa **15 replicas**.
- Replicas vừa dùng cho **read** vừa dùng cho **failover** → kết hợp lợi ích Multi-AZ + Read Replica.
- Storage là **shared cluster volume** (SSD), không gắn local trên instance → tất cả instance cùng truy cập.
- Dữ liệu được **đồng bộ replicate** trên **6 storage nodes** trải 3 AZ. Replication diễn ra ở tầng storage → không tốn CPU/RAM của instance.
- Chỉ Primary được **ghi**; Primary + Replicas đều **đọc** được.
- Storage **tự động sửa chữa** khi có lỗi disk bằng cách dùng dữ liệu từ các bản sao khác.
- Billing storage theo **high-water mark** (vẫn còn áp dụng với nhiều version hiện tại). Có thể tái sử dụng storage đã giải phóng nhưng vẫn tính phí theo mức cao nhất từng đạt.
- **Endpoint**:
    - Cluster endpoint → luôn trỏ Primary (read + write)
    - Reader endpoint → load balance trên tất cả replicas (chỉ read)
    - Có thể tạo Custom endpoint + mỗi instance có endpoint riêng
- **Chi phí**: Không Free Tier. Compute tính theo giây (min 10 phút). Backup miễn phí bằng đúng dung lượng storage đang dùng.
- **Tính năng độc quyền**:
    - **Backtrack**: Roll back dữ liệu tại chỗ về thời điểm trước (không cần tạo cluster mới)
    - **Fast Clone**: Tạo clone gần như tức thì, chỉ lưu delta (thay đổi) so với source

**Summary (Tóm tắt ngắn):** Bài học giới thiệu **kiến trúc Amazon Aurora** – khác biệt căn bản so với RDS thông thường nhờ mô hình **Cluster** (1 Primary + tối đa 15 Replicas) và **Shared Cluster Volume** (SSD, max 128 TiB, 6 bản sao trên 3 AZ). Replicas vừa phục vụ **đọc** vừa phục vụ **failover**. Storage được replicate ở tầng storage nên hiệu quả và bền vững cao. Aurora cung cấp nhiều endpoint linh hoạt, tính phí theo mức tiêu thụ thực tế (có high-water mark), và có các tính năng mạnh như **Backtrack** và **Fast Clone**.