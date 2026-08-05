**Chào mừng trở lại!** Trong bài học này, tôi muốn nói về **kiến trúc AWS khu vực (regional) và toàn cầu (global)**. Hãy bắt đầu ngay.

Trong suốt bài học, tôi muốn bạn nghĩ về một ứng dụng mà bạn quen thuộc và có tính **toàn cầu**. Ví dụ tôi sẽ dùng là **Netflix**, vì đây là ứng dụng mà hầu hết mọi người đều đã từng nghe đến. Netflix có thể được xem như một ứng dụng toàn cầu, nhưng thực chất nó là **tập hợp của nhiều ứng dụng khu vực nhỏ hơn**, tạo nên nền tảng Netflix toàn cầu. Đây là những khối hạ tầng độc lập, hoạt động riêng biệt và được **nhân bản (duplicated)** trên nhiều khu vực khác nhau trên thế giới.

Là **Solutions Architect**, khi thiết kế giải pháp, tôi nhận thấy có **ba loại kiến trúc chính**:

1. **Kiến trúc quy mô nhỏ** – chỉ tồn tại trong **một khu vực** hoặc **một quốc gia**.
2. **Hệ thống cũng chỉ tồn tại trong một khu vực/quốc gia**, nhưng có yêu cầu **Disaster Recovery (DR)** – nếu khu vực chính bị lỗi thì sẽ **failover** sang khu vực phụ.
3. **Hệ thống vận hành trên nhiều khu vực** và cần tiếp tục hoạt động ngay cả khi **một hoặc nhiều khu vực bị lỗi**.

Tùy theo cách bạn thiết kế hệ thống, sẽ có một số **thành phần kiến trúc quan trọng** ánh xạ trực tiếp lên các sản phẩm và dịch vụ của AWS.

### Ở cấp độ **Toàn cầu (Global)**:

- **Định vị và khám phá dịch vụ toàn cầu (Global service location and discovery)**: Khi bạn gõ netflix.com vào trình duyệt, chuyện gì xảy ra? Máy của bạn **khám phá** và trỏ đến đâu?
- **Phân phối nội dung (Content Delivery)**: Nội dung hoặc dữ liệu của ứng dụng được đưa đến người dùng toàn cầu như thế nào? Có các **kho lưu trữ phân tán** trên toàn cầu hay được kéo từ một vị trí trung tâm?
- **Kiểm tra sức khỏe toàn cầu và failover (Global health checks and failover)**: Phát hiện hạ tầng ở một vị trí có **khỏe mạnh** hay không, và chuyển khách hàng sang quốc gia/khu vực khác khi cần.

Đây là các **thành phần toàn cầu**. Tiếp theo là các **thành phần khu vực (Regional)**:

- Điểm vào khu vực (**Regional entry point**)
- Khả năng mở rộng và độ bền vững khu vực (**Regional scaling and regional resilience**)
- Các dịch vụ và thành phần ứng dụng khác nhau

Trong phần còn lại của khóa học, chúng ta sẽ xem xét các kiến trúc cụ thể. Khi đó, tôi muốn bạn luôn nghĩ về chúng dưới góc độ **thành phần toàn cầu** và **thành phần khu vực** — phần nào dùng cho **độ bền vững toàn cầu**, phần nào chỉ mang tính **địa phương**.

### Nhìn từ góc độ **Toàn cầu** (dùng ví dụ Netflix)

Giả sử có một nhóm người dùng đang chuẩn bị xem tập mới nhất của **Ozark**. Client Netflix sẽ sử dụng **DNS** để thực hiện **service discovery** ban đầu. Netflix đã cấu hình DNS để trỏ đến một hoặc nhiều endpoint dịch vụ.

Để đơn giản, giả sử Netflix có **vị trí chính** ở một region Mỹ của AWS (ví dụ us-east-1), và nếu region này lỗi thì sẽ dùng **Úc** làm region phụ. Một cấu hình hợp lệ khác là gửi khách hàng đến **vị trí gần nhất** (trong trường hợp này là Úc). Nhưng ở đây tôi giả định có **primary** và **secondary** region.

Đây chính là thành phần **DNS** của kiến trúc, và **Route 53** là giải pháp triển khai trong AWS. Nhờ tính linh hoạt, Route 53 có thể được cấu hình theo rất nhiều cách. Điểm then chốt cho kiến trúc toàn cầu này là nó có **health checks**. Nó có thể xác định region Mỹ có khỏe mạnh hay không và **điều hướng toàn bộ phiên** đến Mỹ khi bình thường, hoặc chuyển sang Úc nếu region chính gặp sự cố.

