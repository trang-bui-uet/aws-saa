Chào mừng đến với bài học demo này, nơi bạn sẽ di chuyển từ **kiến trúc đơn khối (monolithic)** ở bên trái màn hình sang **kiến trúc phân tầng (tiered)** ở bên phải. Về cơ bản, bạn sẽ tách kiến trúc ứng dụng WordPress. Bạn sẽ di chuyển cơ sở dữ liệu từ việc nằm cùng máy chủ với ứng dụng sang một máy chủ khác. Và đây sẽ là **bước một** trong việc di chuyển kiến trúc này từ monolith sang kiến trúc hoàn toàn đàn hồi (elastic). Bây giờ, đây là giai đoạn đầu tiên của nhiều giai đoạn, nhưng nó là một bước cần thiết.

Để thực hiện demo này, bạn sẽ cần một số hạ tầng. Trước khi áp dụng hạ tầng, hãy đảm bảo bạn đã đăng nhập vào **tài khoản AWS chung** — tức là tài khoản quản lý của tổ chức — và như thường lệ, bạn cần chọn vùng **Northern Virginia**.

Một khi đã có cả hai, có một **liên kết triển khai một-click** đính kèm với bài học này. Hãy nhấp vào liên kết đó. Điều này sẽ triển khai hạ tầng cơ sở **Animals for Life**:

- Nó sẽ triển khai phiên bản ứng dụng WordPress đơn khối.
- Nó cũng sẽ triển khai một phiên bản cơ sở dữ liệu **MariaDB** riêng mà bạn sẽ sử dụng như một phần của quá trình di chuyển.

Bây giờ, mọi thứ đã sẵn sàng. Tên stack nên được đặt mặc định phù hợp. Tất cả những gì bạn cần làm là cuộn xuống dưới cùng, đánh dấu hộp **capabilities**, và nhấp vào **Create Stack**.

Cũng đính kèm với bài học này là **tài liệu lệnh bài học** chứa tất cả các lệnh bạn sẽ sử dụng trong suốt demo. Hãy mở nó trong tab mới. Bạn sẽ tham khảo nó liên tục khi thực hiện các điều chỉnh cho kiến trúc WordPress. Bây giờ, chúng ta sẽ cần stack CloudFormation này hoàn thành hoàn toàn trước khi tiếp tục, vậy hãy tạm dừng video và tiếp tục khi stack CloudFormation chuyển sang trạng thái **CREATE_COMPLETE**.

**Xác minh Hạ tầng**

Bây giờ stack đã chuyển sang trạng thái **CREATE_COMPLETE**, chúng ta có thể tiếp tục. Điều này đã tạo ra hạ tầng cơ sở Animals for Life, bao gồm một số phiên bản EC2. Hãy xem xét chúng.

1. Nhấp vào **Services** và sau đó tìm và mở **EC2** trong tab mới.
2. Một khi ở bảng điều khiển EC2, nếu thấy bất kỳ hộp thoại nào về cập nhật giao diện người dùng, hãy đóng chúng lại, sau đó nhấp vào **Instances (running)**.

Ở đây, bạn sẽ thấy hai phiên bản EC2:

- Một sẽ được gọi là **A4L-WORDPRESS**, và đây là **monolith**. Đây là phiên bản EC2 chứa ứng dụng WordPress và cơ sở dữ liệu tích hợp. Đây là cài đặt WordPress mà chúng ta sẽ di chuyển từ đó.
- Và phiên bản **A4L-DB-WORDPRESS**. Phiên bản này chứa cài đặt **MariaDB** độc lập. Chúng ta sẽ di chuyển cơ sở dữ liệu cho WordPress từ phiên bản monolith sang phiên bản DB này. Điều này sẽ tạo ra kiến trúc ứng dụng phân tầng thay vì monolith hiện tại.

**Thiết lập WordPress và Tạo Dữ liệu Ban đầu**

Bước số một là thực hiện cài đặt WordPress. Để làm điều đó:

1. Sao chép **địa chỉ IPv4 công khai** của phiên bản EC2 WordPress vào clipboard.
2. Mở nó trong tab mới. (Lưu ý: Cẩn thận không sử dụng liên kết “open address”, vì nó sẽ sử dụng HTTPS, mà chúng ta hiện không sử dụng.)
3. Một khi dán địa chỉ IP và nhấn Enter, bạn sẽ thấy hộp thoại cài đặt WordPress quen thuộc.

