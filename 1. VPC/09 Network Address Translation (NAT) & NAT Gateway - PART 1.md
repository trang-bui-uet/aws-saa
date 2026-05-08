- A set of processes - remapping SRC or DST IPs
- IP masquerading - hiding CIDR Blocks behind one IP
    - **IP Masquerading** (hay còn gọi là **PAT - Port Address Translation**) cho phép **toàn bộ một dải private IP** (một CIDR block lớn, ví dụ: 10.0.0.0/16) **ẩn sau chỉ một public IP duy nhất**.
    - Khi nhiều instance trong private subnet gửi traffic ra internet, NAT sẽ dịch tất cả private IP của chúng thành **một public IP** (thường là Elastic IP của NAT Gateway).
    - Đồng thời NAT còn theo dõi **cổng (port)** để phân biệt traffic của từng instance khi phản hồi quay về.
- Public IPv4 Addresses running out Lý do NAT ra đời và vẫn rất quan trọng:
    - Chúng ta đang cạn kiệt địa chỉ **Public IPv4** trên thế giới.
    - Không thể cấp public IP cho từng máy chủ, nên phải dùng NAT để “tiết kiệm” public IP bằng cách để nhiều máy chia sẻ một IP.
- Gives Private CIDR range outgoing internet access NAT 
	- cho phép các máy chủ nằm trong **private subnet** (private CIDR) **ra được internet** (outgoing access), nhưng **internet không thể chủ động kết nối vào** chúng (inbound bị chặn). → Đây là thiết kế bảo mật rất tốt.

**NAT Architecture**

![[Pasted image 20260409220810.png]]

**NAT Gateways**
- Runs from a **public subnet**
- Uses **Elastic IPs** (Static IPv4 Public)
- **AZ resilient Service** (HA in that AZ)
- For region resilience - **NATGW in each AZ** ...
- .. RT in for each AZ with that NATGW as target
- Managed, scales to **45 Gbps**, $ Duration & Data Volume

![[Pasted image 20260409221114.png]]