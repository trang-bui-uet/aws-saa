Chào mừng bạn quay lại. Trong bài học này, tôi muốn giới thiệu nhanh về sản phẩm **Aurora Global Database**.

Tên gọi đã phần nào thể hiện chức năng của nó. Để tránh nhầm lẫn: **Global Database** cho phép bạn tạo **replication ở cấp độ toàn cầu** bằng Aurora từ một **vùng chính (primary region)** đến tối đa **5 vùng phụ (secondary regions)**.

Đây là kiến thức bạn chỉ cần **nắm khái niệm** cho kỳ thi. Tôi không kỳ vọng nó xuất hiện nhiều, nhưng bạn cần hiểu rõ chức năng mà Aurora Global Database cung cấp. Tôi sẽ giữ bài học ngắn gọn. Hãy cùng xem kiến trúc trước.

### Kiến trúc phổ biến

Đây là một kiến trúc điển hình khi sử dụng Aurora Global Database. Môi trường hoạt động trên **hai hoặc nhiều vùng**.

- **Vùng chính (Primary Region)**: Ví dụ **US East 1** (bên trái). Vùng này hoạt động giống như một **Aurora cluster thông thường**:
    - Có **1 instance Read/Write**
    - Có thể có tối đa **15 Read Replica**
- **Vùng phụ (Secondary Region)**: Ví dụ **AP Southeast 2** (Sydney – bên phải).
    - Có thể có tối đa **16 replica**
    - **Toàn bộ cluster vùng phụ là Read Only** trong điều kiện hoạt động bình thường.
    - Tất cả 16 replica đều là Read Replica.

**Replication** từ vùng chính sang các vùng phụ diễn ra ở **lớp lưu trữ (storage layer)**. Thời gian replication thường **dưới 1 giây**.

Ứng dụng có thể:

- Ghi dữ liệu vào **Primary instance** ở vùng chính
- Đọc dữ liệu từ **replica** ở vùng chính hoặc các vùng phụ

### Khi nào nên sử dụng Aurora Global Database?

**1. Disaster Recovery và Business Continuity xuyên vùng**

Aurora Global Database rất phù hợp cho **khôi phục thảm họa xuyên vùng** và **duy trì hoạt động liên tục**.

Bạn có thể tạo Global Database với nhiều vùng phụ. Nếu xảy ra sự cố ảnh hưởng đến toàn bộ một vùng AWS, bạn có thể **promote** các cluster vùng phụ lên thành Primary để chúng có thể thực hiện cả **Read và Write**.

Nhờ thời gian replication **~1 giây**, cả **RPO** và **RTO** đều rất thấp khi thực hiện failover xuyên vùng.

**2. Global Read Scaling (Mở rộng đọc toàn cầu)**

Nếu bạn muốn cung cấp **độ trễ thấp** cho khách hàng quốc tế (độ trễ thấp = hiệu năng tốt), bạn có thể tạo nhiều vùng phụ được replicate từ vùng chính. Ứng dụng đặt tại các vùng phụ chỉ cần thực hiện **đọc** trên các cluster phụ → mang lại trải nghiệm hiệu năng tốt cho người dùng.

### Các điểm quan trọng cần nhớ

- Replication diễn ra ở **storage layer**, thường **≤ 1 giây** từ Primary đến tất cả Secondary.
- Đây là **replication một chiều** (Primary → Secondary), **không phải hai chiều**.
- Replication **không ảnh hưởng** đến hiệu năng database vì nó xảy ra ở lớp lưu trữ → **không tốn thêm CPU**.
- Mỗi vùng phụ có thể có tối đa **16 replica** (vì không có Primary Read/Write nên tất cả đều là Read Replica).
- Tất cả replica trong vùng phụ đều có thể được **promote** thành Read/Write khi xảy ra sự cố.
- Hiện tại tối đa **5 vùng phụ** (số lượng này có thể thay đổi trong tương lai như hầu hết các giới hạn của AWS).

Đối với kỳ thi, sản phẩm này **không xuất hiện nhiều**, nhưng bạn cần có nhận thức cơ bản về kiến trúc và use case để khi gặp câu hỏi hoặc khi triển khai thực tế sẽ có điểm bắt đầu vững chắc.

Đó là toàn bộ nội dung lý thuyết của bài học này. Hãy hoàn thành video và khi sẵn sàng, tôi sẽ gặp bạn ở bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Aurora Global Database là gì
- Kiến trúc Primary vs Secondary Region
- Replication ở storage layer (~1 giây)
- Use case: Cross-region DR & Global Read Scaling
- Giới hạn: 5 Secondary Region, 16 replica/vùng phụ
- Replication một chiều, không ảnh hưởng performance

**Notes (Chi tiết):**

- **Global Database** cho phép replicate Aurora từ **1 Primary Region** đến tối đa **5 Secondary Region**.
- **Primary Region**: 1 Read/Write + tối đa 15 Read Replica.
- **Secondary Region**: Toàn bộ cluster là **Read Only**, tối đa **16 replica**.
- Replication diễn ra ở **storage layer**, thời gian thường **≤ 1 giây**, **một chiều**, **không tốn CPU**.
- **Use case chính**:
    - **Disaster Recovery / Business Continuity**: Promote Secondary thành Primary khi Primary Region bị sự cố → RPO/RTO thấp.
    - **Global Read Scaling**: Đặt ứng dụng ở Secondary Region để phục vụ đọc với độ trễ thấp cho khách hàng quốc tế.
- Tất cả replica ở Secondary đều có thể được promote thành Read/Write khi cần.

**Summary (Tóm tắt ngắn):** Aurora Global Database cung cấp **replication toàn cầu** từ 1 Primary Region đến tối đa 5 Secondary Region với độ trễ ~1 giây ở lớp lưu trữ. Phù hợp cho **Disaster Recovery xuyên vùng** (RPO/RTO thấp) và **mở rộng đọc toàn cầu** với hiệu năng cao. Secondary Region hoàn toàn Read Only (tối đa 16 replica) và có thể được promote khi cần.