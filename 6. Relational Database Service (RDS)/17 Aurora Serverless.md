**Chào mừng trở lại!** Trong bài học này, mình sẽ nói về **Aurora Serverless**.

**Aurora Serverless** là dịch vụ đối với Aurora giống như **Fargate** đối với ECS. Nó cung cấp một phiên bản của sản phẩm cơ sở dữ liệu Aurora mà bạn **không cần phải provision cố định** các database instance với kích thước nhất định, cũng như **không cần lo quản lý** các database instance đó. Đây là một bước tiến gần hơn tới mô hình **database-as-a-service**; nó loại bỏ thêm một phần overhead quản trị — đó là overhead quản lý từng database instance riêng lẻ.

Từ giờ trở đi, khi nhắc đến các sản phẩm Aurora mà chúng ta đã học trước đó trong khóa học, bạn nên gọi là **Aurora Provisioned** để phân biệt với **Aurora Serverless** (nội dung bài học này).

### Cách hoạt động cơ bản của Aurora Serverless

Với **Aurora Serverless**, bạn **không cần provision tài nguyên** theo cách giống **Aurora Provisioned**. Bạn vẫn tạo một **cluster**, nhưng Aurora Serverless sử dụng khái niệm **ACU** (Aurora Capacity Units).

- Mỗi **ACU** đại diện cho một lượng **compute** nhất định và lượng **memory** tương ứng.
- Với mỗi cluster, bạn có thể thiết lập **giá trị minimum và maximum**.
- Aurora Serverless sẽ **tự động scale** giữa các giá trị này, tăng hoặc giảm capacity dựa trên tải thực tế của cluster.
- Thậm chí nó có thể **giảm xuống 0 và pause**, nghĩa là lúc đó bạn **chỉ bị tính phí phần storage** mà cluster đang sử dụng.
- **Billing** được tính dựa trên tài nguyên bạn thực sự sử dụng theo **đơn vị giây (per-second)**.
- Aurora Serverless vẫn cung cấp mức **resilience tương đương** Aurora Provisioned: cluster storage được **replicate trên 6 storage node** trải rộng nhiều Availability Zone.

### Các lợi ích cấp cao của Aurora Serverless

- **Đơn giản hơn nhiều**: Loại bỏ phần lớn sự phức tạp khi quản lý database instance và capacity.
- **Dễ scale hơn**: Tự động scale compute và memory dưới dạng ACU khi cần, **không gây gián đoạn** kết nối của client.
- **Tiết kiệm chi phí**: Bạn chỉ trả tiền cho tài nguyên database thực sự sử dụng theo giây, khác với Aurora Provisioned (bạn phải provision trước và vẫn bị tính phí dù có dùng hết hay không).

### Kiến trúc của Aurora Serverless

Kiến trúc Aurora Serverless có nhiều điểm **tương đồng** với Aurora Provisioned, nhưng cũng có những **khác biệt quan trọng**.

**Điểm giống nhau:**

- Vẫn tồn tại **Aurora cluster architecture**, nhưng dưới dạng **Aurora Serverless cluster**.
- Vẫn sử dụng **cùng cluster volume architecture** như Aurora Provisioned.

**Điểm khác biệt:**

- Thay vì dùng **provisioned server**, Aurora Serverless dùng **ACU** (Aurora Capacity Units).
- Các ACU được cấp phát từ một **warm pool** do AWS quản lý.
- ACU là **stateless**, được chia sẻ giữa nhiều khách hàng AWS, và **không có local storage** → có thể được cấp phát rất nhanh khi cần.
- Khi ACU được gán cho cluster, chúng truy cập vào **cluster storage** giống hệt như instance trong Aurora Provisioned.

**Cơ chế scale:**

- Khi tải tăng vượt quá số ACU đang dùng (và vẫn nằm trong giới hạn maximum), hệ thống sẽ cấp thêm ACU.
- Khi ACU mới đã active, các compute resource cũ không còn cần thiết sẽ được thu hồi.

**Quản lý kết nối (điểm quan trọng):**

