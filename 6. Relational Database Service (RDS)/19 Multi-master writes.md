**Chào mừng trở lại.** Trong bài học này, tôi muốn nói về một tính năng nâng cao của **Amazon Aurora: Multi-Master writes**.

Tính năng này cho phép một **Aurora cluster** có nhiều instance có khả năng thực hiện cả **đọc và ghi**. Điều này khác với chế độ mặc định của Aurora, chỉ cho phép **một writer** và nhiều reader. Chúng ta sẽ bắt đầu bằng việc xem xét kiến trúc.

### Ôn lại chế độ mặc định

Chế độ mặc định của Aurora được gọi là **Single-Master**. Nó tương đương với:

- **Một instance đọc-ghi** (một database instance có thể thực hiện cả thao tác đọc và ghi).
- Cộng thêm **không hoặc nhiều read-only replica**.

Một Aurora cluster chạy ở chế độ **Single-Master** mặc định có một số **endpoint** để tương tác với database:

- **Cluster endpoint**: dùng cho thao tác **đọc hoặc ghi**.
- **Read endpoint**: dùng để **cân bằng tải đọc** trên các read-only replica trong cluster.

Một điểm quan trọng với cluster Single-Master là **failover mất thời gian**. Khi failover xảy ra, một replica phải được **promote** từ chế độ read-only sang read-write.

Trong chế độ **Multi-Master**, tất cả instance mặc định đều có khả năng **cả đọc lẫn ghi**. Vì vậy **không có khái niệm failover kéo dài** nếu một instance trong Multi-Master cluster bị lỗi.

### Kiến trúc Multi-Master ở mức cao

Ở mức cao, một **Multi-Master Aurora cluster** có vẻ tương tự Single-Master:

- Cùng cấu trúc cluster.
- Cùng **shared storage**.
- Nhiều Aurora-provisioned instance cũng tồn tại trong cluster.

**Sự khác biệt bắt đầu từ đây:**

- **Không có cluster endpoint** để sử dụng.
- Ứng dụng phải **tự chịu trách nhiệm kết nối** đến các instance trong cluster.
- **Không có load balancing** giữa các instance trong Multi-Master cluster.
- Ứng dụng kết nối trực tiếp đến **một hoặc tất cả** instance trong cluster và thực hiện thao tác.

**Điểm quan trọng cần hiểu:** Không có khái niệm endpoint được cân bằng tải cho cluster. Ứng dụng có thể khởi tạo kết nối đến một hoặc cả hai instance trong Multi-Master cluster.

![[Pasted image 20260730210338.png]]

### Cách hoạt động của kiến trúc ghi

Khi một **read-write node** trong Multi-Master cluster nhận được yêu cầu ghi từ ứng dụng:

1. Nó **ngay lập tức đề xuất** rằng dữ liệu được commit đến **tất cả storage node** trong cluster.
2. Mỗi node trong cluster sẽ **xác nhận hoặc từ chối** thay đổi được đề xuất.
    - Nó từ chối nếu **xung đột** với một thay đổi đang diễn ra (ví dụ: thay đổi khác từ ứng dụng khác đang ghi vào instance read-write khác trong cluster).

Instance đang ghi sẽ tìm kiếm **quorum** (đa số) các node đồng ý. Khi đạt được quorum, nó có thể **commit** thay đổi vào shared storage. Nếu quorum từ chối → nó **hủy thay đổi** và trả lỗi về ứng dụng.

Giả sử đạt được quorum:

- Thay đổi được **commit** vào storage.
- Được **replicate** trên mọi storage node trong cluster (giống Single-Master).

**Điểm khác biệt lớn nhất:** Trong Multi-Master cluster, thay đổi đó **còn được replicate đến các node khác** trong cluster.

Điều này cho phép các writer khác cập nhật dữ liệu mới vào **in-memory cache** của chúng. Kết quả là mọi thao tác đọc từ bất kỳ instance nào trong cluster đều **nhất quán** với dữ liệu trên shared storage.

Vì các instance **cache dữ liệu**, ngoài việc commit xuống disk, dữ liệu cũng phải được cập nhật trong in-memory cache của các instance khác. Đó chính là mục đích của bước replication này.

