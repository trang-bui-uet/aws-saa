Chào mừng bạn quay trở lại! Đây là **phần hai** của bài học. Chúng ta sẽ tiếp tục ngay từ nơi phần một kết thúc.

Tiếp theo, hãy cùng xem về **Reserved Instances**. Đây là một phần rất quan trọng trong hầu hết các triển khai lớn trên AWS. Trong khi **On-Demand** thường được dùng cho nhu cầu không xác định hoặc ngắn hạn và không chịu được gián đoạn, thì **Reserved Instances** dành cho nhu cầu sử dụng **dài hạn** và **ổn định** của EC2.

![[Pasted image 20260525232113.png|689]]

**Reservations** (đặt trước) khá dễ hiểu. Đây là cam kết bạn đưa ra với AWS để sử dụng lâu dài tài nguyên EC2. Giả sử đây là một instance T3 bình thường. Nếu bạn sử dụng instance này mà không có reservation nào áp dụng, bạn sẽ bị tính phí theo mức giá **On-Demand** (tính theo giây).

Nhưng nếu bạn biết chắc instance này sẽ sử dụng **dài hạn** và là thành phần cốt lõi của hệ thống, bạn có thể mua một **reservation**. Hiệu quả của reservation là **giảm mạnh** (hoặc loại bỏ hoàn toàn) chi phí theo giây cho instance đó, miễn là reservation khớp với instance.

**Lưu ý quan trọng**: Bạn cần lập kế hoạch reservation thật cẩn thận, vì nếu mua reservation mà không sử dụng, bạn **vẫn phải trả tiền** cho reservation đó nhưng không nhận được lợi ích.

Reservation có thể được mua cho một loại instance cụ thể và **khóa theo Availability Zone** hoặc **theo Region**:

- Nếu khóa theo **Availability Zone**: Chỉ áp dụng cho instance trong AZ đó, nhưng **được ưu tiên capacity** (sẽ nói chi tiết ở bài khác).
- Nếu mua theo **Region**: Không ưu tiên capacity, nhưng có thể áp dụng cho bất kỳ AZ nào trong Region đó.

Reservation cũng có thể có **hiệu ứng một phần**. Ví dụ: Bạn có reservation cho t3.large, nhưng chạy một instance lớn hơn → reservation sẽ áp dụng một phần và vẫn giảm giá cho phần tương ứng.

---

**Tóm tắt Reserved Instances**: Bạn cam kết với AWS sẽ sử dụng tài nguyên trong một khoảng thời gian dài để đổi lấy **giá rẻ hơn đáng kể**. Tuy nhiên, **một khi đã cam kết, bạn phải trả tiền dù có dùng hay không**. Vì vậy chỉ nên dùng Reserved Instances cho những thành phần **luôn chạy ổn định**, **không thay đổi** trong hạ tầng.

### Các lựa chọn thanh toán cho Reserved Instances:

1. **No Upfront** (Không trả trước): Cam kết 1 hoặc 3 năm, trả phí theo giây **giảm** (dù instance có chạy hay không). Dễ quản lý dòng tiền, nhưng **mức giảm giá thấp nhất**.
2. **All Upfront** (Trả hết trước): Trả toàn bộ chi phí cho 1 hoặc 3 năm ngay từ đầu. Không còn phí theo giây nữa. Đây là cách **tiết kiệm nhất** (đặc biệt là cam kết 3 năm + All Upfront).
3. **Partial Upfront** (Trả một phần trước): Trả một khoản tiền ban đầu nhỏ hơn, kết hợp với phí theo giây thấp hơn No Upfront. Đây là **lựa chọn cân bằng tốt** về chi phí và dòng tiền.

**Reserved Instances** rất phù hợp cho các thành phần cần **chi phí thấp nhất**, **sử dụng ổn định lâu dài** và **không chịu được gián đoạn**.

---

Tiếp theo là **Dedicated Hosts**.

![[Pasted image 20260525232158.png]]

**Dedicated Host** là một máy chủ EC2 vật lý được **dành riêng hoàn toàn cho bạn**. Bạn trả tiền cho cả host (không phải từng instance). Host này được thiết kế cho một dòng instance cụ thể (ví dụ: T3 hoặc M5).

**Lý do sử dụng Dedicated Hosts**:

