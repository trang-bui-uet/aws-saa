**Chào mừng bạn quay trở lại.** Trong bài học này, tôi muốn trình bày hai chủ đề tối ưu hóa EC2 quan trọng: **Enhanced Networking** và **EBS Optimized Instances**. Cả hai đều mang lại lợi ích lớn cho hiệu suất của EC2 và hỗ trợ các tính năng hiệu suất cao khác như Placement Groups. Là Solutions Architect, việc hiểu rõ kiến trúc và lợi ích của chúng là rất cần thiết.

---

**Enhanced Networking**

**Enhanced Networking** là tính năng được thiết kế để cải thiện hiệu suất mạng tổng thể của EC2. Đây là tính năng bắt buộc cho các tính năng hiệu suất cao như **Cluster Placement Groups**.

**Enhanced Networking** sử dụng kỹ thuật gọi là **SR-IOV** (Single Root I/O Virtualization). Ở mức cao, nó giúp card mạng vật lý bên trong EC2 host nhận biết được ảo hóa.

**Không có Enhanced Networking:**

- Card mạng vật lý không nhận biết ảo hóa.
- Host phải đứng ở giữa, điều khiển từng instance truy cập card mạng theo thời gian.
- Quá trình diễn ra ở phần mềm → chậm hơn, tiêu tốn nhiều CPU của host.
- Khi host chịu tải nặng, dễ xảy ra giảm hiệu suất, tăng độ trễ và thay đổi băng thông.

**Với Enhanced Networking (SR-IOV):**

- Card mạng vật lý nhận biết được ảo hóa.
- Thay vì chỉ có một card vật lý, nó cung cấp nhiều **logical cards** trên một card vật lý.
- Mỗi instance được cấp quyền truy cập độc quyền vào một logical card.
- Card mạng vật lý xử lý toàn bộ quá trình mà không tiêu tốn nhiều CPU của host.

**Lợi ích chính của Enhanced Networking:**

- Tăng **I/O** tổng thể trên host và giảm tải CPU host.
- Tăng đáng kể **băng thông mạng** (higher bandwidth).
- Tăng **Packets Per Second (PPS)** — rất tốt cho ứng dụng cần xử lý nhiều gói tin nhỏ.
- Giảm **độ trễ** và đặc biệt là **độ trễ ổn định** (consistent low latency).

**Enhanced Networking** được bật mặc định hoặc miễn phí trên hầu hết các loại instance EC2 hiện đại. Đối với kỳ thi Solutions Architect Associate, bạn không cần biết chi tiết cách bật/tắt, chỉ cần hiểu lợi ích và khi nào nên dùng.

---

**EBS Optimized Instances**

Một instance có được **EBS Optimized** hay không phụ thuộc vào tùy chọn được bật/tắt trên từng instance.

**Bối cảnh:** EBS là block storage được cung cấp qua mạng. Trước đây, networking trên EC2 chia sẻ chung stack mạng cho cả traffic dữ liệu thông thường và traffic EBS. Điều này gây ra tranh chấp và giới hạn hiệu suất.

Khi instance được **EBS Optimized**:

- Đã có các tối ưu hóa stack.
- Có **dedicated capacity** riêng cho traffic EBS.
- Giúp EBS đạt tốc độ nhanh hơn.
- Traffic EBS không ảnh hưởng đến hiệu suất mạng dữ liệu thông thường và ngược lại.

Trên hầu hết các instance hiện đại, **EBS Optimized** được hỗ trợ và **bật mặc định miễn phí**. Việc tắt nó cũng không có tác dụng vì khả năng này đã được tích hợp sẵn vào phần cứng.

**EBS Optimization** đặc biệt quan trọng với các instance type có hiệu suất cao, cần **throughput** và **IOPS** lớn — đặc biệt khi sử dụng volume loại **gp2** và **io1** (yêu cầu độ trễ thấp và ổn định).

Tóm lại, **EBS Optimized** đơn giản là việc cung cấp dung lượng mạng riêng biệt cho storage networking trên EC2 instance.

---

**Kết luận:**

Cả **Enhanced Networking** (SR-IOV) và **EBS Optimized** đều là các tính năng tối ưu hóa quan trọng giúp EC2 đạt hiệu suất cao hơn về mạng và lưu trữ. Chúng thường được bật sẵn trên các instance hiện đại mà không mất thêm chi phí.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Enhanced Networking = SR-IOV
- Lợi ích: Higher bandwidth, Higher PPS, Lower & consistent latency, Less host CPU
- Bắt buộc cho Cluster Placement Groups
- EBS Optimized: Dedicated capacity cho EBS traffic
- Cả hai đều bật mặc định miễn phí trên instance hiện đại

**Notes (Chi tiết):**

- **Enhanced Networking (SR-IOV)**: Giúp card mạng vật lý nhận biết ảo hóa → mỗi instance có logical card riêng. Giảm tải CPU host, tăng băng thông, tăng PPS, giảm độ trễ ổn định. Rất quan trọng cho workload hiệu suất cao và Cluster Placement Groups.
- **EBS Optimized**: Cung cấp dedicated network capacity cho EBS, tách biệt traffic EBS và traffic dữ liệu thông thường. Cải thiện throughput, IOPS và độ trễ của EBS. Bật mặc định trên hầu hết instance hiện đại.
- Cả hai tính năng đều giúp EC2 đạt hiệu suất cao hơn mà không tốn thêm chi phí trên các instance loại mới.

**Summary (Tóm tắt ngắn):** Bài học trình bày hai tính năng tối ưu hóa EC2 quan trọng:

- **Enhanced Networking** (dùng SR-IOV) cải thiện hiệu suất mạng tổng thể: tăng băng thông, PPS, giảm độ trễ ổn định và giảm tải CPU host.
- **EBS Optimized** cung cấp dung lượng mạng riêng cho EBS, tránh xung đột với traffic dữ liệu thông thường, giúp EBS đạt hiệu suất cao hơn (đặc biệt với gp2/io1).

Cả hai đều được bật mặc định miễn phí trên các instance EC2 hiện đại.