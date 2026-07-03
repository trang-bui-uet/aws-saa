![[Pasted image 20260701222129.png]]

Chào mừng bạn quay trở lại! Trong video này, tôi muốn nói về **loại hosted zone** khác có sẵn trong Route 53, đó chính là **private hosted zone**. Hãy bắt đầu ngay thôi.

**Private hosted zone** hoạt động giống hệt như **public hosted zone**, chỉ khác là nó **không công khai**. Thay vì công khai, nó được liên kết với các **VPC** trong AWS, và chỉ có thể truy cập được từ bên trong những **VPC** đã được liên kết với nó. Bạn có thể liên kết private hosted zone với các VPC trong tài khoản của mình thông qua giao diện console, CLI và API — thậm chí với VPC ở các tài khoản khác nếu dùng CLI và API. Tất cả những thứ còn lại đều giống nhau: bạn có thể tạo **resource records** trong đó, và những records này có thể được phân giải (resolvable) bên trong các VPC.

Ngay cả kỹ thuật **split-view** hoặc **split-horizon DNS** cũng có thể sử dụng được. Đây là khi bạn có public hosted zone và private hosted zone **cùng tên**. Nghĩa là bạn có thể có phiên bản zone khác nhau cho người dùng nội bộ so với người dùng công khai. Bạn có thể làm điều này nếu muốn **intranet** của công ty chạy trên cùng một địa chỉ với website công khai. Khi người dùng ở nội bộ sẽ thấy intranet, còn người ngoài sẽ thấy website công khai. Hoặc nếu bạn muốn một số hệ thống chỉ có thể truy cập qua DNS của doanh nghiệp nhưng chỉ bên trong môi trường nội bộ.

Bây giờ hãy xem nhanh cách **private hosted zone** hoạt động qua kiến trúc end-to-end để bạn hình dung rõ hơn.

**Kiến trúc End-to-End**

Chúng ta bắt đầu với một **private hosted zone**, và giống như public zone, chúng ta có thể tạo các records bên trong zone này. Từ **internet công khai**, người dùng có thể thực hiện các truy vấn DNS bình thường cho những thứ như netflix.com hay catagram.io. Nhưng **private hosted zone** thì không thể truy cập từ internet công khai. Tuy nhiên, nó có thể được truy cập từ các **VPC**.

Giả sử cả ba VPC này đều có dịch vụ bên trong và sử dụng **Route 53 resolver** (địa chỉ VPC + 2). Bất kỳ VPC nào được liên kết với private hosted zone đều có thể truy cập zone đó qua resolver. Các VPC không được liên kết sẽ gặp vấn đề giống như người dùng trên internet công khai bên trái: không thể truy cập.

**Private hosted zone** rất tuyệt vời khi bạn cần cung cấp records qua DNS nhưng thông tin nhạy cảm và chỉ nên truy cập được từ các VPC nội bộ. Hãy nhớ rằng, để truy cập private hosted zone, dịch vụ phải chạy bên trong một VPC và VPC đó phải được liên kết với private hosted zone.

**Split-View / Split-Horizon DNS**

Trước khi kết thúc bài học ngắn này, hãy nói về **split-view** hoặc **split-horizon DNS**. Hãy xem tình huống sau: bạn có một VPC đang chạy **Amazon WorkSpace**, và để hỗ trợ một số ứng dụng kinh doanh, bạn tạo private hosted zone chứa một số records. Private hosted zone được liên kết với VPC 1 bên phải, nghĩa là WorkSpace có thể dùng Route 53 resolver để truy cập private hosted zone. Ví dụ: truy cập record “accounting” được lưu trong private hosted zone.

Private hosted zone không thể truy cập từ internet công khai, nhưng **split-view** cho phép chúng ta tạo một **public hosted zone** có **cùng tên**. Public hosted zone này có thể chỉ chứa một tập con các records so với private hosted zone. Từ internet công khai, truy cập public hosted zone sẽ hoạt động như bình thường:

1. Qua **ISP resolver server**
2. Sau đó đến **DNS root servers**
3. Tiếp theo đến **.org TLD servers**
4. Và cuối cùng đến **name servers của animalsforlife** do Route 53 cung cấp

Bất kỳ records nào trong public hosted zone đều có thể truy cập được. Nhưng các records chỉ có trong private hosted zone (như “accounting” trong ví dụ) thì sẽ **không thể truy cập** từ internet công khai.

Đây là một kiến trúc phổ biến khi bạn muốn sử dụng **cùng một domain name** cho truy cập công khai và nội bộ, nhưng có bộ records khác nhau cho từng bên. Đây là kỹ thuật bạn cần thành thạo khi làm **architect** thiết kế giải pháp, **developer** tích hợp DNS vào ứng dụng, hoặc **engineer** triển khai trong AWS.

Đó là tất cả những gì tôi muốn chia sẻ về lý thuyết **private hosted zone**. Hãy hoàn thành video này, và khi bạn sẵn sàng, tôi rất mong được gặp bạn ở video tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Private Hosted Zone trong Route 53
- Liên kết với VPC (cùng tài khoản & cross-account)
- Split-view / Split-horizon DNS
- Kiến trúc End-to-End & Route 53 Resolver
- So sánh Public vs Private Hosted Zone
- Ứng dụng thực tế: Intranet & Internal services

**Notes (Chi tiết):**

- **Private Hosted Zone** hoạt động giống public hosted zone nhưng **chỉ truy cập được từ VPC đã liên kết**, không công khai trên internet.
- Có thể liên kết qua **Console, CLI, API**; hỗ trợ cross-account qua CLI/API.
- **Split-view DNS**: Dùng cùng domain name cho public và private zone → nội bộ thấy records riêng (intranet), công khai thấy records khác (website).
- Kiến trúc: VPC sử dụng Route 53 Resolver (VPC+2) để phân giải private zone. VPC không liên kết thì không truy cập được.
- Ví dụ thực tế: Amazon WorkSpace truy cập record “accounting” nội bộ qua private zone, trong khi public zone chỉ có records công khai.
- Quy trình truy vấn public: ISP resolver → Root servers → TLD servers → Route 53 name servers.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu **Private Hosted Zone** trong Route 53: cách hoạt động, liên kết với VPC, và kỹ thuật **Split-View / Split-Horizon DNS** để dùng cùng domain cho nội bộ và công khai với bộ records khác nhau. Đây là kiến trúc quan trọng cho hệ thống nội bộ an toàn và tích hợp DNS trong AWS.