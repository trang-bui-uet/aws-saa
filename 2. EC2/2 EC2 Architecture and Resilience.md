Bài học này sẽ tóm tắt kiến trúc của EC2
Nhìn cụ thể vào máy chủ EC2, chúng được thiết kế vật lý như thế nào, tại sao chúng có khả năng phục hồi ở mức AZ, và khả năng phục hồi của EBS (Elastic Block Store) là nhân tố quan trọng trong quyết định của SA.

### EC2 Architecture

#### Các điểm chính
EC2 instances là virtual machine (OS + Resource)
EC2 instances chạy trên EC2 hosts và chúng là máy chủ vật lý thật (Physical Service Hardware)
Máy chủ này (EC2 hosts) có thể là Shared Hosts (Chia sẻ) hoặc Dedicated Hosts (Dành riêng)
Shared Host: Bạn chi sẻ máy chủ với nhiều dùng khác, bạn chỉ trả tiền cho instances bạn chạy
Dedicated Host: Bạn sở hữu nguyên máy chủ, bạn trả tiền cho toàn bộ máy chủ
EC2 là một dịch vụ có khả năng phục hồi (resilient) ở mức AZ lý do là hosts thì chạy trong 1 AZ nên AZ lỗi, Host lỗi, Instance lỗi.

#### Kiến trúc:
Chúng ta có region us-east-1, trong region có hai AZ AZ-A, AZ-B, trong AZ-A chúng ta có hai subnet sn-A, sn-B.
Trong mỗi AZ chúng ta có một EC2 host, mỗi EC2 host chạy trong 1 AZ
EC2 hosts có vài phần cứng nội bộ logically CPU và memory, chúng cũng có ổ cứng cục bộ gọi là Instance Store, Instance store chỉ lưu dữ liệu tạm thời, dữ liệu sẽ mất khi instance stop hoặc terminate, host bị lỗi, instance bị reboot. Nếu instance bị di chuyển từ host này sang host khác thì dữ liệu sẽ mất.
Ngoài ra chúng ta có hai loại networking: Storage networking mạng cho lưu trữ, **Data networking** mạng cho dữ liệu.
Khi instance được cấp phát vào một subnet cụ thể trong VPC thì thực chất là: Một **Primary Elastic Network Interface (ENI)** được khởi tạo trong subnet, và nó ánh xạ trực tiếp tới phần cứng mạng vật lý trên EC2 host.
ENI (Elastic Network Interface) có thể coi là một card mạng ảo (virtual NIC) được AWS tạo ra và gắn trực tiếp vào subnet mà bạn chọn.
Instances có thể có nhiều network interfaces thậm chí trong các subnet khác nhau miễn là chúng thuộc 1 AZ. Mọi thứ về EC2 thì tập chung xung quanh kiến trúc này, sự thật là nó chạy trong 1 AZ cụ thể.
EC2 có thể sử dụng remote storage, cho nên EC2 có thể kết nối tới **Elastic Block Store (EBS)**, EBS thì cũng chạy bên trong một AZ cụ thể, cho nên dịch vụ chạy trong AZ-A thì khác với dịch vụ chạy trong AZ-B và bạn không thể truy cập chúng qua các AZ khác nhau, EBS cho bạn cấp phát volumns (ổ cứng)