Chúng ta sẽ tạo một blog đơn giản:

- **Site Title**: The Best Cats
- **Username**: admin
- **Password**: Thay vì sử dụng mật khẩu ngẫu nhiên, hãy sử dụng cùng mật khẩu phức tạp mà chúng ta đã dùng cho các template CloudFormation: **AnimalsForLife** (với thay thế số). Nếu bạn quay lại tab CloudFormation và đến tab **Parameters**, đây là cùng mật khẩu mà chúng ta đã dùng cho DB password và DB root password. (Tất nhiên, trong production, đây là thực hành cực kỳ tệ. Chúng ta chỉ làm điều này trong demo để giữ mọi thứ đơn giản và tránh sai lầm.)
- **Email**: Chỉ cần gõ một email giả. Chúng ta không muốn bạn sử dụng email thật của mình cho việc này; tôi sẽ gõ test@test.com, và bạn có thể làm tương tự.
- Nhấp vào **Install WordPress**.

Điều này đã cài đặt ứng dụng WordPress, và hiện nó đang sử dụng máy chủ MariaDB nằm trên cùng phiên bản EC2 — tức là một phần của monolith.

Bây giờ, chúng ta sẽ đăng nhập. Gõ **admin** và sử dụng mật khẩu mạnh **AnimalsForLife**, sau đó nhấp vào **Log In**. Một khi đã đăng nhập, chúng ta sẽ tạo một bài đăng blog đơn giản để tạo dữ liệu cơ sở dữ liệu và hệ thống tập tin:

1. Nhấp vào **Posts**.
2. Chọn bài đăng “Hello world” hiện có và nhấp **Trash**.
3. Nhấp vào **Add New**.
4. Đóng hộp thoại giới thiệu.
5. Đối với tiêu đề, gõ **The Best Cats Ever!!!**
6. Tiếp theo, nhấp vào dấu **+** (plus) và thêm một **Gallery**.

Tại thời điểm này, bạn sẽ cần một số hình ảnh để tải lên. Tôi đã đính kèm **liên kết hình ảnh** với bài học này. Nếu bạn nhấp vào liên kết đó, nó sẽ tải xuống một tệp zip. Nếu giải nén tệp zip đó, nó chứa bốn tệp hình ảnh — tất cả bốn con mèo của tôi!

Một khi đã tải xuống và giải nén tệp đó, nhấp vào **Upload**, tìm các hình ảnh đó (chọn tất cả bốn), và nhấp vào **Open**. Điều đó sẽ thêm các hình ảnh này vào bài đăng blog. Một khi chúng được tải lên, hãy nhấp vào **Publish**, và sau đó **Publish** lần nữa.

Việc xuất bản bài đăng blog này làm hai việc:

1. Nó thêm dữ liệu vào cơ sở dữ liệu cục bộ đang chạy trên phiên bản ứng dụng đơn khối.
2. Nó lưu trữ các tệp hình ảnh đã tải lên trên hệ thống tập tin của phiên bản cục bộ.

Tôi đang nhấn mạnh rằng các hình ảnh này được lưu trữ trên hệ thống tập tin cục bộ vì, như bạn sẽ thấy sau trong khóa học, đây là một trong những thứ chúng ta cần di chuyển khi chuyển sang kiến trúc hoàn toàn đàn hồi. Chúng ta không thể có hình ảnh được lưu trữ trên chính các phiên bản; chúng ta sẽ cần di chuyển chúng sang hệ thống tập tin chia sẻ. Tuy nhiên, hiện tại, chúng ta đang tập trung hoàn toàn vào cơ sở dữ liệu.

Tại thời điểm này, chúng ta có blog đang hoạt động. Các hình ảnh được lưu trữ trên hệ thống tập tin cục bộ của **A4L-WORDPRESS**, và dữ liệu cho bài đăng blog đó được lưu trữ trên cơ sở dữ liệu MariaDB cũng đang chạy trên phiên bản EC2 này.

**Di chuyển Cơ sở dữ liệu**

