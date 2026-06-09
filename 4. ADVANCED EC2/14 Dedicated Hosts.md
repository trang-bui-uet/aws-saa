**Chào mừng bạn quay trở lại.** Trong bài học này, tôi muốn nói về **Dedicated Hosts** và **Dedicated Instances** — hai tính năng trong AWS cung cấp các mức độ cách ly vật lý khác nhau cho EC2 instances.

Khi bạn khởi chạy một EC2 instance thông thường, nó chạy trên mô hình **Shared Tenancy**. Điều này có nghĩa là instance của bạn chạy trên một máy chủ vật lý trong data center của AWS, nhưng máy chủ đó cũng đang chạy các instance của khách hàng khác. Các instance của bạn được cách ly an toàn ở mức phần mềm nhờ hypervisor của AWS, nhưng về mặt vật lý, bạn đang chia sẻ phần cứng máy chủ với các khách hàng khác.

Đối với hầu hết workload, **Shared Tenancy** là lựa chọn phù hợp vì tính **chi phí thấp** và **linh hoạt**. Tuy nhiên, có những trường hợp bạn có yêu cầu nghiêm ngặt về quy định pháp lý, chính sách tuân thủ, hoặc thỏa thuận cấp phép phần mềm phức tạp không cho phép chia sẻ phần cứng vật lý với khách hàng khác. Đây chính là lúc **Dedicated Instances** và **Dedicated Hosts** phát huy tác dụng.

---

**Dedicated Instances**

**Dedicated Instances** là các EC2 instances chạy trên **single-tenant hardware** (phần cứng dành riêng cho một tenant).

Khi bạn khởi chạy Dedicated Instance, AWS đảm bảo instance sẽ chạy trên phần cứng vật lý **chỉ dành riêng cho tài khoản AWS của bạn**. Không có instance nào của khách hàng khác được đặt trên cùng máy chủ vật lý đó.

Tuy nhiên, với **Dedicated Instances**, bạn **không có quyền nhìn thấy hoặc kiểm soát** phần cứng vật lý cụ thể. Mỗi khi bạn stop và start instance, AWS có thể di chuyển nó sang một máy chủ vật lý khác trong data center. Nó vẫn là phần cứng dành riêng cho tài khoản của bạn, nhưng bạn không kiểm soát được máy chủ nào.

**Dedicated Instances** được tính phí theo instance, và có thêm một khoản phí theo region cho mỗi giờ mà ít nhất một Dedicated Instance đang chạy trong region đó.

---

**Dedicated Hosts**

**Dedicated Host** là một máy chủ vật lý thực sự bên trong data center của AWS, được **dành riêng hoàn toàn cho bạn**. Khi bạn allocate một Dedicated Host, bạn không chỉ mua một instance riêng — bạn đang thuê toàn bộ máy chủ vật lý.

Khi allocate Dedicated Host, bạn có **quyền nhìn thấy đầy đủ** các thông tin của máy chủ vật lý, bao gồm:

- Số lượng physical sockets
- Số lượng physical cores
- Host ID

Bạn có toàn quyền kiểm soát cách các instance được đặt lên host đó. Bạn có thể khởi chạy nhiều EC2 instances trên cùng một Dedicated Host cho đến khi hết capacity vật lý.

Đặc biệt, bạn có thể cấu hình **instance affinity**, giúp đảm bảo rằng instance sẽ luôn khởi động lại trên **chính xác cùng Dedicated Host** đó khi stop và start.

**Dedicated Hosts** chủ yếu được dùng để đáp ứng hai nhu cầu:

1. Yêu cầu tuân thủ doanh nghiệp hoặc quy định pháp lý về cách ly máy chủ vật lý.
2. **Bring Your Own License (BYOL)** — cho phép bạn mang giấy phép phần mềm của riêng mình vào AWS.

Nhiều phần mềm doanh nghiệp như **Microsoft Windows Server**, **Microsoft SQL Server**, hoặc **Oracle Database** có mô hình cấp phép theo physical sockets hoặc physical cores. Với Dedicated Hosts, bạn có thể theo dõi chính xác số cores và sockets để sử dụng giấy phép hiện có, giúp giảm đáng kể chi phí licensing.

Về thanh toán, **Dedicated Hosts** được tính phí theo host, bất kể bạn chạy bao nhiêu EC2 instances trên host đó. Miễn là host vẫn được allocate cho tài khoản, bạn phải trả phí cho toàn bộ máy chủ vật lý.

---

**Tóm tắt các điểm khác biệt quan trọng cho kỳ thi:**

|Tiêu chí|**Dedicated Instances**|**Dedicated Hosts**|
|---|---|---|
|Mức độ cách ly|Phần cứng dành riêng cho tài khoản|Toàn bộ máy chủ vật lý dành riêng|
|Quyền kiểm soát phần cứng|Không có quyền nhìn thấy/kiểm soát server cụ thể|Có đầy đủ quyền nhìn thấy sockets, cores, host ID|
|Kiểm soát vị trí instance|Không có|Có (Instance Affinity)|
|Hỗ trợ BYOL (Bring Your Own License)|Không hỗ trợ|Hỗ trợ tốt (theo sockets/cores)|
|Cách tính phí|Theo instance + phí region|Theo host (toàn bộ máy chủ)|

**Dedicated Instances** chạy trên phần cứng dành riêng cho tài khoản, nhưng bạn không có quyền kiểm soát máy chủ cụ thể và không hỗ trợ licensing theo sockets/cores.

**Dedicated Hosts** cung cấp toàn bộ máy chủ vật lý với quyền kiểm soát đầy đủ và hỗ trợ mô hình **Bring Your Own License (BYOL)**.

---

Đó là toàn bộ kiến thức lý thuyết về **Dedicated Hosts** và **Dedicated Instances** cần biết cho kỳ thi Associate level. Bạn có thể xem hết video này, và khi sẵn sàng, tôi sẽ gặp bạn ở bài học tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Shared Tenancy (mặc định)
- Dedicated Instances: Single-tenant hardware, không kiểm soát server cụ thể
- Dedicated Hosts: Thuê toàn bộ máy chủ vật lý, full visibility (sockets/cores)
- Instance Affinity
- BYOL (Bring Your Own License)
- Cách tính phí: Instance vs Host

**Notes (Chi tiết):**

- **Dedicated Instances**: Phần cứng dành riêng cho tài khoản, nhưng AWS có thể di chuyển instance sang server khác khi stop/start. Không hỗ trợ BYOL theo sockets/cores. Phí theo instance + phí region.
- **Dedicated Hosts**: Thuê toàn bộ physical server. Có quyền xem sockets, cores, host ID. Có thể cấu hình Instance Affinity. Hỗ trợ BYOL rất tốt cho Windows, SQL Server, Oracle. Phí theo host.
- Dedicated Hosts phù hợp khi cần tuân thủ quy định hoặc muốn sử dụng giấy phép phần mềm hiện có để tiết kiệm chi phí.

**Summary (Tóm tắt ngắn):** Bài học phân biệt **Dedicated Instances** và **Dedicated Hosts**. Dedicated Instances cung cấp single-tenant hardware nhưng không cho phép kiểm soát server cụ thể và không hỗ trợ BYOL. Dedicated Hosts cho phép thuê toàn bộ máy chủ vật lý, có full visibility, kiểm soát placement, hỗ trợ Instance Affinity và BYOL — rất hữu ích cho compliance và giảm chi phí licensing.