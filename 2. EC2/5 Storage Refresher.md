Trong bài này chúng ta sẽ tóm tắt nhanh một vài thông tin quan trọng liên quan đến lưu trữ.
Bao gồm:
- Block Storage
- File Storage
- Object Storage
- IO Block Size
- IOPS
- Throughput

Trong bài kiểm tra mong đợi bạn biết loại storage thích hợp cho một tình huống cụ thể. Cho nên trước khi chúng ta sang các bài học cụ thể về AWS storage, tôi muốn tóm tắt nhanh.

Hãy bắt đầu bằng cách nói về một vài thuật ngữ quan trọng về Storage.
**Thuật ngữ quan trọng**:
1: **Direct (local) Attached Storage** - Storage ở trên EC2 host, đây là ổ cứng kết nối trực tiếp với Ec2 host. Nó gọi là Instance Store, lưu trữ gắn trực tiếp thông thường siêu nhanh bởi vì nó gắn trực tiếp vào phần cứng. Nhưng nó lại chịu thiệt hại từ một vài vấn đề. Nếu ổ cứng, phần cứng lỗi storage có thể bị mất. Nếu một EC2 instance di chuyển giữa các host storage có thể mất. Giải pháp thay thế là lưu trữ gắn qua mạng (Network attached storage)
2: **Network Attached Storage**: Nơi mà các volume được tạo ra, và được gắn tới một thiết bị qua mạng. Trong môi trường tại chỗ (on permises) loại này sử dụng các giao thức như là ISCSI hoặc Fiber channel, trong AWS nó sử dụng một sản phẩm gọi là Elastic Block Store (EBS), Network Storage nói chung là có khả năng phục hồi cao, và nó tách biệt khỏi phần cứng của instance, cho nên network storage có thể vượt qua các sự cố khi EC2 host bị ảnh hưởng.
3: **Ephemeral storage**: Nó là một bộ nhớ tạm, không cần tồn tại lâu, là loại mà bạn không thể tin tưởng sự bền vững.
4: **Persistent storage**: Lưu trữ tồn tại như một thực thể riêng biệt, nó vẫn tồn tại ngay cả sau khi thiết bị nó gắn vào không còn nữa (instance).
**Một ví dụ của Ephemeral storage là Instance Store**, đây là ổ cứng vật lý gắn trực tiếp vào EC2 host, nó không bền vững.
Ngược lại là **Persistent storage (lưu trữ bền vững) như EBS**. Hãy nhớ kỹ điều này. Bạn sẽ gặp các câu hỏi kiểm tra kiến thức về việc phần biệt loại storage nào là Ephemeral storage và loại nào là persistent storage.

#### Tiếp theo sẽ là ba loại lưu trữ chính có sẵn trong AWS:
Loại lưu trữ sẽ quyết định cách lưu trữ bạn hoặc server sẽ thấy và nó có thể được sử dụng để làm gì.
1: Loại đầu tiên là **Block Storage** (lưu trữ khối), với block storage bạn tạo ra một volume (ổ đĩa). Ví dụ bên trong EBS chính là một volume của Block Storage, 1 volume gồm rất nhiều block đã được đánh địa chỉ, số lượng block nhiều hay ít phụ thuộc vào kích thước volume. Block storage đơn giản chỉ là một tập hợp các khối có thể địa chỉ được, nó có thể được hiển thị dưới dạng volume logic hoặc như một ổ vật lý trống. 
Khi đưa một block storage cho server (dù ổ vật lý hay volume) hệ điều hành sẽ tạo file system lên trên ví dụ (NTFS trên window, ext3/ext4 trên Linux) và mount nó thành ổ C hoặc root.
Block storage có thể là ổ HDD, SSD, hoặc các thiết bị vật lý. Trong môi trường mạng Network Attached Storage (NAS) hoặc Storage Area Network (SAN) cũng cung cấp block storage qua mạng.
Điểm quan trọng là block storage không có cấu trúc sẵn. Hệ điều hành sẽ tự động tạo file system và mount để sử dụng.
Trong AWS, với block storage bạn có thể mount một volume ví dụ EBS volume, bạn cũng có thể boot instance từ EBS volume. Vì vậy hầu hết EC2 instance đều sử dụng EBS volume làm boot volume để chứa hệ điều hành và khởi động instance.
2: **File Storage (lưu trữ tệp)** cung cấp bởi một file server, như server trong môi trường on permises. Nó được cung cấp dưới dạng file system hoàn chỉnh, với cấu trúc thư mục sẵn có từ đầu. Bạn có thể truy cập, duyệt, tạo mới...Bạn truy cập file bằng cách biết cấu trúc thư mục, duyệt qua các thư mục, tìm vị trí file. 
Bạn không thể boot từ File Storage bởi vì nó đưa cho bạn file system hoàn chỉnh sẵn. bạn chỉ có thể tương tác ở mức cao, hệ điều hành không có quyền truy cập vào thiết bị lưu trữ thật sự.
**Tóm lại**: File Storage chỉ cho bạn sử dụng file và thư mục qua mạng, chứ không cho bạn quyền “sờ mó” trực tiếp vào phần cứng lưu trữ. Boot OS lại đòi hỏi quyền truy cập sâu vào phần cứng → nên không thể boot từ File Storage.
Vì vậy file storage **có thể mount** vào instance nhưng **không thể dùng để boot instance**.

