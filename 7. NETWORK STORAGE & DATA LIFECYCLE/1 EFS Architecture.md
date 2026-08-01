**Chào mừng trở lại.** Trong bài học này, tôi sẽ giới thiệu một sản phẩm cực kỳ hữu ích trong AWS: **Elastic File System**, hay còn gọi là **EFS**. Đây là sản phẩm có thể mang lại lợi ích cho hầu hết các dự án AWS vì nó cung cấp **hệ thống tệp dựa trên mạng**, có thể được **mount** vào các instance **Linux EC2** và được sử dụng bởi **nhiều instance cùng lúc**.

Với ví dụ **Animals for Life WordPress** mà chúng ta đã sử dụng xuyên suốt khóa học đến nay, EFS sẽ cho phép chúng ta **lưu trữ media của các bài post bên ngoài các EC2 instance riêng lẻ**. Điều này có nghĩa là media sẽ **không bị mất** khi instance được thêm vào hoặc gỡ bỏ. Và điều đó mang lại lợi ích đáng kể về **khả năng scale** cũng như **kiến trúc tự phục hồi (self-healing)**. Tóm lại, chúng ta đang đưa các EC2 instance tiến gần hơn đến trạng thái **stateless**.

Hãy cùng bước vào và xem xét **kiến trúc của EFS**.

Dịch vụ **EFS** là cách triển khai của AWS đối với một tiêu chuẩn lưu trữ chia sẻ khá phổ biến gọi là **NFS** (Network File System) — cụ thể là **phiên bản 4** của Network File System. Với EFS, bạn tạo **file system**, đây là thực thể cơ bản của sản phẩm, và các file system này có thể được **mount** vào các instance **Linux EC2**.

Linux sử dụng cấu trúc **cây (tree structure)** cho hệ thống tệp. Các thiết bị có thể được mount vào các thư mục trong hệ thống phân cấp đó, và một file system EFS, ví dụ, có thể được mount vào thư mục tên là **/nfs/media**.

Điều ấn tượng hơn nữa là các file system EFS có thể được mount trên **nhiều EC2 instance**, do đó dữ liệu trên các file system đó có thể được **chia sẻ giữa rất nhiều EC2 instance**. Hãy ghi nhớ điều này khi chúng ta nói về việc phát triển kiến trúc của nền tảng WordPress Animals for Life. Nhớ rằng, nó có hạn chế là media của các bài post — hình ảnh, video, âm thanh — đều được lưu trữ trên chính instance local. Nếu instance bị mất, media cũng bị mất theo.

**Lưu trữ EFS tồn tại độc lập với EC2 instance**, giống như EBS tồn tại độc lập với EC2. EBS là **block storage**, trong khi EFS là **file storage**, nhưng các instance Linux có thể mount file system EFS như thể chúng được kết nối trực tiếp vào instance.

**EFS là dịch vụ private**. Mặc định, nó bị cô lập trong VPC mà nó được provision vào. Về mặt kiến trúc, việc truy cập vào file system EFS được thực hiện thông qua **mount target**, là những thành phần nằm bên trong VPC. Nhưng chúng ta sẽ nói chi tiết hơn ở phần tiếp theo khi xem kiến trúc dưới dạng hình ảnh.

Mặc dù EFS là dịch vụ private, bạn vẫn có thể truy cập file system EFS thông qua các phương thức **hybrid networking** mà chúng ta chưa đề cập đến. Vì vậy, nếu VPC của bạn được kết nối với các mạng khác, thì EFS có thể được truy cập qua các kết nối đó — sử dụng **VPC peering**, **VPN**, hoặc **AWS Direct Connect** (là kết nối mạng vật lý riêng tư giữa VPC và mạng on-premises hiện có của bạn). Đừng lo lắng về các sản phẩm hybrid này; tôi sẽ đề cập chi tiết tất cả chúng ở phần sau của khóa học. Hiện tại, chỉ cần hiểu rằng EFS có thể được truy cập từ bên ngoài VPC bằng các sản phẩm hybrid networking này miễn là bạn cấu hình quyền truy cập.

Bây giờ hãy nhìn vào kiến trúc của EFS dưới dạng hình ảnh.

![[Pasted image 20260731224748.png]]