- Do số lượng ACU thay đổi động, cách quản lý connection phức tạp hơn so với cluster provisioned.
- Aurora Serverless sử dụng một **shared proxy fleet** do AWS quản lý (hoàn toàn transparent với người dùng).
- Ứng dụng client **không kết nối trực tiếp** vào ACU, mà đi qua proxy fleet.
- Proxy sẽ broker connection giữa application và ACU.
- Nhờ vậy, việc **scale in/out diễn ra mượt mà**, **không gây gián đoạn** kết nối của ứng dụng.

Bạn chỉ cần quan tâm đến việc chọn **minimum và maximum ACU**. Bạn chỉ bị tính phí cho số ACU đang sử dụng tại từng thời điểm + chi phí storage.

### Các trường hợp sử dụng phù hợp với Aurora Serverless

1. **Ứng dụng ít được sử dụng** Ví dụ: blog lưu lượng thấp như “The Best Cats”. Kết nối chỉ xuất hiện vài phút mỗi ngày hoặc vài ngày trong tuần. Bạn chỉ trả tiền theo giây khi thực sự có tải.
2. **Ứng dụng mới** Khi bạn chưa biết trước mức tải, khó chọn size instance. Với Aurora Provisioned bạn phải đoán và provision trước (có thể gây disruption khi thay đổi). Với Serverless, database sẽ tự scale theo tải thực tế.
3. **Workload biến động** Ứng dụng bình thường nhẹ nhưng có peak (ví dụ 30 phút/giờ hoặc vào ngày sale). Không cần provision cố định theo peak hay theo mức trung bình.
4. **Workload không thể dự đoán** Khi bạn không có đủ dữ liệu để dự đoán tải. Có thể đặt khoảng ACU rộng (min thấp – max cao), theo dõi trong thời gian đầu. Nếu vẫn không ổn định, Aurora Serverless là lựa chọn phù hợp lâu dài.
5. **Database dùng cho Development & Test** Có thể cấu hình **pause** khi không có tải → chỉ bị tính phí storage. Rất tiết kiệm chi phí cho môi trường dev/test.
6. **Ứng dụng multi-tenant** Khi chi phí infrastructure tăng tỷ lệ thuận với doanh thu (ví dụ tính phí theo license/tháng). Khi có nhiều user hơn → tải tăng → chi phí DB tăng, nhưng doanh thu cũng tăng tương ứng → rất hợp lý.

### Lưu ý về kỳ thi

Aurora Serverless **chưa xuất hiện nhiều** trong đề thi hiện tại, nhưng sẽ xuất hiện ngày càng nhiều trong tương lai. Việc nắm vững kiến trúc ngay từ bây giờ sẽ giúp bạn có lợi thế, đặc biệt là các câu hỏi so sánh với các sản phẩm RDS khác.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Aurora Serverless vs Aurora Provisioned
- ACU (Aurora Capacity Units)
- Scale min/max + pause xuống 0
- Proxy fleet quản lý connection
- Warm pool ACU do AWS quản lý
- Use cases: infrequent, new, variable, unpredictable, dev/test, multi-tenant
- Billing theo giây + chỉ tính storage khi pause

**Notes (Chi tiết):**

- **Aurora Serverless** = phiên bản không cần provision instance cố định (tương tự Fargate với ECS).
- Dùng **ACU** thay vì instance: mỗi ACU = compute + memory tương ứng.
- Có thể set **min/max ACU**, tự scale theo tải, thậm chí **pause về 0** (chỉ tính phí storage).
- Billing theo **per-second**.
- Vẫn có **cluster volume** replicate 6 node trên nhiều AZ (resilience giống Provisioned).
- ACU lấy từ **warm pool** shared, stateless, không có local storage → cấp phát nhanh.
- Connection đi qua **proxy fleet** do AWS quản lý → scale mượt, không gián đoạn client.
- Phù hợp với: app ít dùng, app mới, workload biến động/không dự đoán được, dev/test, multi-tenant (chi phí gắn với doanh thu).

**Summary (Tóm tắt ngắn):** **Aurora Serverless** là phiên bản database gần với **fully managed** hơn so với **Aurora Provisioned**. Thay vì provision instance cố định, bạn chỉ cần set **min/max ACU**, hệ thống tự scale (kể cả pause về 0). Kết nối được quản lý qua **proxy fleet** nên scale không gây gián đoạn. Rất phù hợp với các workload không ổn định, không dự đoán được hoặc chỉ dùng theo thời gian (dev/test, blog ít traffic, multi-tenant).