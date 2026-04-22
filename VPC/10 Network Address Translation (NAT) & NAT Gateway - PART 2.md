![[Pasted image 20260409221536.png]]
- VPC được chia thành **3 Availability Zone (AZ A, B, C)**.
- Mỗi AZ có:
    - Subnet DB (private)
    - Subnet APP (private)
    - Subnet WEB (public) chứa **1 NAT Gateway**
- Các instance ở **private subnet** (DB & APP) muốn ra Internet phải đi qua **NAT Gateway của chính AZ đó**.
- Mỗi AZ có NAT Gateway và Route Table riêng.
- Nếu 1 AZ bị down, 2 AZ còn lại vẫn ra Internet bình thường (không ảnh hưởng toàn bộ hệ thống).

**NAT instance vs NAT gateway**

|Thuộc tính|**NAT Gateway** (Khuyến nghị)|**NAT Instance** (Cách cũ)|
|---|---|---|
|**Availability**|Cao, tự động redundant trong AZ. Dễ làm multi-AZ.|Phải tự viết script để failover (chuyển đổi khi hỏng).|
|**Bandwidth**|Scale tự động lên đến **45 Gbps**|Giới hạn bởi loại instance (t2.medium, m5.large...)|
|**Maintenance**|AWS quản lý hoàn toàn (không cần vá lỗi, update)|Bạn phải tự quản lý (update OS, patch, monitoring...)|
|**Performance**|Được tối ưu sẵn cho NAT traffic|Dùng EC2 bình thường, hiệu năng kém hơn|
|**Cost**|Tính theo giờ + lượng dữ liệu truyền qua|Tính theo giờ instance + dữ liệu|
|**Type & Size**|Không cần chọn (AWS tự lo)|Phải tự chọn loại instance phù hợp|

**What about IPv6?**
- Với **IPv6** trong AWS, bạn **không cần NAT** như IPv4.
- Tất cả IPv6 address đều là public và có thể route trực tiếp qua Internet Gateway (IGW).
- NAT Gateway **không hỗ trợ IPv6**.
- Muốn ra vào Internet 2 chiều → dùng route ::/0 trỏ đến **IGW**.
- Muốn chỉ ra Internet (outbound only) → dùng **Egress-Only Internet Gateway**.