Về mặt kiến trúc, nó trông như thế này. EFS chạy bên trong một VPC — trong trường hợp này là VPC của Animals for Life. Bên trong EFS, bạn tạo các **file system**, và chúng sử dụng **POSIX permissions**. Nếu bạn không biết POSIX là gì, tôi đã đính kèm một liên kết trong bài học cung cấp thêm thông tin. Tóm tắt siêu ngắn: đây là một tiêu chuẩn về khả năng tương tác được sử dụng trong Linux. Vì vậy, một hệ thống tệp có POSIX permissions là thứ mà **tất cả các bản phân phối Linux** đều hiểu.

File system EFS được làm cho khả dụng bên trong VPC thông qua **mount target**, và các mount target này chạy từ các **subnet** bên trong VPC. Mount target có địa chỉ IP được lấy từ dải địa chỉ IP của subnet mà chúng nằm trong đó, và để đảm bảo **high availability**, bạn cần đảm bảo đặt mount target ở **nhiều Availability Zone**. Giống như NAT Gateway, để có hệ thống hoàn toàn highly available, bạn cần có một mount target trong **mọi Availability Zone** mà VPC sử dụng.

Chính các **mount target** này là thứ mà các instance sử dụng để kết nối đến file system EFS. Ngoài ra, như tôi đã đề cập ở màn hình trước, bạn có thể có một mạng **on-premises**, và mạng này thường được kết nối với VPC bằng các sản phẩm hybrid networking như VPN hoặc Direct Connect, và bất kỳ máy chủ Linux nào đang chạy trên môi trường on-premises đều có thể sử dụng hybrid networking này để kết nối đến cùng các mount target và truy cập file system EFS.

Trước khi chuyển sang phần demo nơi bạn sẽ có trải nghiệm thực tế tạo file system và truy cập nó từ nhiều EC2 instance, có một số điều về EFS mà bạn cần biết cho kỳ thi.

**Thứ nhất**, EFS chỉ dành cho **instance Linux**. Từ góc độ chính thức của AWS, nó chỉ được hỗ trợ chính thức khi sử dụng instance Linux.

EFS cung cấp **hai performance mode**:

- **General Purpose**
- **Max I/O**

**General Purpose** lý tưởng cho các trường hợp sử dụng nhạy cảm với độ trễ: web server, hệ thống quản lý nội dung (CMS), có thể dùng cho home directory, hoặc thậm chí phục vụ tệp nói chung miễn là bạn đang dùng instance Linux. **General Purpose là mặc định**, và đó là thứ chúng ta sẽ sử dụng trong phần này của khóa học trong các demo.

**Max I/O** có thể scale lên mức **aggregate throughput** và số lượng operations per second cao hơn, nhưng nó có đánh đổi là **độ trễ tăng lên**. Vì vậy, chế độ Max I/O phù hợp với các ứng dụng **có độ song song cao**. Nếu bạn có bất kỳ ứng dụng hoặc workload chung nào như big data, xử lý media, phân tích khoa học — bất cứ thứ gì có độ song song cao — thì nó có thể hưởng lợi từ Max I/O. Nhưng với hầu hết các use case, hãy chọn **General Purpose**.

Ngoài ra còn có **hai throughput mode**:

- **Bursting**
- **Provisioned**

**Bursting mode** hoạt động giống như volume **GP2** trong EBS, nghĩa là nó có **burst pool**, nhưng throughput của loại này **scale theo kích thước của file system**. Càng lưu trữ nhiều dữ liệu trong file system, hiệu năng bạn nhận được càng tốt.

Với **Provisioned**, bạn có thể chỉ định yêu cầu throughput **độc lập với kích thước**. Điều này giống như so sánh giữa GP2 và IO1. Với Provisioned, bạn có thể chỉ định throughput riêng biệt với lượng dữ liệu lưu trữ, vì vậy linh hoạt hơn, nhưng đây **không phải** là tùy chọn mặc định. Nói chung, bạn nên chọn **Bursting**.

Đối với kỳ thi, bạn không cần nhớ các con số thô, nhưng tôi đã liên kết một số thông tin trong phần mô tả bài học nếu bạn muốn tìm hiểu thêm về các đặc tính hiệu năng của các tùy chọn khác nhau.

**Amazon EFS file system có hai storage class**:

- **Infrequent Access (IA)**: Đây là storage class chi phí thấp hơn, được thiết kế để lưu trữ những thứ được truy cập không thường xuyên. Nếu bạn cần lưu trữ dữ liệu một cách tiết kiệm chi phí nhưng không định truy cập thường xuyên, bạn có thể dùng Infrequent Access.
- **Standard**: Storage class này dùng để lưu trữ các tệp được truy cập thường xuyên. Nó cũng là **mặc định**, và bạn nên coi đây là lựa chọn mặc định khi chọn giữa các storage class.

Về mặt khái niệm, chúng phản ánh các đánh đổi của các storage class object trong S3: dùng **Standard** cho dữ liệu được sử dụng hàng ngày, và **Infrequent Access** cho bất cứ thứ gì không được sử dụng một cách nhất quán. Và giống như S3, bạn có khả năng sử dụng **Lifecycle Policy** để di chuyển dữ liệu giữa các class.

Được rồi, đó là toàn bộ lý thuyết về EFS. Đây không phải là sản phẩm quá khó hiểu, nhưng bạn cần hiểu nó về mặt kiến trúc cho kỳ thi. Và để hỗ trợ điều đó, bây giờ là lúc cho phần demo. Tôi muốn bạn thực sự hiểu cách EFS hoạt động. Đây là thứ mà bạn có lẽ sẽ sử dụng nếu làm việc với AWS trong các dự án thực tế. Cách tốt nhất để hiểu là sử dụng nó, và đó là những gì chúng ta sẽ làm trong bài học tiếp theo — một demo. Bạn sẽ có cơ hội tạo một file system EFS, provision một số EC2 instance, sau đó mount file system đó vào cả hai EC2 instance, tạo một tệp test, và thấy rằng tệp đó có thể truy cập được từ cả hai instance, chứng minh rằng EFS là một **shared network file system**.

Tại thời điểm này, đó là toàn bộ lý thuyết mà tôi muốn đề cập. Hãy hoàn thành video này, và khi bạn sẵn sàng, tôi mong được gặp bạn trong bài học demo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- EFS là gì & lợi ích với WordPress (stateless)
- NFS v4 & khả năng mount nhiều instance
- Kiến trúc: File System + Mount Target + Multi-AZ
- Performance Mode: General Purpose vs Max I/O
- Throughput Mode: Bursting vs Provisioned
- Storage Class: Standard vs Infrequent Access (IA)
- Chỉ hỗ trợ Linux + Hybrid networking access

**Notes (Chi tiết):**

- **EFS** = Elastic File System, triển khai **NFS v4** của AWS, cung cấp **file storage** chia sẻ qua mạng.
- Cho phép nhiều **EC2 Linux instance** mount cùng một file system → dữ liệu media (ảnh, video…) không bị mất khi instance bị terminate → hỗ trợ **scale** và **self-healing**.
- EFS tồn tại **độc lập** với EC2 (giống EBS nhưng là file storage thay vì block storage).
- Truy cập thông qua **Mount Target** đặt trong subnet của VPC. Để **High Availability** cần mount target ở **mọi AZ**.
- Có thể truy cập từ on-premises qua **VPN / Direct Connect / VPC Peering**.
- **Performance Mode**:
    - **General Purpose** (mặc định): phù hợp web server, CMS, latency-sensitive.
    - **Max I/O**: throughput cao hơn nhưng latency tăng, phù hợp workload parallel (big data, media processing).
- **Throughput Mode**:
    - **Bursting** (mặc định): scale theo kích thước file system (giống GP2).
    - **Provisioned**: chỉ định throughput độc lập với dung lượng (linh hoạt hơn).
- **Storage Class**:
    - **Standard**: dữ liệu truy cập thường xuyên (mặc định).
    - **Infrequent Access (IA)**: chi phí thấp hơn cho dữ liệu ít truy cập.
    - Có thể dùng **Lifecycle Policy** để chuyển giữa các class.
- Chỉ chính thức hỗ trợ **Linux instance**.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu **Amazon EFS** — dịch vụ **file storage chia sẻ** dựa trên NFS v4, giúp các EC2 Linux instance lưu trữ dữ liệu chung (ví dụ media WordPress) bên ngoài instance, hướng tới kiến trúc **stateless**. EFS sử dụng **Mount Target** trong VPC (cần multi-AZ để HA), hỗ trợ hai performance mode (**General Purpose** mặc định & **Max I/O**), hai throughput mode (**Bursting** mặc định & **Provisioned**), và hai storage class (**Standard** & **IA**). Phần tiếp theo sẽ là demo thực hành tạo và mount EFS.