3: **Object Storage** (lưu trữ đối tượng)

Đây là một hệ thống **rất trừu tượng**. Bạn chỉ đơn giản là lưu các **object** (đối tượng). Không có cấu trúc thư mục nào cả.
 
Nó chỉ là một **bộ sưu tập phẳng** (flat collection) của các object, và **một object có thể là bất cứ thứ gì**. Object có thể đính kèm **metadata** (dữ liệu mô tả), nhưng khi muốn lấy một object ra, bạn thường chỉ cần cung cấp một **key** (khóa). Đổi lại, hệ thống sẽ trả về **value** của object đó — tức là dữ liệu thực tế.

Object có thể là:

- Dữ liệu nhị phân (binary data)
- Ảnh
- Video
- Ảnh mèo (như con Whiskers ở giữa ví dụ)
- Hay bất kỳ loại dữ liệu nào khác

**Điểm quan trọng nhất** của object storage là nó **hoàn toàn phẳng** (flat storage). Nó không có cấu trúc thư mục. Bạn chỉ có một **container** (thùng chứa). Trong AWS, đó chính là **S3**, và bên trong một **S3 Bucket** là các object.

**Lợi ích lớn** của object storage là nó **siêu mở rộng** (highly scalable). Nó có thể được hàng nghìn hoặc thậm chí hàng triệu người truy cập **đồng thời**.

Tuy nhiên, object storage **không mount được** vào file system của hệ điều hành, và **chắc chắn không boot được**.

Vì vậy, rất quan trọng là bạn phải hiểu rõ sự khác biệt giữa **ba loại lưu trữ chính** này."

**Tóm tắt nhanh 3 loại lưu trữ (để bạn dễ so sánh):**

| Loại       | Cấu trúc                  | Mount được? | Boot được? | Dùng cho gì?                    | Ví dụ AWS |
| ---------- | ------------------------- | ----------- | ---------- | ------------------------------- | --------- |
| **Block**  | Khối thô (raw blocks)     | Có          | Có         | Ổ cứng, boot OS, database       | EBS       |
| **File**   | File system sẵn (thư mục) | Có          | Không      | Chia sẻ file, NAS               | EFS       |
| **Object** | Phẳng, không cấu trúc     | Không       | Không      | Lưu trữ lớn, web, backup, media | **S3**    |
**Vì vậy, nói chung trong môi trường on-premises lẫn trong AWS:**
- Nếu bạn muốn dùng lưu trữ để **boot** (khởi động) hệ điều hành → thì **phải là Block Storage**.
- Nếu bạn muốn dùng **lưu trữ hiệu suất cao** bên trong hệ điều hành → thì **cũng là Block Storage**
	- (Chạy database (MySQL, PostgreSQL, Oracle))
- Nếu bạn muốn **chia sẻ một file system** cho nhiều server khác nhau, nhiều client, hoặc cho nhiều dịch vụ cùng truy cập → thì thường sẽ dùng **File Storage**.
	- Nhiều web server cùng đọc chung thư mục code hoặc file upload (Amazon EFS).
- Nếu bạn muốn **truy cập quy mô lớn** (read/write) dữ liệu object với khả năng mở rộng cực lớn, ví dụ bạn đang làm một ứng dụng webscale và lưu trữ bộ sưu tập ảnh mèo lớn nhất thế giới → thì bạn sẽ dùng **Object Storage**, vì nó **gần như vô hạn về khả năng mở rộng** (almost infinitely scalable).

**Storage Performance**
Có ba loại bạn sẽ thấy, khi bất kỳ ai đề cập đến hiệu suất lưu trữ.
1: **IO (block) size**: Đây là kích thước của mỗi lệnh đọc/ghi (mỗi I/O operation).
2: **IOPS**: Số lượng lệnh đọc/ghi mà ổ cứng/SSD/NVMe có thể thực hiện được trong 1 giây
3: **Throughput** (Thông lượng): Lượng dữ liệu chuyển được trong 1 đơn vị thời gian (thường là MB/s hoặc GB/s)

Khi chúng ta nói về **ba khía cạnh hiệu suất** này, hãy nhớ rằng một thiết bị lưu trữ vật lý — dù là **ổ cứng HDD** hay **SSD** — không phải là thứ duy nhất tham gia vào chuỗi lưu trữ.