Bước tiếp theo là di chuyển cơ sở dữ liệu từ **A4L-WORDPRESS** sang **A4L-DB-WORDPRESS** (là phiên bản MariaDB độc lập, chuyên dụng của chúng ta).

**Bước 1: Kết nối đến Phiên bản Nguồn**

1. Đi đến bảng điều khiển EC2 và chọn **A4L-WORDPRESS**.
2. Nhấp chuột phải và chọn **Connect**.
3. Chúng ta sẽ sử dụng **EC2 Instance Connect**, vậy đảm bảo tên người dùng được đặt thành **ec2-user** và nhấp **Connect**.

Điều này mở ra một terminal. Đây là nơi bạn sẽ sử dụng các lệnh được lưu trữ trong tài liệu lệnh bài học. Đảm bảo bạn có nó sẵn sàng để sao chép và dán để tránh lỗi đánh máy.

**Bước 2: Sao lưu Cơ sở dữ liệu Nguồn**

Đầu tiên, chúng ta cần thực hiện sao lưu cơ sở dữ liệu đang chạy trên phiên bản monolith này và lưu trữ nó dưới dạng tệp .sql trên ổ đĩa cục bộ. Chúng ta sẽ sử dụng tiện ích **mysqldump**:

Bash

```
mysqldump -u root -p A4LWordpress > A4LWordpress.sql
```

Hãy phân tích lệnh này:

- **mysqldump** là tiện ích sao lưu.
- **-u** chỉ định người dùng chúng ta đang sử dụng để kết nối (trong trường hợp này là root).
- **-p** chỉ định rằng chúng ta muốn được nhắc nhập mật khẩu.
- **A4LWordpress** là tên cơ sở dữ liệu chúng ta muốn sao lưu.
- Ký hiệu **>** chuyển hướng đầu ra (thường sẽ in ra màn hình) vào một tệp có tên **A4LWordpress.sql**.

Chạy lệnh này. Nó sẽ nhắc bạn nhập mật khẩu cơ sở dữ liệu. Quay lại tab **Parameters** của CloudFormation, sao chép **DBPassword**, dán vào terminal, và nhấn Enter.

Bạn sẽ không thấy thông báo thành công, nhưng nếu bạn chạy ls -la và nhấn Enter, bạn sẽ thấy **A4LWordpress.sql** được liệt kê trong thư mục. Bây giờ chúng ta đã có bản sao lưu cơ sở dữ liệu!

**Bước 3: Khôi phục Bản sao lưu vào Phiên bản Cơ sở dữ liệu Mới**

Bây giờ, chúng ta cần lấy tệp sao lưu .sql này và đưa nó vào phiên bản cơ sở dữ liệu chuyên dụng mới của chúng ta. Chúng ta sẽ sử dụng cấu trúc lệnh sau:

Bash

```
mysql -u A4LWordpress -h <PRIVATE_IP_OF_DB_INSTANCE> -p A4LWordpress < A4LWordpress.sql
```

Hãy phân tích lệnh này đang làm gì:

- **mysql** kết nối đến phiên bản cơ sở dữ liệu MariaDB mục tiêu.
- **-u A4LWordpress** kết nối bằng một người dùng cơ sở dữ liệu mà chúng ta đã thiết lập sẵn cho bạn có tên **A4LWordpress**.
- **-p** nhắc chúng ta nhập mật khẩu cơ sở dữ liệu.
- **A4LWordpress** là tên cơ sở dữ liệu mục tiêu.
- Ký hiệu **<** lấy nội dung của tệp sao lưu **A4LWordpress.sql** của chúng ta và đưa nó vào lệnh để thực thi trên mục tiêu.
- **-h** chỉ định máy chủ từ xa mà chúng ta muốn kết nối đến. (Khi thực hiện sao lưu, chúng ta đã bỏ qua **-h**, mặc định là máy cục bộ (localhost). Bây giờ chúng ta phải nhắm đến phiên bản EC2 cơ sở dữ liệu từ xa.)

Để lấy địa chỉ IP riêng cho cơ sở dữ liệu mục tiêu:

