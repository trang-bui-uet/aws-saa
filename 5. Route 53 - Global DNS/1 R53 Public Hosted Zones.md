![[Pasted image 20260701215902.png]]
Chào mừng bạn quay lại! Trong video này, tôi muốn nói về **Public Hosted Zones** của **Route 53**. Có hai loại DNS zone trong Route 53: **public** và **private**. Trước tiên, chúng ta sẽ tìm hiểu một số thông tin chung, sau đó sẽ đi sâu vào **public hosted zones** cụ thể.

**Hosted zone** là một cơ sở dữ liệu DNS cho một phần của cơ sở dữ liệu DNS toàn cầu — cụ thể là cho một domain, ví dụ như animalsforlife.org. **Route 53** là dịch vụ có khả năng phục hồi toàn cầu. Các name server này được phân bố trên toàn cầu và có cùng bộ dữ liệu, vì vậy ngay cả khi toàn bộ region gặp sự cố, **Route 53** vẫn hoạt động bình thường.

**Hosted zones** được tạo tự động khi bạn đăng ký domain bằng **Route 53**, và bạn đã thấy điều này trước đó trong khóa học khi tôi đăng ký domain animalsforlife.org. Chúng cũng có thể được tạo riêng nếu bạn muốn đăng ký domain ở nơi khác và sử dụng **Route 53** để host nó. Có **phí hàng tháng** để host mỗi hosted zone và **phí cho các query** thực hiện đối với hosted zone đó.

Một zone, dù public hay private, đều chứa **DNS records**. Ví dụ bao gồm:

- **A records**
- Bản tương đương IPv6
- **MX records**
- **NS records**
- **Text records**

Và tôi đã giới thiệu ở mức cơ bản về chúng trước đó trong khóa học.

Tóm lại, **hosted zones** là các cơ sở dữ liệu được tham chiếu thông qua delegation bằng cách sử dụng name server records. Một hosted zone, khi được tham chiếu theo cách này, sẽ trở thành **authoritative** cho domain đó, ví dụ animalsforlife.org. Vì vậy khi bạn đăng ký domain, name server records cho domain đó sẽ được nhập vào top-level domain zone, và chúng trỏ đến name server của bạn. Sau đó, name server của bạn và zone mà chúng host sẽ trở thành authoritative cho domain đó.

Bây giờ, **public hosted zone** là một cơ sở dữ liệu DNS — tức là zone file được host bởi **Route 53** trên các public name server. Điều này có nghĩa là nó có thể truy cập được từ **internet công cộng** và bên trong các VPC bằng **Route 53 resolver**. Về mặt kiến trúc, khi bạn tạo một public hosted zone, **Route 53** sẽ cấp **bốn public name server**, và zone file được host trên những name server này. Để tích hợp nó với hệ thống DNS công cộng, bạn thay đổi name server records cho domain đó để trỏ đến bốn name server của **Route 53**.

Bên trong public hosted zone, bạn tạo **resource records**, đây là các mục dữ liệu thực tế mà DNS sử dụng. Bạn có thể — và tôi sẽ nói chi tiết trong video sắp tới — sử dụng **Route 53** để host zone file cho các domain đăng ký bên ngoài. Ví dụ, bạn có thể dùng Hover hoặc GoDaddy để đăng ký domain, tạo public hosted zone trong **Route 53**, lấy bốn name server được cấp cho zone đó, sau đó qua giao diện Hover hoặc GoDaddy để thêm những name server này vào hệ thống DNS cho domain của bạn. Tôi sẽ giải thích chi tiết cách thức này trong video tương lai.

Về mặt trực quan, đây là cách **public hosted zone** hoạt động. Chúng ta bắt đầu bằng việc tạo một public hosted zone, trong ví dụ này là animalsforlife.org. Việc tạo này sẽ cấp **bốn Route 53 name server** cho zone này, và tất cả các name server này đều có thể truy cập từ **internet công cộng**. Chúng cũng có thể truy cập từ các AWS VPC bằng **Route 53 resolver** — giả sử DNS được bật cho VPC — thì có thể truy cập trực tiếp từ địa chỉ IP nội bộ của VPC đó.

