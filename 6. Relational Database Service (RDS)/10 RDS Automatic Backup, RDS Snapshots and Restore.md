Chào mừng trở lại. Trong video này, mình sẽ nói về cách **RDS** có thể được **backup** và **restore**, cũng như các phương pháp backup khác nhau hiện có. Chúng ta có khá nhiều nội dung cần cover, vậy nên hãy bắt đầu ngay.

### Hai loại backup trong RDS

Trong **RDS**, có **hai loại chức năng backup**:

- **Automated backups** (backup tự động)
- **Snapshots** (ảnh chụp thủ công)

Cả hai đều được lưu trữ trong **S3**, nhưng sử dụng **bucket do AWS quản lý**, nên bạn **không thể nhìn thấy** chúng trong console S3 của mình. Bạn có thể xem backups trong **RDS console**, nhưng không thể chuyển sang S3 để thấy bất kỳ bucket RDS nào dành cho backup. Hãy nhớ điều này vì mình đã từng thấy câu hỏi về nó trong bài thi.

Lợi ích của việc dùng **S3** là dữ liệu trong backups trở nên **regionally resilient** (có khả năng chịu lỗi theo vùng), vì S3 replicate dữ liệu qua nhiều **Availability Zones** trong cùng region.

Khi backup RDS diễn ra, trong hầu hết các trường hợp chúng được lấy từ **standby instance** nếu bạn đã bật **Multi-AZ**. Vì vậy, dù có gây ra **I/O pause**, nhưng pause này xảy ra trên standby nên **không ảnh hưởng** đến hiệu suất ứng dụng. Nếu không dùng Multi-AZ (ví dụ với môi trường test/dev), thì backup được lấy từ instance duy nhất đang chạy, và bạn có thể bị gián đoạn hiệu suất.

### Snapshots (thủ công)

Bây giờ mình sẽ đi sâu hơn về cách backup hoạt động, bắt đầu với **snapshots**.

- Snapshots **không tự động**. Bạn phải chạy chúng một cách tường minh (hoặc qua script/ứng dụng tùy chỉnh).
- Chúng được lưu trong **S3 do AWS quản lý**.
- Chúng hoạt động giống như **EBS snapshots** mà bạn đã học ở phần khác trong khóa học.
- Snapshots và automated backups được lấy **của toàn bộ instance** (tức là tất cả các database bên trong), chứ không phải chỉ một database đơn lẻ.

**Snapshot đầu tiên** là bản sao **full** của dữ liệu. Từ đó trở đi, snapshots chỉ lưu phần dữ liệu **đã thay đổi** kể từ snapshot trước (incremental).

Khi bất kỳ snapshot nào diễn ra, sẽ có một **gián đoạn ngắn** trong luồng dữ liệu giữa compute và storage.

- Nếu dùng **Single-AZ** → có thể ảnh hưởng ứng dụng.
- Nếu dùng **Multi-AZ** → xảy ra trên standby nên gần như không nhận thấy.

Về thời gian: snapshot đầu tiên có thể mất khá lâu (vì là full copy). Các snapshot sau sẽ nhanh hơn nhiều vì chỉ lưu dữ liệu thay đổi. **Ngoại lệ**: nếu instance có lượng dữ liệu thay đổi lớn, thì các snapshot sau cũng có thể mất nhiều thời gian.

**Snapshots không bao giờ hết hạn**. Bạn phải tự dọn dẹp chúng. Điều này có nghĩa là snapshots **vẫn tồn tại** ngay cả sau khi bạn xóa RDS instance. Chúng chỉ bị xóa khi bạn xóa thủ công hoặc qua process bên ngoài. Hãy nhớ điều này vì nó quan trọng cho bài thi.

Bạn có thể chạy snapshot theo bất kỳ lịch nào bạn muốn: 1 lần/tháng, 1 lần/tuần, 1 lần/ngày, 1 lần/giờ… vì chúng là **manual**. Một cách để đạt **RPO thấp hơn** là chụp snapshot thường xuyên hơn. Khoảng cách giữa các snapshot càng ngắn thì lượng dữ liệu mất mát tối đa khi xảy ra sự cố càng nhỏ.

### Automated Backups

Automated backups diễn ra **một lần mỗi ngày**, kiến trúc giống hệt: cái đầu tiên là full, các cái sau chỉ lưu dữ liệu thay đổi. Bạn có thể nghĩ chúng như **automated snapshots**.

Chúng xảy ra trong một **backup window** được định nghĩa trên instance. Bạn có thể để AWS chọn ngẫu nhiên hoặc chọn khung giờ phù hợp với business.

- Nếu dùng **Single-AZ** → nên chọn khung giờ ít hoặc không có traffic, vì sẽ có I/O pause.
- Nếu dùng **Multi-AZ** → không cần lo vì backup chạy từ standby.

Ngoài automated snapshot, **mỗi 5 phút** database transaction logs cũng được ghi vào S3. Transaction logs lưu các thao tác thực tế thay đổi dữ liệu. Kết hợp với snapshots từ automated backups, bạn có thể **restore database về bất kỳ thời điểm nào** với độ chi tiết **5 phút**. Về lý thuyết, điều này cho phép đạt **RPO 5 phút**.

Automated backups **không được giữ vô thời hạn**. AWS tự động dọn dẹp chúng. Với mỗi RDS instance, bạn có thể đặt **retention period** từ **0 đến 35 ngày**.