Bất kể hạ tầng nằm ở đâu, một **Content Delivery Network (CDN)** vẫn có thể được sử dụng ở cấp độ toàn cầu. CDN đảm bảo nội dung được **cache gần người dùng nhất có thể**. Các vị trí cache này nằm trên toàn cầu và chúng **kéo nội dung** từ origin khi cần.

**Tóm lại góc nhìn toàn cầu**: Chức năng của kiến trúc ở cấp này là đưa khách hàng đến đúng vị trí hạ tầng phù hợp, **cô lập** các lỗi khu vực, và chuyển phiên sang region khác khi cần. Nó cố gắng hướng khách hàng đến region gần nhất (nếu doanh nghiệp có nhiều vị trí), và cuối cùng là cải thiện caching bằng CDN như **CloudFront**.

Nếu phần này hoạt động tốt, khách hàng sẽ được điều hướng đến một region có hạ tầng của ứng dụng (giả sử là một region ở Mỹ). Lúc này lưu lượng đã **đi vào một region cụ thể** của AWS. Tùy kiến trúc, lưu lượng có thể vào **VPC** hoặc sử dụng các dịch vụ AWS công cộng. Từ đây, chúng ta phải nghĩ về kiến trúc theo **góc độ khu vực**.

Cách hiệu quả nhất để nghĩ về kiến trúc hệ thống là xem nó như **tập hợp các region** tạo nên một tổng thể. Nếu bạn nghĩ về các sản phẩm và dịch vụ AWS, **rất ít dịch vụ thực sự là toàn cầu**. Hầu hết chúng chạy trong một region, và nhiều region hợp lại tạo thành AWS. Việc nghĩ theo cách này giúp thiết kế nền tảng lớn trở nên dễ dàng hơn rất nhiều.

### Các **Tier** (tầng) trong kiến trúc ứng dụng

![[Pasted image 20260803224317.png]]

Trong phần còn lại của khóa học, chúng ta sẽ đi sâu vào kiến trúc — cách chúng hoạt động, tích hợp, và các tính năng của sản phẩm. Các môi trường bạn thiết kế thường sẽ có các **tier** khác nhau. Tier ở đây là các nhóm chức năng cấp cao hoặc các vùng khác nhau của ứng dụng:

1. **Web Tier** Giao tiếp từ khách hàng thường đi vào ở **Web Tier**. Đây thường là dịch vụ khu vực như **Application Load Balancer** hoặc **API Gateway** (tùy kiến trúc ứng dụng). Mục đích của Web Tier là làm **điểm vào** cho các ứng dụng/thành phần ứng dụng dựa trên region. Nó **tách biệt** khách hàng khỏi hạ tầng bên dưới, cho phép hạ tầng phía sau **mở rộng, lỗi hoặc thay đổi** mà không ảnh hưởng đến khách hàng.
2. **Compute Tier** Chức năng cung cấp cho khách hàng qua Web Tier được thực hiện bởi **Compute Tier**, sử dụng các dịch vụ như **EC2**, **Lambda**, hoặc containers chạy trên **Elastic Container Service (ECS)**. Trong ví dụ này, Load Balancer sẽ dùng EC2 để cung cấp dịch vụ compute cho khách hàng. Chúng ta sẽ nói chi tiết về các loại compute khác nhau và khi nào nên dùng chúng.
3. **Storage Tier** Compute Tier sẽ tiêu thụ các dịch vụ lưu trữ. Tier này sử dụng các dịch vụ như **EBS** (Elastic Block Store), **EFS** (Elastic File System), và **S3** (đặc biệt cho lưu trữ media). Nhiều kiến trúc toàn cầu sử dụng **CloudFront** (CDN toàn cầu của AWS), và CloudFront có thể dùng **S3** làm origin cho media. Netflix có thể lưu phim và chương trình TV trên S3, sau đó được CloudFront cache. Tất cả các tier này là các thành phần riêng biệt và có thể tiêu thụ dịch vụ của nhau. Ví dụ CloudFront có thể truy cập trực tiếp S3 để lấy nội dung phân phối cho khán giả toàn cầu.
4. **Database & Caching Tier** Ngoài lưu trữ file, hầu hết môi trường đều cần **lưu trữ dữ liệu**. Trong AWS, điều này được cung cấp bởi các sản phẩm như **RDS**, **Aurora**, **DynamoDB**, và **Redshift** (cho data warehousing). Tuy nhiên, để cải thiện hiệu suất, hầu hết ứng dụng **không truy cập trực tiếp** database. Thay vào đó, chúng đi qua một **lớp caching**. Các sản phẩm như **ElastiCache** (caching tổng quát) hoặc **DynamoDB Accelerator (DAX)** khi dùng DynamoDB. Cách này giúp **giảm thiểu đọc** từ database. Ứng dụng sẽ kiểm tra cache trước, chỉ khi dữ liệu không có trong cache thì mới truy vấn database và cập nhật lại cache. Caching thường nằm trong **bộ nhớ**, nên rẻ và nhanh. Database thì đắt hơn dựa trên khối lượng dữ liệu. Vì vậy, càng offload được nhiều đọc sang caching thì càng cải thiện hiệu suất và giảm chi phí.
5. **Application Services** Cuối cùng, AWS có một bộ sản phẩm chuyên cung cấp **dịch vụ ứng dụng** — như **Kinesis**, **Step Functions**, **SQS**, và **SNS**. Chúng cung cấp chức năng cho ứng dụng, từ đơn giản như email/thông báo, đến những chức năng có thể **thay đổi kiến trúc** của ứng dụng (ví dụ khi bạn **tách rời các thành phần** bằng queue).

