Chào mừng trở lại! Trong video này, là video đầu tiên của series, tôi sẽ hướng dẫn chi tiết về **kiến trúc của Relational Database Service (RDS)**. Video này tập trung vào kiến trúc của sản phẩm, các video tiếp theo sẽ đi sâu vào các tính năng cụ thể hơn. Chúng ta có rất nhiều nội dung cần bao quát, vì vậy hãy bắt đầu ngay.
![[Pasted image 20260716212524.png]]

Bây giờ, tôi đã nghe nhiều người gọi RDS là sản phẩm **Database as a Service (DBaaS)**. Chi tiết rất quan trọng, và bạn cần hiểu rõ tại sao đây **không phải** là trường hợp. Một sản phẩm Database as a Service là nơi bạn trả tiền và nhận được một cơ sở dữ liệu. Đây **không phải** những gì RDS làm. Với RDS, bạn trả tiền và nhận được một **database server**. Vì vậy, sẽ chính xác hơn khi gọi nó là sản phẩm **Database Server as a Service**.

Điều này rất quan trọng vì nó có nghĩa là trên **database server** hoặc instance mà RDS cung cấp, bạn có thể có **nhiều cơ sở dữ liệu**. RDS cung cấp phiên bản quản lý của database server mà bạn có thể có on-premises — chỉ với RDS, bạn không phải quản lý phần cứng, hệ điều hành, hoặc cài đặt, cũng như nhiều phần bảo trì của DB engine. Và tất nhiên, RDS chạy trong AWS.

**Các Engine Cơ sở dữ liệu được Hỗ trợ**

Với RDS, bạn có nhiều engine cơ sở dữ liệu để sử dụng, bao gồm:

- **MySQL**
- **MariaDB**
- **PostgreSQL**
- **Oracle** (Thương mại)
- **Microsoft SQL Server** (Thương mại)

Một số là mã nguồn mở và một số là thương mại, vì vậy sẽ có các hàm ý về **licensing**. Nếu phù hợp với kỳ thi bạn đang chuẩn bị, sẽ có video riêng về chủ đề này.

Bây giờ, có một thuật ngữ cụ thể mà tôi muốn bạn **tách biệt hoàn toàn** khỏi RDS, đó là **Amazon Aurora**. Bạn có thể thấy Amazon Aurora được thảo luận chung với RDS, nhưng đây thực sự là một sản phẩm khác. Amazon Aurora là một database engine và sản phẩm tùy chỉnh được tạo bởi AWS, có khả năng tương thích với một số engine ở trên, nhưng nó được thiết kế hoàn toàn bởi AWS. Nhiều tính năng tôi sẽ trình bày khi nói về RDS là khác nhau đối với Aurora, và hầu hết là những cải tiến. Vì vậy trong đầu bạn, hãy **tách Aurora khỏi RDS**.

**Tóm tắt Kiến trúc RDS**

Tóm lại, RDS là sản phẩm **managed database server as a service**. Nó cung cấp cho bạn một database instance (database server) được AWS quản lý phần lớn.

**Quy tắc chính:** Bạn **không có quyền truy cập** vào hệ điều hành hoặc truy cập SSH trên RDS.

Có một dấu hoa thị nhỏ ở đây vì có một biến thể của RDS gọi là **RDS Custom** nơi bạn có một số quyền truy cập cấp thấp hơn, nhưng tôi sẽ đề cập đến điều đó trong video khác nếu cần. Nói chung, khi nghĩ về RDS, hãy nghĩ: **không có truy cập SSH và không có quyền truy cập hệ điều hành**.

**Trực quan hóa Kiến trúc RDS**

Những gì tôi nghĩ có thể giúp bạn lúc này là xem kiến trúc RDS điển hình một cách trực quan. Sau đó, trong các video còn lại của series, tôi sẽ đi sâu hơn vào một số yếu tố của sản phẩm.

RDS là một dịch vụ chạy trong **VPC**. Vì vậy, nó **không phải** là dịch vụ công cộng như S3 hoặc DynamoDB; nó cần hoạt động trong các subnet trong VPC ở một region AWS cụ thể.

