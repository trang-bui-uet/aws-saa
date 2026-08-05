**ICMP (Internet Control Message Protocol)** là protocol dùng để gửi thông điệp kiểm soát/chẩn đoán mạng — không dùng để truyền dữ liệu ứng dụng như HTTP.

Ví dụ quen thuộc nhất: lệnh **`ping`** dùng ICMP để hỏi “máy kia còn sống / có phản hồi không?”.

Trong ngữ cảnh **NLB health check**:
- Kiểm tra **ICMP** ≈ máy/instance có reachable ở mức mạng không
- Kiểm tra **TCP handshake** ≈ cổng TCP có mở và chấp nhận kết nối không

Cả hai đều chỉ nói “mạng/cổng ổn”, **không** kiểm tra ứng dụng có chạy đúng (như HTTP 200) — đó là điểm khác với health check Layer 7 của ALB.