- Đáp ứng **yêu cầu tuân thủ quy định nghiêm ngặt** (không được chia sẻ phần cứng vật lý với khách hàng khác).
- **Cấp phép phần mềm phức tạp** (bring-your-own-license): Nhiều phần mềm enterprise yêu cầu license gắn với socket, core hoặc host vật lý. Dedicated Host cho phép bạn kiểm soát vị trí instance và nhận báo cáo socket/core từ AWS.

**Dedicated Host** thường là cách **đắt nhất** để dùng EC2, nhưng có thể trở nên **rẻ hơn nhiều** nếu bạn mang license của mình (hệ điều hành, database enterprise…).

---

**Dedicated Instances** (Dễ nhầm lẫn với Dedicated Host):

![[Pasted image 20260525232231.png]]

**Dedicated Instances** là các instance chạy trên phần cứng **dành riêng cho một khách hàng** trong VPC. Bạn không chia sẻ host với khách hàng AWS khác.

**Sự khác biệt quan trọng**:

- Với **Dedicated Host**: Bạn mua host, kiểm soát vị trí instance, thấy socket/core → hỗ trợ license phức tạp.
- Với **Dedicated Instances**: Bạn **không kiểm soát** vị trí instance, không thấy socket/core → **chỉ dùng cho mục đích tuân thủ**, không dùng cho license yêu cầu chi tiết phần cứng.

---

Cuối cùng là **Spot Instances**.

**Spot Instances** là cách AWS bán **dung lượng dư thừa** với **giá cực rẻ** (có thể giảm đến **90%** so với On-Demand).

**Nhược điểm**: Nếu AWS cần dung lượng đó cho khách On-Demand hoặc Reserved, họ sẽ **ngắt instance của bạn** chỉ với **2 phút cảnh báo**.

**Spot Instances** phù hợp cho các workload:

- **Fault-tolerant** (chịu lỗi)
- **Stateless** (không lưu trạng thái)
- Ví dụ: Batch processing, phân tích dữ liệu nền, rendering, testing…

---

**Tóm tắt các lựa chọn mua EC2**:

- **On-Demand**: Linh hoạt, ngắn hạn, giá cao.
- **Reserved Instances**: Dài hạn, ổn định, giá rẻ.
- **Dedicated Hosts / Dedicated Instances**: Tuân thủ quy định & license.
- **Spot Instances**: Giá rẻ nhất, chịu được gián đoạn.

---

**Phần Gợi Nhớ (Cues / Recall)** _(Chỉ gồm câu hỏi + keywords – KHÔNG có câu trả lời bên cạnh)_
- Reserved Instances là gì và dùng khi nào?
- Lợi ích chính của Reserved Instances?
- Term length của reservations (1-year hay 3-year)?
- Payment structures cho Reserved Instances?
- AZ-specific vs Regional Reservation khác nhau ra sao?
- Rủi ro khi mua reservations?
- Dedicated Hosts là gì?
- Lý do chính sử dụng Dedicated Hosts?
- Dedicated Instances khác Dedicated Hosts ở điểm nào?
- Spot Instances hoạt động như thế nào?
- Ưu điểm và nhược điểm của Spot Instances?
- Tóm tắt 4 purchasing options chính của EC2?

---

**Tóm tắt (Summary)**

Reserved Instances là hình thức cam kết sử dụng dài hạn EC2 (1 năm hoặc 3 năm) nhằm giảm đáng kể chi phí cho các workload ổn định và liên tục. Bạn có thể chọn thanh toán No Upfront, Partial Upfront hoặc All Upfront (All Upfront cho mức giảm giá cao nhất).

Dedicated Hosts là máy chủ vật lý được cấp riêng hoàn toàn cho một khách hàng, phù hợp với nhu cầu tuân thủ quy định nghiêm ngặt hoặc licensing phức tạp (BYOL), vì cho phép kiểm soát placement và xem thông tin socket/core.

Dedicated Instances là các instance chạy trên phần cứng dành riêng, nhưng không có quyền kiểm soát host hay visibility socket/core.

Spot Instances cho phép mua công suất dư thừa của AWS với giá rất rẻ (giảm đến 90%), đổi lại instance có thể bị terminate bất cứ lúc nào chỉ với 2 phút cảnh báo. Đây là lựa chọn lý tưởng cho workload fault-tolerant và stateless.

Tổng kết: Chọn On-Demand cho nhu cầu ngắn hạn linh hoạt, Reserved cho dài hạn ổn định, Dedicated (Hosts/Instances) cho compliance và licensing, Spot cho chi phí thấp nhất với workload chịu được gián đoạn.