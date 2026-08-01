**Chào mừng trở lại**, và trong bài học này, tôi muốn giới thiệu một dịch vụ ngày càng xuất hiện nhiều hơn trong kỳ thi: **Database Migration Service**, còn được gọi là **DMS**. Bài học này là phần mở rộng từ bài học trong khóa học **Associate Architect**, vì vậy dù bạn đã học khóa đó và xem bài học trước, bạn vẫn nên xem đầy đủ bài học này.

Sản phẩm này không chỉ xuất hiện trong kỳ thi, mà nếu bạn làm việc với vai trò **Solutions Architect** trong môi trường AWS và dự án liên quan đến cơ sở dữ liệu, bạn sẽ sử dụng dịch vụ này rất nhiều. Đây là thứ bạn cần nắm vững dù thế nào đi nữa. Hãy bắt đầu ngay.

### Độ phức tạp của việc di chuyển cơ sở dữ liệu

Việc di chuyển cơ sở dữ liệu là công việc phức tạp. Thông thường, nếu loại trừ các công cụ của nhà cung cấp, đây là quy trình thủ công từ đầu đến cuối. Nó thường bao gồm việc thiết lập **replication** (rất phức tạp), hoặc thực hiện **backup tại một thời điểm** rồi khôi phục sang cơ sở dữ liệu đích. Nhưng làm thế nào để xử lý các thay đổi xảy ra giữa lúc backup và khi cơ sở dữ liệu mới đi vào hoạt động? Làm thế nào để di chuyển giữa các loại cơ sở dữ liệu khác nhau? Đây chính là những vấn đề mà **DMS** giải quyết. Về bản chất, đây là dịch vụ **di chuyển cơ sở dữ liệu được quản lý**.

### Kiến trúc cơ bản của DMS

Khái niệm khá đơn giản. Bắt đầu với một **replication instance** chạy trên **EC2**. Instance này chạy một hoặc nhiều **replication tasks**. Bạn cần định nghĩa **source endpoint** và **destination endpoint** trỏ đến cơ sở dữ liệu nguồn và đích. Hạn chế duy nhất của dịch vụ là **một trong hai endpoint phải nằm trong AWS**. Bạn không thể dùng sản phẩm này để di chuyển giữa hai cơ sở dữ liệu on-premises.

Bạn không cần phải có kinh nghiệm thực tế sử dụng sản phẩm, vì sẽ có bài demo riêng trong phần này để bạn thực hành. Trong bài lý thuyết này, chúng ta tập trung vào **kiến trúc**.

Về mặt kiến trúc, bạn bắt đầu với một cơ sở dữ liệu nguồn và một cơ sở dữ liệu đích, trong đó **ít nhất một cái phải nằm trong AWS**. Các cơ sở dữ liệu có thể sử dụng nhiều engine tương thích như: **MySQL, Aurora, Microsoft SQL, MariaDB, MongoDB, PostgreSQL, Oracle, Azure SQL** và nhiều loại khác.

Ở giữa là **Database Migration Service (DMS)**, sử dụng một **replication instance** – về bản chất là một EC2 instance với phần mềm di chuyển và khả năng giao tiếp với dịch vụ DMS. Trên instance này, bạn định nghĩa các **replication tasks**, và mỗi replication instance có thể chạy nhiều task. Task định nghĩa tất cả các tùy chọn liên quan đến việc di chuyển. Về mặt kiến trúc, hai yếu tố quan trọng nhất là **source endpoint** và **destination endpoint**, lưu trữ thông tin để replication instance và task có thể truy cập cơ sở dữ liệu nguồn và đích.

Một task về cơ bản di chuyển dữ liệu từ cơ sở dữ liệu nguồn (sử dụng thông tin trong source endpoint) sang cơ sở dữ liệu đích (sử dụng thông tin trong destination endpoint). Giá trị của DMS nằm ở cách nó xử lý các kiểu di chuyển này.

### Ba loại job chính trong DMS