Khi bạn đang đọc hoặc ghi dữ liệu, quá trình bắt đầu từ **ứng dụng**, sau đó là **hệ điều hành và hệ thống lưu trữ con**, rồi đến **cơ chế vận chuyển** để đưa dữ liệu đến ổ đĩa (có thể là mạng hoặc bus lưu trữ cục bộ như **SATA**). Tiếp theo là **giao diện lưu trữ trên ổ đĩa**, bản thân **ổ đĩa**, và **công nghệ** mà ổ đĩa sử dụng.

**Tất cả đều là các thành phần của chuỗi đó.** Bất kỳ điểm nào trong chuỗi đều có thể trở thành **yếu tố giới hạn** (bottleneck), và chính **mắt xích yếu nhất** (lowest common denominator) của toàn bộ chuỗi sẽ quyết định **hiệu suất cuối cùng**.

**1. IO Size (Block Size) – Kích thước khối**

**IO size** (hay block size) là **kích thước của các khối dữ liệu** mà bạn đang ghi vào đĩa. Nó được biểu thị bằng **kilobytes (KB)** hoặc **megabytes (MB)**, và có thể dao động từ kích thước rất nhỏ đến rất lớn.

Ứng dụng có thể chọn ghi hoặc đọc dữ liệu với **bất kỳ kích thước nào**. Nó sẽ hoặc là dùng đúng block size làm kích thước tối thiểu, hoặc dữ liệu sẽ bị **chia nhỏ** thành nhiều khối khi ghi vào đĩa.

**Ví dụ:** Nếu block size của hệ thống lưu trữ là **16KB**, và bạn ghi **64KB** dữ liệu, thì nó sẽ sử dụng **4 blocks**.

**2. IOPS – Số thao tác IO mỗi giây**

**IOPS** đo số lượng **thao tác đọc/ghi (IO operations)** mà hệ thống lưu trữ có thể thực hiện được trong **1 giây**.

Nói cách khác, đó là số lần đọc hoặc ghi mà ổ đĩa hay hệ thống lưu trữ có thể xử lý trong một giây.

**So sánh với xe hơi:** IOPS giống như **số vòng tua máy mỗi giây** mà động cơ có thể tạo ra (với kích thước bánh xe mặc định).

Một số loại ổ (media types) giỏi cung cấp **IOPS cao**, trong khi một số loại khác lại giỏi **throughput cao**.

Nếu dùng **lưu trữ qua mạng (network storage)** thay vì lưu trữ cục bộ, thì **mạng** cũng sẽ ảnh hưởng đến số IOPS có thể đạt được. Độ trễ cao giữa thiết bị và hệ thống lưu trữ mạng có thể làm giảm mạnh số thao tác mỗi giây.

**3. Throughput – Thông lượng**

**Throughput** là **tốc độ dữ liệu** mà hệ thống lưu trữ có thể ghi nhận được trên một vùng lưu trữ (có thể là ổ vật lý hoặc volume).

Thông thường, throughput được đo bằng **megabytes per second (MB/s)**. Nó có mối quan hệ chặt chẽ với **block size** và **IOPS**.

**Công thức đơn giản:** **Throughput ≈ IOPS × Block Size**

**Ví dụ:**

- Hệ thống có block size **16KB**, đạt **100 IOPS** → Throughput tối đa = **1.6 MB/s**.
- Nhưng nếu ứng dụng chỉ ghi dữ liệu theo khối **4KB**, và giới hạn là 100 IOPS → Thì bạn chỉ đạt **400 KB/s** throughput.

**Kết luận:** Để đạt được **throughput tối đa**, bạn phải dùng **đúng block size** mà nhà cung cấp lưu trữ khuyến nghị, đồng thời đẩy mạnh số IOPS vào hệ thống.

![[Pasted image 20260514212322.png]]

### **Quy tắc chọn Block Size dễ nhớ**

**Dùng Block Size NHỎ (4KB – 16KB) khi:**

- Công việc có **nhiều random IO** (đọc/ghi ngẫu nhiên)
- Cần **độ trễ thấp (low latency)**
- Ứng dụng đọc/ghi dữ liệu nhỏ lẻ (ví dụ: 4KB, 8KB, 16KB)
- Database, web server, máy ảo, email server, desktop

**Lợi ích:** IOPS cao, latency thấp, ít lãng phí khi đọc dữ liệu nhỏ.

**Dùng Block Size LỚN (64KB – 1MB) khi:**

- Công việc chủ yếu **sequential** (đọc/ghi dữ liệu liên tục, theo thứ tự)
- Cần **throughput cao** (tốc độ MB/s hoặc GB/s lớn)
- Dữ liệu thường lớn (file video, backup, copy folder lớn)
- Backup, media, big data, streaming, log archiving

**Lợi ích:** Throughput rất cao, giảm số lượng IO operations, giảm overhead.