1. Quay lại bảng điều khiển EC2 của bạn.
2. Chọn phiên bản **A4L-DB-WORDPRESS**.
3. Tìm và sao chép **Private IPv4 address** của nó vào clipboard.
4. Quay lại cửa sổ terminal của bạn.
5. Dán địa chỉ IP riêng đó vào vị trí <PRIVATE_IP_OF_DB_INSTANCE> trong lệnh của bạn.

Lệnh cuối cùng của bạn sẽ trông giống như thế này:

Bash

```
mysql -u A4LWordpress -h 10.0.1.123 -p A4LWordpress < A4LWordpress.sql
```

(Lưu ý: Địa chỉ IP riêng thực tế của bạn sẽ khác với ví dụ trên. Luôn sao chép IP cụ thể của bạn từ bảng điều khiển AWS.)

Nhấn Enter. Bạn sẽ được nhắc nhập mật khẩu. Sử dụng cùng **DBPassword** từ parameters CloudFormation của bạn. Dán nó và nhấn Enter. Dữ liệu sao lưu hiện đã được nhập thành công vào máy chủ cơ sở dữ liệu chuyên dụng mới của bạn!

**Cấu hình lại WordPress để Sử dụng Cơ sở dữ liệu Mới**

Hiện tại, WordPress vẫn đang nhìn vào cơ sở dữ liệu cục bộ trên phiên bản ứng dụng. Chúng ta cần cấu hình lại nó để trỏ đến máy chủ cơ sở dữ liệu mới.

1. Trong terminal của bạn, điều hướng đến thư mục WordPress:
    
    Bash
    
    ```
    cd /var/www/html
    ```
    
2. Mở tệp cấu hình WordPress bằng trình soạn thảo văn bản **nano** với quyền quản trị:
    
    Bash
    
    ```
    sudo nano wp-config.php
    ```
    
3. Cuộn xuống trong tệp này cho đến khi bạn tìm thấy dòng định nghĩa máy chủ cơ sở dữ liệu:
    
    PHP
    
    ```
    define( 'DB_HOST', 'localhost' );
    ```
    
4. Chúng ta muốn thay đổi **'localhost'** thành địa chỉ IP riêng của phiên bản cơ sở dữ liệu chuyên dụng mới của chúng ta. Xóa localhost và dán địa chỉ IP riêng của phiên bản cơ sở dữ liệu vào trong dấu ngoặc đơn. Nó sẽ trông như thế này:
    
    PHP
    
    ```
    define( 'DB_HOST', '10.0.1.123' );
    ```
    
    (Một lần nữa, đảm bảo bạn sử dụng IP riêng cụ thể của phiên bản cơ sở dữ liệu của bạn.)
    
5. Lưu tệp bằng cách nhấn **Ctrl + O** và sau đó Enter.
    
6. Thoát trình soạn thảo bằng cách nhấn **Ctrl + X**.
    

**Kiểm tra Cấu hình Phân tầng Mới**

WordPress hiện đã được cấu hình để giao tiếp với cơ sở dữ liệu từ xa. Hãy xác minh rằng mọi thứ hoạt động.

**Xác minh Kết nối**

Quay lại tab trình duyệt đang chạy ứng dụng WordPress và làm mới trang. Nếu blog tải thành công với bài đăng và hình ảnh của bạn còn nguyên, điều đó có nghĩa là WordPress đang đọc dữ liệu thành công từ phiên bản cơ sở dữ liệu mới!

**Bài kiểm tra Cuối cùng: Dừng Cơ sở dữ liệu Cục bộ**

Để chắc chắn tuyệt đối rằng WordPress không còn sử dụng cơ sở dữ liệu cục bộ trên **A4L-WORDPRESS**, hãy dừng dịch vụ cơ sở dữ liệu đang chạy cục bộ trên máy chủ ứng dụng.

Trong terminal được kết nối đến **A4L-WORDPRESS**, chạy lệnh sau để dừng MariaDB:

Bash

```
sudo service mariadb stop
```

Điều này tắt công cụ cơ sở dữ liệu cục bộ. Bây giờ, công cụ cơ sở dữ liệu duy nhất đang chạy là trên phiên bản **A4L-DB-WORDPRESS** từ xa.

Quay lại tab blog WordPress của bạn và nhấn refresh. Vì nó tải thành công, điều này xác nhận không còn nghi ngờ gì nữa rằng WordPress đã được di chuyển hoàn toàn và đang giao tiếp trực tiếp với phiên bản cơ sở dữ liệu chuyên dụng!