1. **Full Load** Dùng để di chuyển dữ liệu hiện có. Nếu bạn có thể chấp nhận thời gian ngừng hoạt động đủ lâu để sao chép toàn bộ dữ liệu, đây là lựa chọn phù hợp. Tùy chọn này chỉ di chuyển dữ liệu từ nguồn sang đích và tạo bảng khi cần thiết.
2. **Full Load + CDC** **CDC** là viết tắt của **Change Data Capture**. Phương pháp này di chuyển dữ liệu hiện có và đồng thời sao chép các thay đổi đang diễn ra. Nó thực hiện Full Load, đồng thời bắt các thay đổi trên nguồn. Sau khi Full Load hoàn tất, các thay đổi đã bắt được sẽ được áp dụng lên đích. Khi quá trình áp dụng thay đổi đạt trạng thái ổn định, bạn có thể tắt ứng dụng, để các thay đổi còn lại chảy sang đích, rồi khởi động lại ứng dụng và trỏ chúng sang cơ sở dữ liệu đích mới.
3. **CDC Only** Chỉ sao chép các thay đổi dữ liệu. Trong một số trường hợp, việc sao chép dữ liệu hiện có bằng phương pháp khác ngoài AWS DMS sẽ hiệu quả hơn. Một số cơ sở dữ liệu như **Oracle** có công cụ export/import riêng, và trong những trường hợp đó, có thể dùng công cụ đó để di chuyển dữ liệu ban đầu, rồi dùng DMS chỉ để sao chép các thay đổi bắt đầu từ thời điểm bulk load. **CDC-only** rất hiệu quả khi bạn cần chuyển dữ liệu số lượng lớn bằng cách khác ngoài DMS.

### Schema Conversion Tool (SCT)

**DMS** không hỗ trợ sẵn việc chuyển đổi schema, nhưng AWS có công cụ riêng gọi là **Schema Conversion Tool (SCT)**. Mục đích duy nhất của công cụ này là thực hiện chỉnh sửa hoặc chuyển đổi schema giữa các phiên bản hoặc các engine cơ sở dữ liệu khác nhau. Đây là công cụ mạnh mẽ thường đi kèm với các cuộc di chuyển bằng DMS.

**DMS** là công cụ tuyệt vời để di chuyển cơ sở dữ liệu từ on-premises lên AWS. Đây là công cụ bạn sẽ dùng cho hầu hết các cuộc di chuyển cơ sở dữ liệu lớn, vì vậy với tư cách Solutions Architect, bạn cần hiểu nó từ đầu đến cuối.

### Gợi ý cho kỳ thi

Trong kỳ thi, nếu gặp bất kỳ tình huống di chuyển cơ sở dữ liệu nào, miễn là **một trong hai cơ sở dữ liệu nằm trong AWS** và không liên quan đến loại database lạ không được hỗ trợ, bạn có thể mặc định chọn **DMS**. Đây luôn là lựa chọn an toàn cho các câu hỏi về di chuyển cơ sở dữ liệu. Nếu câu hỏi đề cập đến **di chuyển không downtime**, thì chắc chắn nên chọn DMS.

### Chi tiết thêm về SCT

**Schema Conversion Tool (SCT)** là ứng dụng độc lập, chỉ được dùng khi chuyển đổi từ một engine cơ sở dữ liệu sang engine khác. Nó được dùng trong các cuộc di chuyển khi engine nguồn và đích không tương thích. Một trường hợp sử dụng khác là các cuộc di chuyển quy mô lớn khi bạn cần cách chuyển dữ liệu từ on-premises lên AWS khác với data link thông thường.

**Lưu ý quan trọng**: SCT **không** được dùng khi di chuyển giữa các engine tương thích. Ví dụ: di chuyển từ MySQL on-premises sang RDS MySQL trên AWS → engine giống nhau → **không dùng SCT**.

SCT hỗ trợ cả cơ sở dữ liệu **OLTP** (MySQL, Microsoft SQL, Oracle) và **OLAP** (Teradata, Oracle, Vertica, Greenplum).

Ví dụ sử dụng SCT:

- On-premises **Microsoft SQL** → AWS **RDS MySQL** (engine thay đổi).
- On-premises **Oracle** → AWS **Aurora** (engine thay đổi).

### Kết hợp DMS + SCT + Snowball cho di chuyển lớn

Đối với các cuộc di chuyển quy mô lớn (nhiều terabyte), việc truyền dữ liệu qua mạng thường không tối ưu vì tốn thời gian và chiếm băng thông. DMS có thể kết hợp với dòng sản phẩm **Snowball** để truyền dữ liệu số lượng lớn.

