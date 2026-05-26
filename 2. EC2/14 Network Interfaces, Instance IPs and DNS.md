**Chào mừng bạn quay trở lại.** Và trong bài học này, tôi muốn đề cập đến một số **lý thuyết networking** liên quan đến các **EC2 instances**. Tôi muốn nói về **network interfaces**, **instance IPs**, và **instance DNS**.

**EC2** là một sản phẩm rất phong phú về tính năng và có rất nhiều nuance trong cách bạn có thể kết nối và tương tác với nó. Vì vậy, việc hiểu rõ cách **interfaces**, **IPs**, và **DNS** hoạt động là **cực kỳ quan trọng**.

### 1. Kiến trúc tổng quát của EC2 Networking

Hãy bắt đầu và xem xét. Về mặt kiến trúc, đây là cách **EC2** hoạt động. Chúng ta có một **EC2 instance**, và nó **luôn bắt đầu với một network interface**. Interface này được gọi là **ENI (Elastic Network Interface)**.

Và mọi **EC2 instance** đều có **ít nhất một interface**, đó là **primary interface** hoặc **primary ENI**.

**Tùy chọn**, bạn có thể gắn một hoặc nhiều **secondary elastic network interfaces**, chúng có thể nằm ở các **subnet riêng biệt**, nhưng **mọi thứ phải nằm trong cùng một availability zone**.

> Nhớ rằng, **EC2** được cô lập trong một **availability zone**, nên điều này rất quan trọng. Một instance có thể có các network interfaces khác nhau ở các subnet riêng biệt, nhưng tất cả những subnet đó phải nằm trong cùng một availability zone (trong trường hợp này là availability zone A).

### 2. Những gì thực sự được gắn vào Network Interface

![[Pasted image 20260523001005.png]]

Bây giờ, các **network interfaces** này có một số thuộc tính hoặc những thứ được gắn vào chúng. Điều này rất quan trọng vì khi bạn xem một **EC2 instance** qua console UI, những thứ này thường được trình bày như thể chúng được gắn trực tiếp vào instance.

Nhưng khi bạn tương tác với instance từ góc nhìn networking, bạn thường đang thấy các yếu tố thuộc về **primary network interface**.

Ví dụ: khi bạn launch một instance với **security groups**, thì những **security groups** đó thực tế nằm trên **network interface**, chứ không phải trên instance.

**Những thuộc tính quan trọng gắn trên mỗi ENI:**

- **MAC address**: Địa chỉ phần cứng của interface, có thể nhìn thấy được bên trong hệ điều hành. Dùng cho **software licensing**.
- **Primary IPv4 private address**: Lấy từ dải địa chỉ của subnet. Khi bạn chọn VPC + subnet cho EC2 instance, thực chất bạn đang chọn cho **primary network interface**.
- **Secondary private IP addresses**: Có thể có 0 hoặc nhiều.
- **Public IPv4 address**: Có thể có 0 hoặc 1 (dynamic).
- **Elastic IP address**: Có thể có **một Elastic IP cho mỗi private IP** trên interface.
- **IPv6 addresses**: Có thể có 0 hoặc nhiều (mặc định publicly routable).
- **Security groups**: Được áp dụng trên **network interface**. Một security group ảnh hưởng đến **tất cả IP addresses** trên interface đó.
- **Source / Destination Check**: Có thể bật/tắt. Phải **tắt** nếu muốn instance làm **NAT instance**.

> **Quan trọng**: Nếu bạn cần các IP addresses khác nhau trên một instance bị ảnh hưởng bởi **security groups khác nhau**, thì bạn **phải** tạo nhiều interfaces và gán security group riêng cho từng interface.

**Secondary ENI** có thể **detach** và **attach** sang instance khác – đây là điểm khác biệt lớn so với primary ENI.

### 3. DNS Names của EC2 Instance