**Tóm tắt và Dọn dẹp**

Bằng cách tách cơ sở dữ liệu khỏi máy chủ ứng dụng, chúng ta đã phá vỡ kiến trúc đơn khối của mình.

Như đã đề cập trước đó, việc chạy cơ sở dữ liệu tự quản lý trên phiên bản EC2 hiếm khi được khuyến nghị cho production vì chi phí quản trị. Trong hầu hết mọi tình huống, tốt hơn là sử dụng dịch vụ được quản lý như **AWS RDS** (Relational Database Service). Tuy nhiên, hiểu quy trình di chuyển thủ công này là rất quan trọng để hiểu cách kết nối cơ sở dữ liệu hoạt động trước khi di chuyển sang các giải pháp được quản lý. Trong bài học tiếp theo, chúng ta sẽ phát triển kiến trúc này hơn nữa bằng cách di chuyển cơ sở dữ liệu tự quản lý này sang phiên bản RDS.

**Phá hủy Hạ tầng**

Để tránh phát sinh bất kỳ chi phí AWS không cần thiết nào, hãy dọn dẹp các tài nguyên được tạo cho bài học này:

1. Đi đến bảng điều khiển **CloudFormation**.
2. Chọn stack mà chúng ta đã tạo ở đầu bài học này (ví dụ: stack EC2 đơn khối).
3. Nhấp vào **Delete** và xác nhận việc xóa.

Điều này sẽ dọn dẹp tất cả các phiên bản EC2 và cấu hình mạng, trả tài khoản AWS của bạn về trạng thái ban đầu.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Di chuyển từ **kiến trúc monolith** sang **tiered**: Tách DB sang server riêng biệt
- Triển khai hạ tầng CloudFormation: **A4L-WORDPRESS** (monolith) + **A4L-DB-WORDPRESS** (MariaDB chuyên dụng)
- Setup WordPress + tạo dữ liệu ban đầu (blog post + hình ảnh)
- **Backup DB**: mysqldump -u root -p A4LWordpress > A4LWordpress.sql
- **Restore DB**: mysql -u A4LWordpress -h <private_ip> -p A4LWordpress < A4LWordpress.sql
- **Reconfig wp-config.php**: Thay đổi DB_HOST từ 'localhost' sang Private IP của DB instance
- **Test**: Stop local mariadb service → xác nhận site vẫn hoạt động
- **Cleanup**: Delete CloudFormation stack để trả về trạng thái ban đầu

**Notes (Chi tiết):**

- **Kiến trúc tiered** là bước đầu tiên để đạt được kiến trúc đàn hồi (elastic) đầy đủ sau này.
- Sử dụng **EC2 Instance Connect** để truy cập terminal và thực hiện backup/restore DB.
- **Private IP** của DB instance được dùng để kết nối từ WordPress app (cùng VPC, không dùng Public IP).
- Thay đổi define('DB_HOST', ...) trong **wp-config.php** là bước then chốt để WordPress trỏ sang DB từ xa.
- Dừng dịch vụ sudo service mariadb stop trên instance monolith để chứng minh dữ liệu hoàn toàn đến từ DB instance mới.
- Trong production nên dùng **AWS RDS** thay vì tự quản lý MariaDB trên EC2 để giảm overhead quản trị.
- Hình ảnh vẫn nằm trên filesystem cục bộ của EC2 — đây là vấn đề cần giải quyết khi chuyển sang kiến trúc elastic đầy đủ (sẽ học sau).

**Summary (Tóm tắt ngắn):** Bài học hướng dẫn chi tiết quy trình **migrate database** của WordPress từ kiến trúc **monolithic** (cùng server) sang **tiered** (DB server riêng biệt). Các bước chính: triển khai hạ tầng qua CloudFormation → setup WP và tạo dữ liệu → backup bằng mysqldump → restore sang DB instance mới bằng mysql -h <private_ip> → reconfigure wp-config.php → test bằng cách stop local DB → dọn dẹp tài nguyên bằng cách xóa stack CloudFormation. Đây là nền tảng quan trọng để hiểu cách kết nối database hoạt động trước khi chuyển sang managed service như AWS RDS.