Quy trình hoạt động:

- **Bước 1**: Dùng **Schema Conversion Tool** để trích xuất dữ liệu từ cơ sở dữ liệu và lưu cục bộ, sau đó chuyển dữ liệu đó lên thiết bị **Snowball** đã đặt từ AWS.
- **Bước 2**: Gửi thiết bị về AWS. AWS nạp dữ liệu vào bucket **S3**, rồi **DMS** di chuyển từ S3 sang cơ sở dữ liệu đích.

Nếu dùng **Change Data Capture**, bạn cũng có thể di chuyển các thay đổi sau lần bulk transfer ban đầu. Các thay đổi này cũng đi qua S3 làm trung gian trước khi được DMS ghi vào đích.

Thông thường DMS truyền dữ liệu qua mạng (Direct Connect, VPN hoặc VPC peer). Nhưng nếu khối lượng dữ liệu quá lớn so với đường truyền mạng hiện có, bạn có thể đặt Snowball và kết hợp DMS + SCT để truyền nhanh và hiệu quả hơn.

**Quy tắc cần nhớ cho kỳ thi**: **SCT chỉ được dùng khi engine cơ sở dữ liệu thay đổi**. Trong trường hợp dùng với Snowball, lý do là bạn đang chuyển đổi dữ liệu sang định dạng file chung để truyền qua Snowball, nên về bản chất vẫn là thay đổi engine.

### Kết luận

Đó là toàn bộ nội dung tôi muốn trình bày trong bài học này. Đây là phần mở rộng so với mức độ Associate Architect. Bạn sẽ được thực hành sản phẩm này trong bài demo, còn bài học này chỉ tập trung vào lý thuyết.

Cảm ơn bạn đã theo dõi. Hãy hoàn thành bài học này, và khi sẵn sàng, tôi rất mong được gặp lại bạn ở bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Database Migration Service (DMS)
- Replication Instance + Replication Tasks
- Source & Destination Endpoints
- 3 loại migration: Full Load, Full Load + CDC, CDC Only
- Schema Conversion Tool (SCT)
- Kết hợp DMS + SCT + Snowball
- Quy tắc thi: khi nào dùng DMS và SCT

**Notes (Chi tiết):**

- **DMS** là dịch vụ quản lý việc di chuyển cơ sở dữ liệu. Yêu cầu **ít nhất một endpoint nằm trong AWS**.
- **Replication Instance** chạy trên EC2, chứa các **Replication Tasks**.
- **Source/Destination Endpoints** lưu thông tin kết nối để task truy cập DB nguồn và đích.
- **Full Load**: Chỉ copy dữ liệu hiện có (chấp nhận downtime).
- **Full Load + CDC**: Copy dữ liệu + bắt thay đổi liên tục → hỗ trợ **zero-downtime**.
- **CDC Only**: Chỉ bắt thay đổi (khi đã bulk load bằng công cụ khác).
- **SCT** chỉ dùng khi **engine thay đổi** (ví dụ SQL Server → MySQL, Oracle → Aurora). Không dùng khi engine giống nhau (MySQL → RDS MySQL).
- Với dữ liệu lớn (multi-TB): dùng **SCT** extract → **Snowball** → S3 → **DMS** load vào target. CDC cũng đi qua S3.
- Trong kỳ thi: gặp câu hỏi migration DB (một bên trong AWS) → mặc định chọn **DMS**. Câu hỏi **no-downtime** → chắc chắn dùng DMS.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu **AWS Database Migration Service (DMS)** – dịch vụ quản lý việc di chuyển cơ sở dữ liệu. DMS dùng **Replication Instance** trên EC2 chạy các task, kết nối qua **Source/Destination Endpoints**. Có 3 kiểu migration: **Full Load**, **Full Load + CDC** (hỗ trợ zero-downtime) và **CDC Only**. **Schema Conversion Tool (SCT)** chỉ dùng khi engine thay đổi. Với dữ liệu lớn, kết hợp **DMS + SCT + Snowball**. Trong thi, DMS là lựa chọn mặc định an toàn cho hầu hết câu hỏi di chuyển cơ sở dữ liệu.