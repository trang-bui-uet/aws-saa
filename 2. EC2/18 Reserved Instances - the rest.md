**Phần 1: Giới thiệu về Standard Reserved Instances**

Chào mừng bạn quay lại! Trong bài học trước, tôi đã nói về các loại launch của **EC2**, và một trong những loại đó là reserved. Về mặt lịch sử, chỉ có một loại tùy chọn mua reserved, nhưng theo thời gian, **AWS** đã thêm các tùy chọn mới linh hoạt hơn, và vì vậy **reserved** đã trở thành được biết đến như **Standard Reserved**.

Như tôi đã nói trong bài học trước, đây là lựa chọn tuyệt vời cho nhu cầu sử dụng đã biết, **dài hạn** và **ổn định**. Nếu bạn cần quyền truy cập vào **EC2** chạy **24/7, 365 ngày** với giá rẻ nhất trong **một hoặc ba năm**, thì bạn sẽ chọn **Standard Reserved**. Nhưng bạn còn có một số lựa chọn linh hoạt khác.

---

**Phần 2: Scheduled Reserved Instances**

![[Pasted image 20260527001228.png]]

**Scheduled Reserved Instances** khá tình huống cụ thể, nhưng khi gặp phải những tình huống đó, chúng mang lại lợi thế lớn. Chúng rất phù hợp khi bạn có yêu cầu **dài hạn**, nhưng không cần chạy liên tục.

Ví dụ: Bạn có quy trình xử lý batch cần chạy **hàng ngày 5 giờ**. Đây là nhu cầu đã biết và dài hạn, nhưng **Standard Reserved Instance** sẽ không phù hợp vì không chạy 24/7.

**Scheduled Reserved Instance** là một cam kết: bạn chỉ định **tần suất**, **thời lượng** và **thời gian** (ví dụ: 23:00 hàng ngày trong 5 giờ). Bạn đặt trước dung lượng và nhận được giá **rẻ hơn On-Demand**, nhưng chỉ được sử dụng trong khung thời gian đã đặt.

Các tình huống khác: phân tích doanh số chạy **mỗi thứ Sáu 24 giờ**, hoặc quy trình phân tích lớn cần **100 giờ EC2 mỗi tháng**.

**Hạn chế**: Không hỗ trợ tất cả instance types và regions, yêu cầu tối thiểu **1.200 giờ/năm**, và cam kết tối thiểu **1 năm**.

Đây là lựa chọn lý tưởng cho nhu cầu sử dụng **dài hạn, đã biết, ổn định** nhưng **không chạy toàn thời gian**.

---

**Phần 3: Capacity Reservations**

![[Pasted image 20260527001254.png]]

**Capacity Reservations** rất hữu ích khi bạn có quy trình **quan trọng**, **không chịu được gián đoạn**, và cần đảm bảo có dung lượng khi cần.

AWS ưu tiên cung cấp dung lượng theo thứ tự sau:

- **1.** Cam kết từ Reserved Instances
- **2.** Yêu cầu On-Demand
- **3.** Spot Instances

**Capacity Reservation** khác với Reserved Instance ở chỗ có **2 thành phần**:

- **Thành phần thanh toán** (billing)
- **Thành phần dung lượng** (capacity)

---

**Phần 4: Regional vs Zonal Reservations**

- **Regional Reservation**: Linh hoạt, áp dụng cho toàn Region, được giảm giá thanh toán khi launch instance ở bất kỳ AZ nào. Tuy nhiên **không reserve capacity** cụ thể ở AZ nào.
- **Zonal Reservation**: Giảm giá thanh toán + **reserve capacity** thực sự trong **một AZ cụ thể**. Nếu launch ở AZ khác thì không được lợi ích.

Cả hai đều yêu cầu cam kết **1 hoặc 3 năm**.

---

**Phần 5: On-Demand Capacity Reservations**

Khi không muốn cam kết dài hạn 1-3 năm nhưng vẫn cần đảm bảo dung lượng, bạn sử dụng **On-Demand Capacity Reservations**.

Bạn đặt trước dung lượng trong **AZ cụ thể** và **phải trả tiền dù có sử dụng hay không**. Không có giảm giá, chỉ reserve capacity.

Bạn cần lập kế hoạch chính xác vì dung lượng không dùng vẫn bị tính phí.

---

**Phần 6: Savings Plans**

![[Pasted image 20260527001336.png]]

**Savings Plans** giống như Reserved Instance nhưng linh hoạt hơn. Bạn cam kết **chi tiêu theo giờ** (ví dụ: $20/giờ) trong 1 hoặc 3 năm để nhận giảm giá.

Có 2 loại chính:

1. **Compute Savings Plans**: Áp dụng cho **EC2, Fargate, Lambda**, tiết kiệm **lên đến 66%**.
2. **EC2 Instance Savings Plans**: Chỉ dành cho **EC2**, tiết kiệm **lên đến 72%**.

**Cách hoạt động**: Bạn được áp dụng giá Savings Plan đến mức cam kết mỗi giờ. Vượt quá sẽ tính giá On-Demand.

**Lợi ích thực tế**: Rất phù hợp cho tổ chức đang chuyển dần từ EC2 sang Container (Fargate) và Serverless (Lambda).

---

**Kết thúc bài học**

Với những điều đã nói, đó là tất cả những gì tôi muốn đề cập trong bài học lý thuyết này. Hãy hoàn thành bài học, và khi bạn sẵn sàng, tôi mong chờ được gặp bạn trong bài tiếp theo.