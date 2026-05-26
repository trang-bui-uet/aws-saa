Bài học gồm hai phần này sẽ đi sâu vào các tùy chọn mua EC2 khác nhau, giải thích cơ chế hoạt động và các tình huống phù hợp để sử dụng từng loại.
- **On-Demand** (Theo nhu cầu)
- **Spot** (Spot - Giá theo đấu giá)
- **Reserved** (Reserved - Đặt trước)
- **Dedicated Instance** (Dedicated Instance - Instance chuyên dụng)
- **Dedicated Host** (Dedicated Host - Máy chủ chuyên dụng)


Welcome back và trong bài học này, tôi muốn nói về **EC2 purchase options**. **EC2 purchase options** thường được gọi là launch types, nhưng cách gọi chính thức của AWS là **purchase options**, vì vậy để nhất quán, chúng ta sẽ tập trung vào tên này. Vậy là **EC2 purchase options**.

Hãy cùng tìm hiểu tất cả các loại chính, tập trung vào tình huống nên và không nên sử dụng từng loại. Bắt đầu ngay nào.

Loại purchase option đầu tiên là mặc định: **On-Demand**. **On-Demand** rất đơn giản vì nó hoàn toàn bình thường, không có gì nổi bật. Đây là lựa chọn mặc định vì nó là mức trung bình, không có ưu hay nhược điểm đặc biệt.
![[Pasted image 20260525223911.png|309]]

Cách hoạt động: Hãy tưởng tượng hai EC2 hosts. Các instances có kích thước khác nhau khi launch bằng **On-Demand** sẽ chạy trên các hosts này. Instances của nhiều khách hàng AWS khác nhau được trộn lẫn trên cùng một pool phần cứng. Dù instances được cách ly và bảo vệ, nhưng chúng vẫn chia sẻ pool phần cứng chung. Nhờ đó AWS có thể phân bổ tài nguyên hiệu quả, nên giá khởi điểm của **On-Demand** khá hợp lý.

Về giá cả, **On-Demand** sử dụng **per-second billing** khi instance đang chạy. Bạn chỉ trả tiền cho tài nguyên khi instance ở trạng thái **running**. Khi shutdown instance, bạn không phải trả tiền cho compute nữa. Tuy nhiên, các dịch vụ liên quan như storage vẫn tính phí liên tục dù instance có chạy hay không.

**On-Demand** nên sử dụng trong những trường hợp nào? Đây là lựa chọn mặc định, vì vậy bạn nên **luôn bắt đầu đánh giá bằng On-Demand** và chỉ chuyển sang loại khác khi có lý do chính đáng. Với **On-Demand**, **không có gián đoạn**, bạn launch instance, trả phí theo giây, và instance sẽ chạy cho đến khi bạn dừng lại. Tuy nhiên, **On-Demand không có capacity reservation**. Trong trường hợp AWS bị sự cố lớn và thiếu capacity, các instance dùng Reserved Instance sẽ được ưu tiên. Do đó, nếu workload rất quan trọng với business, bạn nên cân nhắc lựa chọn khác.

**On-Demand** có giá dự đoán được, ổn định, không có discount đặc biệt. Nó phù hợp cho **short-term workloads**, những công việc chỉ cần chạy một lần rồi terminate. Nếu bạn không chắc về thời lượng hoặc loại workload, **On-Demand** là lựa chọn lý tưởng. Đặc biệt phù hợp với short-term hoặc unknown workloads mà **không thể chịu được gián đoạn**.

Tiếp theo là **Spot Instances** – cách rẻ nhất để sử dụng EC2 capacity.

**Spot** hoạt động bằng cách AWS bán dung lượng dư thừa với giá giảm mạnh. Trong mỗi region, với mỗi loại instance, AWS theo dõi dung lượng trống và đưa ra **spot price**. Bạn đặt **maximum price**, nhưng thực tế chỉ trả theo spot price hiện tại. Nếu spot price vượt quá maximum price bạn đặt, **Spot Instances sẽ bị terminate ngay lập tức**.

**Spot** có thể rẻ hơn tới **90%** so với On-Demand, nhưng có trade-off lớn: **có thể bị gián đoạn bất cứ lúc nào**.

**Không nên dùng Spot** cho các workload không chịu được gián đoạn như: domain controllers, mail servers, traditional websites, flight control systems, hay bất kỳ hệ thống business-critical nào.

**Nên dùng Spot** cho: highly parallel workloads (có thể chia nhỏ và rerun), scientific analysis, media processing, image processing, bursty capacity needs, cost-sensitive workloads, và các ứng dụng **stateless** (không lưu state trên instance).

Tóm lại, **Spot** là lựa chọn tuyệt vời cho workload có thể chịu gián đoạn, không time-critical, và muốn tiết kiệm chi phí tối đa. Ngược lại, nó là **anti-pattern** cho workload dài hạn, cần độ tin cậy cao.

---

**Tóm tắt** (Summary)
**EC2 Purchase Options** gồm hai loại chính:

- **On-Demand**: Lựa chọn mặc định, giá ổn định, **không bị gián đoạn**, phù hợp cho workload ngắn hạn và **không chịu được gián đoạn**.
- **Spot Instances**: Rẻ nhất (tiết kiệm đến **90%**), nhưng **có thể bị terminate bất cứ lúc nào**, chỉ phù hợp với workload **stateless**, parallel, bursty và **chịu được gián đoạn**.

**Quy tắc vàng**: Bắt đầu mọi thứ bằng **On-Demand**, chỉ dùng **Spot** khi muốn tiết kiệm chi phí mạnh và workload cho phép bị ngắt quãng.

---

**Gợi Nhớ (Cue)**
- On-Demand là gì?
- On-Demand hoạt động như thế nào?
- Ưu điểm của On-Demand là gì?
- Nhược điểm của On-Demand là gì?
- Khi nào nên dùng On-Demand?
- Spot Instances là gì?
- Spot Instances hoạt động ra sao?
- Rủi ro lớn nhất của Spot là gì?
- Khi nào nên dùng Spot Instances?
- Khi nào không nên dùng Spot Instances?
- So sánh On-Demand và Spot?