- Đối với ví dụ này, hãy sử dụng **us-east-1** làm region chính.
- Để minh họa một số phần cross-region của kiến trúc này, region phụ sẽ là **ap-southeast-2**.
- Trong us-east-1, chúng ta sẽ có một VPC. Hãy sử dụng ba **Availability Zone (AZ)** ở đây: A, B, và C.
![[Pasted image 20260716212404.png]]

Thành phần đầu tiên của RDS mà tôi muốn giới thiệu là **RDS Subnet Group**. Đây là thứ bạn tạo, và bạn có thể coi đây là danh sách các subnet mà RDS có thể sử dụng cho một database instance hoặc các instance. Vì vậy trong trường hợp này, hãy nói rằng chúng ta tạo một cái sử dụng cả ba Availability Zone. Trong thực tế, điều này có nghĩa là thêm bất kỳ subnet nào trong ba availability zone đó mà bạn muốn RDS sử dụng. Và trong ví dụ này, tôi thực sự sẽ tạo một cái khác. Chúng ta sẽ có **hai database subnet group**, và bạn sẽ thấy lý do trong giây lát. Trong database subnet group trên cùng, hãy nói rằng tôi thêm hai **public subnet**. Và trong database subnet group dưới cùng, hãy nói ba **private subnet**.

Vì vậy khi launch một RDS instance, cho dù bạn chọn có high availability hay không (tôi sẽ nói về cách hoạt động trong video sắp tới), bạn cần chọn một DB subnet group để sử dụng. Vì vậy hãy nói rằng tôi chọn database subnet group dưới cùng và launch một RDS instance, và tôi chọn một cái có high availability. Vì vậy nó sẽ chọn một subnet cho **primary instance** và một cái khác cho **standby**. Nó chọn ngẫu nhiên trừ khi bạn chỉ định ưu tiên cụ thể, nhưng nó sẽ đặt primary và standby trong các availability zone khác nhau.

Bây giờ, vì các database instance này nằm trong **private subnet**, điều đó có nghĩa là chúng sẽ có thể truy cập từ bên trong VPC hoặc từ bất kỳ mạng nào được kết nối, chẳng hạn như mạng on-premises được kết nối bằng VPN hoặc Direct Connect, hoặc bất kỳ VPC nào khác xuất hiện với cái này. Và tôi sẽ đề cập đến tất cả các chủ đề đó ở nơi khác trong khóa học, nếu tôi chưa làm như vậy.

Bây giờ, tôi cũng có thể launch một tập hợp RDS instance khác sử dụng database subnet group trên cùng, và quy trình tương tự sẽ được tuân theo giả sử rằng tôi chọn sử dụng **Multi-AZ**. RDS sẽ chọn hai subnet khác nhau trong hai availability zone khác nhau để sử dụng. Bây giờ, vì đây là public subnet, chúng ta cũng có thể, nếu thực sự muốn, chọn làm cho các instance này có thể truy cập từ internet công cộng bằng cách cấp cho chúng public addressing. Và đây là điều **thực sự bị phản đối** từ góc độ bảo mật, nhưng đây là điều bạn cần biết là một tùy chọn khi triển khai RDS instance vào public subnet.

Bây giờ, bạn có thể sử dụng một DB subnet group duy nhất cho nhiều instance, nhưng sau đó bạn bị giới hạn khi sử dụng các subnet đã xác định giống nhau. Nếu bạn muốn tách các database giữa các tập hợp subnet khác nhau, như trong ví dụ này, thì bạn cần nhiều DB subnet group. Và nói chung, như một **best practice**, tôi thích có một DB subnet group cho một triển khai RDS. Tôi thấy nó mang lại cho tôi sự linh hoạt tổng thể tốt nhất.

Được rồi, vì vậy một số khía cạnh quan trọng khác của RDS mà tôi muốn đề cập:

- Thứ nhất, **RDS instance có thể có nhiều database** trên chúng.
- Thứ hai, mọi RDS instance có **storage riêng** được cung cấp bởi **EBS**. Vì vậy nếu bạn có một cặp Multi-AZ, primary và standby, mỗi cái có storage riêng.

Bây giờ, điều này **khác** với cách Amazon Aurora xử lý storage, vì vậy hãy cố gắng nhớ kiến trúc này. Đối với RDS, mỗi instance có storage EBS riêng.