Bên trong hosted zone này, chúng ta có thể tạo một số resource records — trong trường hợp này là một www record, hai MX records cho email, và một text record. Bên trong VPC, phương thức truy cập là trực tiếp: VPC resolver sử dụng địa chỉ **VPC + 2**. Địa chỉ này có thể truy cập từ bất kỳ instance nào bên trong VPC sử dụng nó làm DNS resolver. Vì vậy, chúng có thể query hosted zone giống như query bất kỳ public DNS zone nào bằng **Route 53 resolver**.

Từ góc nhìn public DNS, kiến trúc giống nhau vì cùng sử dụng zone file, nhưng cơ chế hơi khác một chút. DNS bắt đầu từ **DNS root servers**, đây là những server đầu tiên được query bởi resolver server của người dùng. Ví dụ Bob đang dùng laptop, kết nối với ISP DNS resolver server, server này query root servers. Root servers có thông tin về top-level domain .org, nên ISP resolver server có thể query các .org server. Những server này host .org zone file, và zone file này có mục cho animalsforlife.org chứa bốn name server, tất cả đều trỏ đến **Route 53 public name server** cho public hosted zone của animalsforlife.org. Quá trình này được gọi là **"walking the tree"**, và đây là cách bất kỳ host nào trên internet công cộng đều có thể truy cập các records bên trong public hosted zone bằng DNS.

Đó chính là cách **public hosted zones** hoạt động. Chúng chỉ là một zone file được host trên bốn name server do **Route 53** cung cấp. Public hosted zone này có thể được truy cập từ internet công cộng hoặc bất kỳ VPC nào được cấu hình cho phép DNS resolution. Có **chi phí hàng tháng** để host public hosted zone này và một khoản phí nhỏ cho bất kỳ query nào thực hiện đối với nó — gần như không đáng kể trong tổng thể, nhưng với các site có lưu lượng lớn thì cần lưu ý.

Vậy là xong về **public hosted zones**. Đó là tất cả những gì tôi muốn trình bày trong video này về phần lý thuyết. Bạn hãy hoàn thành video này, và khi sẵn sàng, tôi rất mong được gặp bạn trong video tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Route 53 Public Hosted Zones vs Private
- Hosted Zone là cơ sở dữ liệu DNS cho domain
- Tạo tự động khi register domain hoặc tạo riêng
- Bốn public name server được cấp
- Resource records (A, AAAA, MX, NS, TXT…)
- Delegation & Authoritative
- Walking the tree trong public DNS
- Truy cập từ internet và VPC Resolver
- Chi phí (monthly + query)

**Notes (Chi tiết):**

- **Public Hosted Zone** là zone file trên **4 public name server** của Route 53, accessible từ internet công cộng và VPC (qua Route 53 Resolver).
- Khi tạo zone cho domain (ví dụ animalsforlife.org), Route 53 cấp 4 name server → thay đổi NS records tại registrar (GoDaddy, Hover…) để delegation.
- Hosted zone chứa **resource records** như A, MX, TXT… và trở thành **authoritative** cho domain.
- Truy cập nội bộ VPC: dùng địa chỉ VPC+2 (DNS resolver).
- Public DNS: Root servers → TLD (.org) → NS của domain → Route 53 name servers (quá trình "walking the tree").
- **Route 53** globally resilient, name servers phân bố toàn cầu.

**Summary (Tóm tắt ngắn):** Video giới thiệu **Public Hosted Zones** trong **AWS Route 53**: cách tạo, hoạt động, delegation qua 4 name server, truy cập từ internet/VPC, và cơ chế DNS resolution ("walking the tree"). Nhấn mạnh tính toàn cầu, chi phí và cách sử dụng cho domain nội bộ hoặc đăng ký bên ngoài.