Như tôi đã nói ở đầu bài, bạn sẽ học về tất cả các thành phần này và cách kết hợp chúng để xây dựng nền tảng. Hiện tại, hãy coi đây chỉ là bài **giới thiệu**. Tôi muốn bạn làm quen với việc nghĩ về kiến trúc từ góc độ **toàn cầu và khu vực**, đồng thời hiểu rằng kiến trúc ứng dụng thường được xây dựng bằng các thành phần từ tất cả các tier này: Web Tier, Compute Tier, Caching, Storage, Database Tier, và Application Services.

Đến đây là toàn bộ phần lý thuyết tôi muốn trình bày. Nhớ rằng đây chỉ là bài **giới thiệu**. Hãy hoàn thành bài học này, và khi sẵn sàng, tôi rất mong được gặp bạn ở bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Ba loại kiến trúc chính (Single Region, Single Region + DR, Multi-Region)
- Thành phần Global: DNS/Discovery, Content Delivery, Health Check & Failover
- Thành phần Regional: Entry Point, Scaling, Resilience, Application Services
- Route 53 + Health Checks
- CloudFront (CDN) + S3 Origin
- Các Tier: Web, Compute, Storage, Database & Caching, Application Services
- Caching (ElastiCache, DAX) để giảm tải Database

**Notes (Chi tiết):**

- **Ba loại kiến trúc**:
    1. Chỉ một region/quốc gia.
    2. Một region + **Disaster Recovery** (failover sang region phụ).
    3. Multi-region, chịu được lỗi của một hoặc nhiều region.
- **Thành phần Global**:
    - Service Discovery → **Route 53** (có health check).
    - Content Delivery → **CloudFront** (cache gần người dùng).
    - Health Check & Failover → chuyển traffic khi region chính lỗi.
- **Thành phần Regional**:
    - Entry Point → ALB hoặc API Gateway.
    - Compute → EC2, Lambda, ECS.
    - Storage → EBS, EFS, S3.
    - Database → RDS, Aurora, DynamoDB, Redshift.
    - Caching → ElastiCache, DAX (ưu tiên đọc từ cache trước).
    - Application Services → SQS, SNS, Kinesis, Step Functions.
- Tư duy quan trọng: Hầu hết dịch vụ AWS là **regional**, rất ít dịch vụ thực sự global. Nên thiết kế hệ thống như **tập hợp các region**.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu cách nghĩ về kiến trúc AWS theo hai cấp độ **Global** và **Regional**. Sử dụng ví dụ Netflix để minh họa DNS (Route 53), CDN (CloudFront), Health Check & Failover. Sau đó đi sâu vào các **Tier** chính của ứng dụng: Web, Compute, Storage, Database & Caching, Application Services. Mục tiêu là giúp học viên hình thành tư duy thiết kế hệ thống có khả năng chịu lỗi khu vực và tối ưu hiệu suất toàn cầu.