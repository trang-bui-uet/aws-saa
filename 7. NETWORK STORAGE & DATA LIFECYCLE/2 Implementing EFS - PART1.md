**Welcome back!** Và trong bài demo này, tôi muốn mang đến cho bạn một số trải nghiệm thực tế mang tính trừu tượng về việc sử dụng **Elastic File System (EFS)**.

### Chuẩn bị hạ tầng

Trước khi bắt đầu, như mọi khi, hãy đảm bảo bạn đã **đăng nhập vào AWS account chung** (management account của organization) và chọn region **Northern Virginia (us-east-1)**.

Đính kèm bài học này có **one-click deployment link**. Hãy click vào đó. Nó sẽ đưa bạn đến màn hình **Quick create stack** với mọi thứ đã được điền sẵn. Bạn chỉ cần:

- Cuộn xuống dưới cùng
- **Tick vào ô Capabilities**
- Click **Create stack**

Bạn cũng sẽ cần gõ một số lệnh trong bài demo này, vì vậy đính kèm còn có file **lesson commands**. Hãy mở nó ở tab mới. Đây là danh sách các lệnh sẽ dùng trong demo, có một số placeholder như **File System ID** cần thay thế khi đi đến phần tương ứng. Hãy giữ file này mở để tham khảo.

Chúng ta cần stack ở trạng thái **CREATE_COMPLETE** trước khi tiếp tục. Hãy **tạm dừng video** và tiếp tục khi stack đã chuyển sang **CREATE_COMPLETE**.

---

### Kiểm tra hạ tầng đã tạo

Stack đã ở trạng thái **CREATE_COMPLETE**. Nó đã tạo:

- **Animals for Life base VPC**
- Một số **EC2 instances**

Vào **EC2 console** → **Instances Running**, bạn sẽ thấy hai instance:

- **A4L-EFS-Instance-A**
- **A4L-EFS-Instance-B**

Chúng ta sẽ tạo **EFS file system** và **mount points**, sau đó mount chúng lên cả hai instance và thao tác với dữ liệu trên file system đó. Mục tiêu là cho bạn trải nghiệm làm việc với **network shared file system**.

---

### Tạo EFS File System

1. Trong ô tìm kiếm phía trên, gõ **EFS** và mở console ở **tab mới**.
2. Giữ nguyên tab EC2 Instances vì sẽ quay lại sau.
3. Trong EFS console, click **Create file system**.

Bạn có hai cách:

- Dùng dialog đơn giản
- Click **Customize** để tùy chỉnh chi tiết hơn

Chúng ta sẽ dùng **Customize**.

**Các thiết lập quan trọng:**

- **Name**: A4L-EFS
- **Storage class**: Chọn **Standard** (dữ liệu được replicate qua nhiều Availability Zones). (Nếu chỉ test/dev hoặc dữ liệu không quan trọng thì có thể chọn **One Zone**).
- **Automatic backups**: **Disable** (trong demo này). Trong production nên bật.
- **Lifecycle management**:
    - Transition into IA: **30 days**
    - Transition out of IA: **On first access**
- **Throughput mode**: **Bursting**
- **Performance mode**: **General Purpose** (mặc định, nên dùng hầu hết trường hợp). Chỉ dùng **Max I/O** khi thật sự cần throughput và IOPS rất cao.
- **Encryption**: **Tắt** (trong demo). Production thì nên bật (dùng KMS).

Click **Next**.

---

### Cấu hình Network (Mount Targets)

Best practice: Tạo **mount target** ở mọi Availability Zone mà bạn sẽ sử dụng EFS.

- Xóa hết Security Group mặc định.
- Chọn subnet:
    - **us-east-1a** → **App-A**
    - **us-east-1b** → **App-B**
    - **us-east-1c** → **App-C**
- Security Group cho mỗi mount target: Chọn **Instance Security Group** (đã được CloudFormation tạo sẵn – cho phép kết nối từ các instance có gắn security group này).

Click **Next**.

---

### File System Policy

Ở bước này bạn có thể:

- Ngăn root access
- Force read-only
- Ngăn anonymous access
- **Enforce encryption in transit**

Trong demo này chúng ta **không dùng** bất kỳ policy nào. Click **Next**.

---

### Review & Create

Kiểm tra lại mọi thứ, sau đó click **Create**.

Sau khi tạo xong, vào file system → tab **Network**. Bạn sẽ thấy **3 mount targets** đang được tạo. **Tất cả 3 mount targets phải ở trạng thái Ready** trước khi tiếp tục.

---

**Kết thúc Part 1.** Hãy hoàn thành video này. Khi cả 3 mount targets đã sẵn sàng, bạn có thể bắt đầu **Part 2**.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- One-click CloudFormation stack (Animals for Life VPC + 2 EC2)
- Tạo EFS File System với Customize
- Storage class Standard vs One Zone
- Lifecycle management (IA)
- Throughput: Bursting | Performance: General Purpose
- Mount Targets ở 3 AZ (App-A/B/C)
- Instance Security Group
- Không dùng File System Policy & Encryption (demo)

**Notes (Chi tiết):**

- Đăng nhập **management account**, region **us-east-1**.
- Dùng **one-click deployment** → đợi stack **CREATE_COMPLETE**.
- Tạo 2 instance: **A4L-EFS-Instance-A** và **A4L-EFS-Instance-B**.
- Tạo EFS tên **A4L-EFS**, chọn **Standard**, **Bursting**, **General Purpose**, tắt backup & encryption.
- Tạo **mount targets** ở 3 AZ với subnet App-A/B/C và **Instance Security Group**.
- Đợi cả 3 mount targets **Ready** trước khi sang Part 2.

**Summary (Tóm tắt ngắn):** Bài demo Part 1 hướng dẫn chuẩn bị hạ tầng và **tạo EFS File System** (Standard, Bursting, General Purpose) cùng **mount targets** ở 3 Availability Zones, sẵn sàng để mount lên 2 EC2 instance trong Part 2.