Bây giờ, nếu bạn chọn sử dụng **Multi-AZ** như trong kiến trúc này, thì primary instance sẽ replicate sang standby bằng **synchronous replication**. Điều này có nghĩa là dữ liệu được replicate sang standby ngay khi được primary nhận. Nó có nghĩa là standby sẽ có cùng tập dữ liệu với primary, vì vậy cùng các database và cùng dữ liệu trong các database đó.

Bây giờ, bạn cũng có thể quyết định có **read replicas**. Tôi sẽ đề cập những gì chúng là và cách chúng hoạt động trong video chuyên biệt khác. Nhưng tóm lại, read replicas sử dụng **asynchronous replication**, và chúng có thể ở cùng region, nhưng cũng ở các region AWS khác. Chúng có thể được sử dụng để scale read load hoặc thêm các lớp resiliency nếu bạn từng cần khôi phục ở một region AWS khác.

Bây giờ, cuối cùng, chúng ta cũng có **backups** của RDS. Có một video chuyên biệt đề cập đến backups sau trong phần này của khóa học, nhưng chỉ cần biết rằng backups xảy ra đến **S3**. Nó đến một AWS-managed S3 bucket, vì vậy bạn không thấy bucket trong tài khoản của mình, nhưng nó có nghĩa là dữ liệu được replicate qua nhiều availability zone trong region đó. Vì vậy nếu bạn có AZ failure, backups sẽ đảm bảo dữ liệu của bạn an toàn. Nếu bạn sử dụng chế độ Multi-AZ, thì backups xảy ra từ standby instance, điều đó có nghĩa là **không có tác động tiêu cực** đến hiệu suất.

Bây giờ, đây là kiến trúc sản phẩm cơ bản. Tôi sẽ mở rộng tất cả các khu vực chính này trong các video chuyên biệt, cũng như cho bạn cơ hội có trải nghiệm thực tế thông qua một số demo và mini-project, nếu phù hợp. Hiện tại, hãy đề cập một điều cuối cùng trước khi kết thúc video này, và đó là **kiến trúc chi phí** của RDS.

Vì vậy trước khi tôi kết thúc video, tôi muốn nói về chi phí RDS. Vì nó là sản phẩm database server as a service, bạn không thực sự được tính phí dựa trên usage của bạn. Thay vào đó, giống như EC2, mà RDS được dựa lỏng lẻo trên đó, bạn được tính phí cho resource allocation, và có một số thành phần khác nhau trong kiến trúc chi phí của RDS.

- Thứ nhất, bạn có **instance size và type**. Về mặt logic, instance càng lớn và càng nhiều tính năng, chi phí càng cao. Và điều này tuân theo mô hình tương tự như cách EC2 được tính phí. Phí bạn thấy là hourly rate, nhưng nó được tính **per second**.
- Tiếp theo, chúng ta có lựa chọn có sử dụng **Multi-AZ** hay không. Vì Multi-AZ có nghĩa là nhiều hơn một instance, sẽ có chi phí bổ sung. Bây giờ, chi phí nhiều hơn bao nhiêu phụ thuộc vào kiến trúc Multi-AZ, mà tôi sẽ đề cập chi tiết trong video khác.
- Tiếp theo là phí hàng tháng **per-gig** cho **storage**, có nghĩa là bạn sử dụng càng nhiều storage, chi phí càng cao. Và một số loại storage, chẳng hạn như provisioned IOPS, tốn kém hơn. Và một lần nữa, điều này được liên kết với cách EBS hoạt động, vì storage dựa trên EBS.
- Tiếp theo là chi phí **data transfer**, và đây là chi phí per gig dữ liệu transfer vào và ra khỏi DB instance từ hoặc đến internet và các region AWS khác.
- Tiếp theo, chúng ta có **backups và snapshots**. Vì vậy bạn nhận được lượng storage mà bạn trả cho database instance, trong snapshot storage miễn phí. Vì vậy nếu bạn có 2 TB storage, thì điều đó có nghĩa là 2 TB snapshots miễn phí. Ngoài ra, có chi phí. Và chi phí này là gig per month của storage. Vì vậy dữ liệu được lưu trữ càng nhiều, nó càng tốn kém; nó được lưu trữ càng lâu, nó càng tốn kém. **1 TB trong một tháng có cùng chi phí với 500 GB trong hai tháng**, vì vậy nó là chi phí per-GB, monthly.
- Và sau đó cuối cùng, chúng ta có bất kỳ chi phí bổ sung nào dựa trên việc sử dụng các loại DB engine thương mại. Và một lần nữa, tôi sẽ đề cập đến điều này, nếu phù hợp, trong video chuyên biệt ở nơi khác trong khóa học.
![[Pasted image 20260716212600.png]]