Giả sử trong ví dụ này, instance nhận được **primary IPv4 private IP address** là **10.16.0.10**. Đây là địa chỉ **static** và **không thay đổi** trong suốt thời gian tồn tại của instance.

Instance cũng được cấp một **private DNS name** có định dạng: ip-10-16-0-10.ec2.internal → Chỉ resolve được **bên trong VPC** và luôn trỏ đến **primary private IP**.

Nếu instance có **public IPv4 address** (dynamic):

- Public IP này **không cố định**. Stop → Start instance sẽ thay đổi public IP.
- Restart instance thì public IP **không thay đổi**.
- Instance được cấp **public DNS name** có dạng: ec2-54-xxx-xxx-xxx.compute-1.amazonaws.com

**Đặc biệt quan trọng về Public DNS:**

- Bên trong VPC → resolve về **primary private IPv4 address**.
- Bên ngoài VPC → resolve về **public IPv4 address**.

Đây là cơ chế giúp traffic luôn dùng private IP khi giao tiếp nội bộ VPC.

### 4. Elastic IP Addresses

**Elastic IP** được allocate cho tài khoản AWS của bạn. Bạn có thể associate nó với bất kỳ private IP nào (primary hoặc secondary interface).

- Khi associate Elastic IP vào primary interface → public IPv4 address thông thường (non-elastic) sẽ **bị xóa**.
- Khi release Elastic IP → instance nhận một **public IPv4 address mới** (khác hoàn toàn).

> **Câu hỏi thi hay gặp**: Nếu instance có public IP dynamic → gán Elastic IP → sau đó release Elastic IP → có lấy lại public IP cũ được không? **Đáp án**: **Không**. Instance sẽ nhận public IP hoàn toàn mới.

### Tóm tắt quan trọng

- **Instances** có một hoặc nhiều **network interfaces** (primary + tùy chọn secondary).
- Trên **mỗi network interface**:
    - 1 **primary private IP** (static)
    - 0 hoặc nhiều **secondary private IPs**
    - 0 hoặc 1 **public IPv4** (dynamic)
    - 0 hoặc nhiều **Elastic IPs**
    - **Security groups** (áp dụng cho toàn bộ interface)
    - **Source/Destination Check**

### Exam Power-Ups (Mẹo thi)

**Mẹo 1 – Secondary ENI & MAC Address** Nhiều phần mềm legacy license dựa trên **MAC address**. Bạn có thể tạo secondary ENI → dùng MAC của nó để license → detach và attach sang instance khác để **di chuyển license** cực kỳ dễ dàng.

**Mẹo 2 – Multi-homed systems** Dùng nhiều ENI ở các subnet khác nhau:

- 1 interface cho **management**
- 1 interface cho **data**

**Mẹo 3 – Security Groups** Không thể gán security group khác nhau cho từng IP trên cùng một interface → phải dùng **nhiều ENI**.

**Mẹo 4 – Public IP trong OS** **Hệ điều hành không bao giờ thấy public IPv4 address**. Public IP chỉ được NAT bởi Internet Gateway. Bên trong OS bạn chỉ thấy và cấu hình **primary private IPv4**.

**Mẹo 5 – Public IP dynamic** Stop & Start instance → public IP thay đổi. Muốn IP cố định → phải dùng **Elastic IP**.

**Mẹo 6 – Public DNS behavior**

- Trong VPC → resolve về private IP
- Ngoài VPC → resolve về public IP

Tôi biết đây là rất nhiều lý thuyết, nhưng đây thực sự quan trọng từ góc nhìn networking, nên bạn cần nắm thật rõ.

Đừng lo, khi chúng ta tiếp tục khóa học, tất cả những khái niệm này sẽ ngấm dần khi bạn thực hành.

**Đó là tất cả những gì tôi muốn trình bày trong bài này.** Hãy đánh dấu video này là **hoàn thành**, và khi bạn sẵn sàng, hãy gặp tôi ở bài học tiếp theo.