### So sánh Failover: Single-Master vs Multi-Master

#### Tình huống Single-Master

- Có **một primary instance** thực hiện đọc + ghi.
- Có **một replica** chỉ thực hiện đọc.
- Bob dùng ứng dụng kết nối qua **cluster endpoint**.
- Cluster endpoint luôn trỏ đến **primary instance**.

Khi **primary instance lỗi**:

- Truy cập vào cluster bị **gián đoạn ngay lập tức**.
- Ứng dụng **không thể fault-tolerant** vì kết nối database bị đứt.
- Cluster nhận ra sự kiện lỗi và chuyển **cluster endpoint** sang replica được chọn làm primary mới.
- Quá trình failover này **mất thời gian** (dù nhanh hơn RDS thông thường vì các replica chia sẻ storage). Việc thay đổi cấu hình không diễn ra tức thì → gây **gián đoạn**.

#### Tình huống Multi-Master

- Cả hai instance đều có thể **ghi** vào shared storage (cả hai đều là writer).
- Ứng dụng có thể kết nối đến **một hoặc cả hai**.
- Giả sử ứng dụng kết nối đến cả hai.

Khi một writer lỗi:

- Ứng dụng có thể **ngay lập tức** chuyển 100% thao tác dữ liệu sang writer còn lại đang hoạt động bình thường.
- **Hầu như không có hoặc rất ít gián đoạn**.

Nếu ứng dụng được thiết kế theo cách này, nó có thể **vận hành xuyên suốt sự cố**. Ứng dụng gần như có thể được mô tả là **fault-tolerant**.

**Aurora Multi-Master cluster** là **một thành phần cần thiết** để xây dựng ứng dụng fault-tolerant. Nó không phải là đảm bảo 100%, nhưng là **nền tảng** để ứng dụng duy trì kết nối đến nhiều writer cùng lúc.

### Lợi ích ở mức cao

- **Tính sẵn sàng tốt hơn và nhanh hơn nhiều**.
- Sự kiện failover có thể được xử lý **bên trong ứng dụng**.
- Không cần gián đoạn traffic giữa ứng dụng và database vì có thể **ngay lập tức** chuyển write sang writer khác.
- Có thể dùng để triển khai **fault tolerance**, nhưng **logic ứng dụng** phải tự load balance giữa các instance — cluster **không tự xử lý** việc này.

---

Đó là tất cả nội dung tôi muốn đề cập trong bài học này. Đây không phải là chủ đề tôi kỳ vọng sẽ xuất hiện chi tiết ngay trên kỳ thi, nên chúng ta giữ ở mức tương đối ngắn gọn.

Hãy hoàn thành video, và khi bạn sẵn sàng, tôi rất mong được gặp bạn ở bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Aurora Multi-Master vs Single-Master
- Không có Cluster Endpoint trong Multi-Master
- Cơ chế Quorum + Replication đến cache
- Failover gần như tức thì nhờ nhiều Writer
- Ứng dụng phải tự quản lý kết nối & load balance

**Notes (Chi tiết):**

- **Single-Master**: Chỉ 1 Writer + nhiều Reader. Failover phải promote replica → mất thời gian.
- **Multi-Master**: Tất cả instance đều là **read-write**. Không có cluster endpoint. Ứng dụng tự kết nối trực tiếp đến từng instance.
- Khi ghi: Instance đề xuất → cần **quorum** đồng ý → commit vào shared storage → **replicate** sang cache của các instance khác để đảm bảo đọc nhất quán.
- Lợi ích lớn nhất: Failover gần như **không gián đoạn** vì ứng dụng có thể chuyển traffic ngay sang writer còn sống.
- Multi-Master là **nền tảng** để xây dựng ứng dụng fault-tolerant, nhưng logic load balancing nằm ở phía ứng dụng.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu tính năng **Aurora Multi-Master** cho phép nhiều instance cùng **đọc + ghi**. Khác với Single-Master (chỉ 1 Writer), Multi-Master không dùng cluster endpoint và dựa vào quorum + replication cache. Ưu điểm chính là **failover cực nhanh** gần như không gián đoạn, giúp xây dựng ứng dụng fault-tolerant (nhưng ứng dụng phải tự quản lý kết nối).