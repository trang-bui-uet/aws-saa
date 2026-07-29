Chào mừng trở lại. Trong video này, tôi muốn nói về một tính năng đặc thù của RDS gọi là **RDS Custom**. Đây là chủ đề khá **niche**. Tôi gần như chưa thấy ai sử dụng nó trong thực tế, và đối với kỳ thi, bạn chỉ cần hiểu ở mức **rất cơ bản**. Vì vậy tôi sẽ trình bày ngắn gọn.

### RDS Custom là gì?

**RDS Custom** lấp đầy khoảng trống giữa sản phẩm **RDS thông thường** và việc tự chạy database engine trên **EC2**.

- **RDS thông thường**: Là dịch vụ database server được AWS quản lý hoàn toàn. Bạn chỉ được truy cập vào database, còn OS và engine thì bị hạn chế rất nhiều.
- **Database trên EC2**: Bạn tự quản lý toàn bộ từ hệ điều hành trở lên → overhead rất lớn.

**RDS Custom** nằm ở vị trí trung gian: bạn vẫn được hưởng một phần lợi ích của RDS, nhưng đồng thời có thể thực hiện một số tùy chỉnh sâu hơn giống như khi chạy database trên EC2.

Hiện tại, **RDS Custom chỉ hỗ trợ Microsoft SQL Server và Oracle**.

Khi sử dụng RDS Custom, bạn có thể kết nối bằng **SSH, RDP và Session Manager** để truy cập trực tiếp vào hệ điều hành và database engine.

### Khác biệt quan trọng so với RDS thông thường

- Với **RDS thông thường**: Bạn sẽ **không thấy** EC2 instance, EBS volume hay backup trong S3 của tài khoản mình, vì tất cả đều chạy trong môi trường do AWS quản lý. Networking được thực hiện bằng cách inject **Elastic Network Interface (ENI)** vào VPC của bạn.
- Với **RDS Custom**: Mọi thứ chạy **bên trong tài khoản AWS của bạn**. Bạn sẽ thấy:
    - EC2 instance
    - EBS volumes
    - Backup nằm trong tài khoản của bạn

### Thực hiện tùy chỉnh trên RDS Custom

Nếu cần thực hiện bất kỳ customization nào, bạn phải chú ý đến **Database Automation settings**:

1. **Pause Database Automation** (tạm dừng automation)
2. Thực hiện các tùy chỉnh cần thiết
3. **Resume Automation** (bật lại automation)

Việc này giúp tránh xung đột giữa thao tác tùy chỉnh của bạn và automation của RDS, đảm bảo database sẵn sàng cho môi trường production.

### Mô hình trách nhiệm (Service Model)

- **On-premises**: Khách hàng chịu trách nhiệm **toàn bộ** (từ hardware đến application optimization).
- **RDS thông thường**: AWS chịu trách nhiệm gần như tất cả, khách hàng chỉ còn **Application Optimization**.
- **Database trên EC2**: AWS chỉ chịu trách nhiệm **Hardware**. Còn lại (OS, patch, backup, HA, scaling, application optimization…) đều do khách hàng đảm nhiệm.
- **RDS Custom**:
    - **Hardware** → AWS quản lý
    - **Application Optimization** → Khách hàng quản lý
    - Tất cả phần còn lại → **Shared responsibility** giữa khách hàng và AWS

→ RDS Custom mang lại lợi ích của cả hai phía: vừa có automation của RDS, vừa có khả năng tùy chỉnh và truy cập sâu hơn (SSH / Session Manager / RDP).

![[Pasted image 20260729012157.png]]

### Lưu ý cuối cùng

Đối với kỳ thi, bạn chỉ cần nhớ:

- RDS Custom hiện **chỉ hỗ trợ Oracle và Microsoft SQL Server**
- Nó cho phép truy cập OS và engine
- Nằm ở vị trí trung gian giữa RDS thông thường và EC2 tự quản lý

Trong thực tế, đây là tính năng khá hiếm gặp, chỉ xuất hiện trong một số tình huống rất đặc thù.

Đó là toàn bộ nội dung tôi muốn trình bày trong video này. Hãy hoàn thành video và khi sẵn sàng, chúng ta sẽ gặp nhau ở bài học tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- RDS Custom là gì?
- Chỉ hỗ trợ Oracle & MS SQL
- Truy cập được OS (SSH / RDP / Session Manager)
- Chạy bên trong tài khoản của bạn
- Pause / Resume Database Automation
- Mô hình Shared Responsibility

**Notes (Chi tiết):**

- **RDS Custom** nằm giữa RDS managed và database tự quản lý trên EC2.
- Cho phép kết nối SSH, RDP, Session Manager để truy cập OS và engine.
- Khác với RDS thông thường: bạn sẽ thấy EC2 instance, EBS volume và backup trong tài khoản của mình.
- Khi cần tùy chỉnh: phải **Pause Automation** → thực hiện thay đổi → **Resume Automation**.
- Mô hình trách nhiệm:
    - Hardware: AWS
    - Application Optimization: Khách hàng
    - Các phần còn lại: Shared giữa AWS và khách hàng
- Hiện chỉ hỗ trợ **Oracle** và **Microsoft SQL Server**.

**Summary (Tóm tắt ngắn):** RDS Custom là lựa chọn trung gian giữa RDS fully managed và database tự chạy trên EC2. Nó cho phép truy cập hệ điều hành (SSH/RDP/Session Manager), chạy bên trong tài khoản của bạn và hiện chỉ hỗ trợ Oracle cùng Microsoft SQL Server. Khi tùy chỉnh phải tạm dừng automation. Đây là chủ đề niche, chỉ cần hiểu ở mức cơ bản cho kỳ thi.