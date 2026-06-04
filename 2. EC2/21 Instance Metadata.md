**Giới thiệu về Instance Metadata**

**Chào mừng bạn quay lại.**

Bài học này chúng ta sẽ tìm hiểu một tính năng **rất quan trọng** của EC2 mang tên **Instance Metadata**. Đây là một kiến trúc rất đơn giản, nhưng nó được sử dụng trong hầu hết các tính năng mạnh mẽ của EC2. Vì vậy, bạn **phải nắm vững** cách hoạt động của nó.

Instance Metadata xuất hiện trong **gần như tất cả các kỳ thi AWS** và bạn sẽ sử dụng nó thường xuyên khi thiết kế và triển khai giải pháp thực tế trên AWS.

### Instance Metadata là gì?

**Instance Metadata** là dịch vụ mà EC2 cung cấp cho các instance. Đây là **dữ liệu về chính instance đó**, giúp instance (hoặc bất kỳ ứng dụng nào chạy bên trong) có thể truy vấn thông tin về môi trường mà bình thường hệ điều hành không thể biết được.

### Cách truy cập Instance Metadata

Bạn có thể truy cập metadata **từ bên trong mọi instance** bằng cùng một cách duy nhất.

**Địa chỉ IP cố định là:**

> **169.254.169.254**

**Hãy ghi nhớ IP này!** Nó xuất hiện rất thường xuyên trong đề thi.

**Mẹo nhớ IP (Exam Tip):** Tôi nhớ bằng cách lặp lại như vần: **“169.254 repeated”** → 169.254.169.254