Được rồi, vì vậy tại thời điểm này, đó là mọi thứ tôi muốn đề cập trong video này. Như tôi đã đề cập ở đầu, đây chỉ là giới thiệu về kiến trúc RDS. Chúng ta sẽ đi sâu hơn vào các điểm chính cụ thể trong các video sắp tới, nhưng hiện tại, đó là mọi thứ tôi muốn đề cập. Vì vậy hãy hoàn thành video, và khi bạn sẵn sàng, tôi mong chờ bạn tham gia cùng tôi trong video tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- RDS là **Database Server as a Service** (không phải DBaaS)
- Hỗ trợ engine: **MySQL, MariaDB, PostgreSQL, Oracle, SQL Server** (có licensing cho commercial)
- **Tách biệt Amazon Aurora** khỏi RDS (Aurora là sản phẩm riêng của AWS)
- Chạy trong **VPC** với **DB Subnet Group** (public/private subnets, có thể tạo nhiều group)
- **Multi-AZ**: Primary + Standby, **synchronous replication**, dedicated EBS storage riêng biệt
- **Read Replicas**: **Asynchronous replication**, scale read load, cross-region DR
- **Backups**: Tự động đến **S3** (AWS-managed bucket), replicate multi-AZ; Multi-AZ backup từ standby → không ảnh hưởng performance
- **Quy tắc quan trọng**: Không có OS access hoặc SSH (trừ **RDS Custom**)
- **Chi phí RDS**: Instance type/size (per second), Multi-AZ, Storage (per GB/month), Data transfer, Backups/Snapshots (miễn phí đến mức allocated storage), Commercial engine license

**Notes (Chi tiết):**

- **RDS Subnet Group**: Danh sách subnet cho RDS instance. Tạo nhiều group để tách public/private subnet. Best practice: 1 group cho 1 deployment RDS.
- Khi launch **Multi-AZ**: RDS tự chọn 2 AZ khác nhau (ngẫu nhiên hoặc theo preference), primary và standby nằm ở AZ khác nhau.
- **Public subnet**: Có thể gán public IP để truy cập internet, nhưng **rất không khuyến khích** vì lý do bảo mật.
- Mỗi RDS instance có **dedicated EBS storage** (khác hoàn toàn với Aurora).
- **Replication**: Multi-AZ dùng synchronous (dữ liệu đồng bộ ngay lập tức). Read Replicas dùng asynchronous (dùng để scale đọc và phục hồi vùng khác).
- **Backups**: Tự động backup về S3 (không thấy bucket trong account). Nếu Multi-AZ thì backup từ standby → không gây giảm hiệu suất primary.
- **Chi phí chi tiết**: Instance tính theo giây (giống EC2), thêm phí Multi-AZ, storage per GB/tháng, data transfer in/out, snapshot vượt quá free tier (bằng allocated storage) tính per GB/tháng, phí license riêng cho Oracle/SQL Server.

**Summary (Tóm tắt ngắn):** Video giới thiệu **kiến trúc RDS** như một dịch vụ **managed database server** chạy trong VPC. Giải thích cách sử dụng **DB Subnet Group**, triển khai **Multi-AZ** với synchronous replication và dedicated EBS storage, **Read Replicas** với asynchronous replication (có thể cross-region), backups tự động đến S3. Nhấn mạnh **quy tắc quan trọng**: không có quyền truy cập OS/SSH (trừ RDS Custom). Kết thúc bằng phân tích **kiến trúc chi phí** dựa trên resource allocation (instance, storage, Multi-AZ, data transfer, backups). Đây là nền tảng vững chắc để hiểu sâu hơn về RDS trong các video tiếp theo.