- 0 = tắt automated backups.
- Tối đa = 35 ngày. Nếu đặt 35 ngày, bạn có thể restore về bất kỳ thời điểm nào trong 35 ngày đó bằng snapshots + transaction logs. Dữ liệu cũ hơn 35 ngày sẽ tự động bị xóa.

Khi bạn **xóa database**, bạn có thể chọn giữ lại automated backups, nhưng — và đây là điểm **rất quan trọng** — chúng vẫn sẽ hết hạn theo retention period đã đặt.

Cách duy nhất để giữ nội dung của một RDS instance **lâu hơn 35 ngày** là: khi xóa instance, bạn phải tạo một **final snapshot**. Snapshot này hoàn toàn do bạn kiểm soát và phải xóa thủ công khi cần.

### Cross-region Backup Replication

RDS cũng cho phép bạn **replicate backups** sang một AWS region khác (bao gồm cả snapshots và transaction logs).

- Có phí cho việc copy dữ liệu cross-region và storage ở region đích.
- **Đây không phải mặc định**. Bạn phải **cấu hình tường minh** trong phần automated backups.

### Restores

Cách RDS xử lý restore **rất quan trọng** và không trực quan ngay từ đầu.

Khi bạn restore từ **automated backup** hoặc **manual snapshot**, RDS sẽ **tạo ra một RDS instance hoàn toàn mới**. Điều này quan trọng vì bạn sẽ phải **cập nhật ứng dụng** để dùng **endpoint address mới** (khác với endpoint cũ).

- Khi restore từ **manual snapshot** → bạn restore database về **một thời điểm cố định** (thời điểm snapshot được tạo). Điều này ảnh hưởng trực tiếp đến **RPO**. Nếu bạn không chụp snapshot ngay trước khi sự cố xảy ra, RPO thường sẽ không tối ưu.
- Với **automated backups** thì khác. Bạn có thể chọn **bất kỳ thời điểm nào** để restore. Cách hoạt động: restore từ snapshot gần nhất, sau đó **replay transaction logs** từ thời điểm đó đến thời điểm bạn chọn. Điều này cải thiện RPO rất nhiều (có thể restore đến vài phút trước khi sự cố xảy ra).

Điều quan trọng cần hiểu: **restore snapshot không phải là quá trình nhanh**. Với database lớn, quá trình này có thể mất khá nhiều thời gian. Hãy tính đến thời gian restore của RDS khi lập kế hoạch **Disaster Recovery** và **Business Continuity**.

Ở video khác trong khóa học, mình sẽ nói về **Read Replicas**. Chúng cung cấp cách cải thiện **RTO** đáng kể khi muốn recovery từ failure.

Automated backups của RDS rất tốt để recovery từ failure hoặc restore dữ liệu bị corruption, nhưng **thời gian restore lâu**, nên bạn phải tính toán vào **RTO planning**.

Nếu phù hợp với exam bạn đang học, sẽ có demo để bạn trải nghiệm thực tế quá trình restore. Nếu không thấy demo thì đừng lo, vì nó không bắt buộc với exam của bạn.

Đến đây là toàn bộ nội dung mình muốn cover trong video này. Hãy hoàn thành video, và khi sẵn sàng, mình rất mong được gặp bạn ở video tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Automated backups vs Snapshots
- Lưu trữ trong S3 (AWS-managed)
- Multi-AZ & I/O pause
- Incremental snapshots
- Transaction logs (5 phút)
- Retention period (0–35 ngày)
- Final snapshot khi xóa instance
- Cross-region replication (không mặc định)
- Restore luôn tạo instance mới
- RPO & RTO khi restore

**Notes (Chi tiết):**

- Cả **automated backups** và **snapshots** đều lưu trong **S3 do AWS quản lý** → không thấy được trong S3 console.
- Backup thường lấy từ **standby** nếu có Multi-AZ → hạn chế ảnh hưởng hiệu suất.
- **Snapshots** là thủ công, incremental sau lần full đầu tiên, **không tự hết hạn**, vẫn tồn tại sau khi xóa instance.
- **Automated backups** chạy 1 lần/ngày + transaction logs mỗi 5 phút → hỗ trợ **Point-in-Time Recovery** với độ chi tiết 5 phút.
- Retention tối đa **35 ngày**. Muốn giữ lâu hơn → phải tạo **final snapshot** trước khi xóa.
- Cross-region replication **phải bật thủ công** và có phí.
- Restore **luôn tạo RDS instance mới** → endpoint thay đổi → phải update application.
- Snapshot restore = điểm thời gian cố định. Automated backup restore = chọn bất kỳ thời điểm nào trong retention.
- Thời gian restore có thể rất lâu với DB lớn → ảnh hưởng RTO. Read Replicas tốt hơn cho RTO.

**Summary (Tóm tắt ngắn):** RDS hỗ trợ **hai loại backup**: **Snapshots** (thủ công, không hết hạn) và **Automated backups** (tự động hàng ngày + transaction logs 5 phút, retention tối đa 35 ngày). Cả hai đều lưu trong S3 managed. Restore luôn tạo **instance mới**. Automated backups cho phép **Point-in-Time Recovery** tốt hơn, nhưng thời gian restore cần được tính vào RTO.