Phần URL tiếp theo luôn là: **latest/meta-data/**

**Nhớ hai thứ này:**

- 169.254 repeated
- latest/meta-data/

**Exam Tip quan trọng:** Nhiều câu hỏi thi yêu cầu bạn chọn **đúng URL** của metadata. Đây là một trong những chi tiết “khó nhằn” bạn **cần học thuộc**. Hãy lặp lại, viết ra flashcard – nó sẽ giúp bạn rất nhiều trong kỳ thi.

### Metadata cung cấp những thông tin gì?

Metadata cho phép bất kỳ ứng dụng nào chạy trên instance truy vấn thông tin về chính nó. Thông tin được chia thành nhiều danh mục, ví dụ:

- **Hostname**
- **Events**
- **Security groups**
- Và **rất nhiều thông tin khác**

Đặc biệt quan trọng là thông tin **networking**. Hệ điều hành bên trong instance **không nhìn thấy** public IPv4 address, nhưng qua **Instance Metadata**, ứng dụng hoàn toàn có thể lấy được thông tin đó.

### Xác thực và IAM Roles

Bạn cũng có thể lấy thông tin **xác thực** qua metadata. Khi EC2 instance được gán **IAM Role**, ứng dụng bên trong instance sẽ lấy **temporary credentials** (Access Key tạm thời) thông qua metadata.

### SSH Keys và User Data

Metadata còn được AWS sử dụng để:

- Truyền **SSH key tạm thời** (khi dùng EC2 Instance Connect).
- Truyền **User Data** – script tự động chạy khi instance khởi động để tự động cấu hình.

### Lưu ý quan trọng về Bảo mật (Exam Tip)

**Một sự thật cực kỳ quan trọng cho kỳ thi:**

- Metadata service **không có authentication** (không cần username/password).
- **Không được mã hóa** (không dùng HTTPS).

→ Bất kỳ ai có quyền truy cập **command line** (shell) của instance đều có thể đọc được toàn bộ metadata.

Bạn có thể chặn bằng firewall cục bộ (block IP 169.254.169.254), nhưng đây là công việc quản trị thêm. **Tóm lại:** Hãy coi metadata như một thứ **có thể bị lộ** và luôn tiếp cận được.

### Phần Demo thực tế

**Kết nối với Instance**

Bây giờ chúng ta chuyển sang **EC2 Console**, bạn sẽ thấy một instance đang chạy với tên **metadata**. Đây chính là instance chúng ta sẽ sử dụng để **demo Instance Metadata**.

#### Cách kết nối với Instance

Chúng ta sẽ kết nối bằng **EC2 Instance Connect** (không cần SSH client hay key pair).

1. Chọn instance
2. Click nút **Connect**
3. Đảm bảo chọn **EC2 Instance Connect**
4. Username là **ec2-user**
5. Click **Connect**

→ Một cửa sổ **web-based shell** sẽ mở ra ngay lập tức.

#### Khám phá Metadata Endpoint

Trước tiên, hãy xem cách sử dụng **Instance Metadata Service**.

**Nhớ lại địa chỉ IP quan trọng:** **169.254.169.254**

Để tương tác với IP này từ command line, chúng ta dùng lệnh curl.

**Lệnh đầu tiên:**

Bash

```
curl http://169.254.169.254/
```

Kết quả trả về danh sách **các version** của metadata service.

**Nhớ URL cho kỳ thi:** latest/meta-data/

Tiếp tục thử lệnh:

Bash

```
curl http://169.254.169.254/latest/
```

Sau đó mở rộng:

Bash

```
curl http://169.254.169.254/latest/meta-data/
```

→ Bạn sẽ thấy **toàn bộ danh mục dữ liệu** có sẵn về instance (ami-id, public keys, security groups, reservation ID…).

#### Truy vấn thông tin cụ thể

Hệ điều hành bên trong instance **không biết** public IPv4 address của chính nó. Hãy kiểm chứng:

Chạy lệnh:

Bash

```
ifconfig
```

→ Chỉ thấy **private IP** (ví dụ: 172.31.29.35).

Nhưng ứng dụng cần biết **public IP** thì sao? Dùng metadata:

Bash

```
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

→ Trả về **public IPv4** (ví dụ: 54.166.21.32). Kiểm tra lại trong EC2 Console để xác nhận khớp hoàn toàn.

**Thử thêm ví dụ khác:**

Bash

```
curl http://169.254.169.254/latest/meta-data/placement/
```

(Kết thúc bằng / nghĩa là đây là **thư mục**, có thể truy vấn sâu hơn)

Bash

```
curl http://169.254.169.254/latest/meta-data/placement/availability-zone
```

→ Trả về **us-east-1a**

**Security Groups:**

Bash

```
curl http://169.254.169.254/latest/meta-data/security-groups
```

→ Trả về tên Security Group của instance.

#### Truy cập User Data

User Data cũng nằm trong cùng service, chỉ cần thay meta-data bằng user-data:

Bash

```
curl http://169.254.169.254/latest/user-data
```

→ Trả về **script** mà CloudFormation đã truyền vào khi tạo instance (trong demo này là script cài jq).

#### Tóm tắt nhanh

**Instance Metadata Service** là một dịch vụ **text-based** cực kỳ đơn giản, chạy bên trong AWS. Bạn chỉ cần nhớ:

- IP: **169.254.169.254**
- Đường dẫn: **[http://169.254.169.254/latest/meta-data/](http://169.254.169.254/latest/meta-data/)**
- Hoặc **/latest/user-data** để lấy script khởi tạo.

#### Cleanup

**Quan trọng:** Sau khi học xong, hãy **xóa stack** để tránh phát sinh chi phí:

1. Vào **CloudFormation**
2. Chọn stack tên **metadata**
3. Click **Delete** → **Delete Stack**

Instance sẽ tự động bị terminate.

---

**Kết thúc demo thực tế.** Đây là một khái niệm rất đơn giản nhưng **cực kỳ quan trọng** cho kỳ thi và công việc thực tế. Hãy nhớ kỹ IP và định dạng URL nhé!

Hoàn thành video và hẹn gặp bạn ở bài học tiếp theo! 🚀