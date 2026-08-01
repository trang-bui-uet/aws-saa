Chào mừng bạn quay trở lại. Đây là phần hai của bài học này. Chúng ta sẽ tiếp tục ngay từ điểm kết thúc của phần một, vậy nên hãy bắt đầu thôi.

Được rồi, hiện tại **cả ba mount target** đều đã ở trạng thái **available**, và điều đó có nghĩa là chúng ta có thể kết nối vào **hệ thống file EFS** này từ bất kỳ Availability Zone nào trong VPC **Animals4Life**.

Vậy việc chúng ta cần làm tiếp theo là kiểm tra quy trình này, và chúng ta sẽ tương tác với hệ thống file này từ các instance EC2. Hãy chuyển sang tab có console EC2 đang mở. Tại thời điểm này, tôi muốn bạn làm một trong hai cách sau (tùy thuộc vào trình duyệt của bạn): hoặc **nhấp chuột phải và Duplicate tab** này để mở một bản sao giống hệt; nếu trình duyệt của bạn không hỗ trợ, thì hãy mở tab mới và sao chép-dán URL vào tab đó. Bạn sẽ có hai tab riêng biệt cùng mở màn hình EC2.

Trên tab đầu tiên, chúng ta sẽ kết nối vào **A4L-EFS-InstanceA**. Nhấp chuột phải, sau đó chọn **Connect**. Chúng ta sẽ sử dụng **Instance Connect**, hãy đảm bảo username là **ec2-user**, rồi nhấn **Connect**.

Hiện tại, instance này chưa được kết nối với hệ thống file EFS, và chúng ta có thể kiểm chứng điều đó bằng cách chạy lệnh **df -k** rồi nhấn Enter. Bạn sẽ thấy rằng không có dòng nào liệt kê hệ thống file EFS này. Tất cả đều là các volume được gắn trực tiếp vào instance EC2, và tất nhiên volume boot được cung cấp bởi EBS.

Trong Linux, mọi thiết bị hoặc hệ thống file đều được mount vào một thư mục. Vì vậy, việc đầu tiên chúng ta cần làm để tương tác với EFS là tạo một thư mục để mount hệ thống file EFS vào. Chúng ta có thể làm điều đó bằng lệnh sau:

Bash

```
sudo mkdir -p /efs/wp-content
```

Tùy chọn **-p** có nghĩa là mọi phần trong đường dẫn sẽ được tạo nếu chưa tồn tại. Vậy lệnh này sẽ tạo thư mục /efs nếu nó chưa có. Hãy nhấn Enter để tạo thư mục đó.

Tôi sẽ clear màn hình để dễ nhìn hơn. Tiếp theo, chúng ta cần cài đặt một gói công cụ cho phép instance này (cụ thể là hệ điều hành) tương tác với dịch vụ EFS. Lệnh tôi sẽ dùng để cài đặt các công cụ này là **sudo** để có quyền admin, sau đó là **dnf** (trình quản lý gói của hệ điều hành này), rồi khoảng trắng, **-y** để tự động đồng ý mọi prompt, khoảng trắng, **install** vì tôi muốn cài một gói, khoảng trắng, và tên gói công cụ cần cài là **amazon-efs-utils**.

Đây là bộ công cụ cho phép hệ điều hành tương tác với EFS. Hãy nhấn Enter để cài đặt, sau đó chúng ta sẽ cấu hình sự tương tác giữa hệ điều hành này và EFS.

Một lần nữa, tôi sẽ clear màn hình để dễ nhìn. Tôi muốn mount hệ thống file EFS vào thư mục mà chúng ta vừa tạo. Nhưng cụ thể hơn, tôi muốn nó được mount mỗi khi instance khởi động lại. Vì vậy, tất nhiên chúng ta cần thêm nó vào file **fstab**.

Nếu bạn còn nhớ file này từ các phần khác trong khóa học, nó nằm ở **/etc/fstab**. Vậy chúng ta cần chạy:

Bash

```
sudo nano /etc/fstab
```

Nhấn Enter, và file này thường chỉ có một hoặc hai dòng, đó là volume root và/hoặc boot của instance. Hãy di chuyển xuống cuối vì chúng ta sẽ thêm một dòng mới. Dòng này có trong tài liệu lệnh của bài học, nhưng chúng ta sẽ dán dòng sau:

text

```
file-system-id:/ /efs/wp-content efs _netdev,tls 0 0
```

Dòng này cho biết chúng ta muốn mount **file system ID** này (tức là file-system-id:/) vào thư mục /efs/wp-content. Chúng ta chỉ định loại hệ thống file là **efs**. Hãy nhớ rằng EFS thực chất dựa trên **NFS** (Network File System), nhưng đây là dịch vụ do AWS cung cấp, vì vậy chúng ta sử dụng loại hệ thống file đặc biệt của AWS là **efs**. Và hỗ trợ cho loại này đã được cài đặt bởi gói công cụ mà chúng ta vừa cài.

Chức năng chi tiết của các tùy chọn này nằm ngoài phạm vi khóa học, nhưng nếu bạn muốn tìm hiểu thêm thì hãy tự nghiên cứu xem chính xác các option này làm gì.

Điều chúng ta cần làm lúc này là trỏ nó đến hệ thống file EFS cụ thể của chúng ta — đó chính là phần từ đầu dòng đến dấu gạch chéo.

Để lấy **File system ID**, chúng ta cần quay lại console EFS, và sao chép toàn bộ **File system ID** (của bạn sẽ khác). Hãy chắc chắn bạn sao chép đúng ID của mình vào clipboard. Sau đó quay lại đây, chọn dấu hai chấm và xóa hết từ đó về đầu dòng. Khi đã xóa xong, hãy dán **File system ID** của bạn vào. Kết quả phải là **File system ID**, dấu hai chấm, rồi dấu gạch chéo.

Tại thời điểm này, chúng ta cần lưu file. Nhấn **Ctrl + O** để lưu, rồi Enter, và **Ctrl + X** để thoát.

Một lần nữa, tôi sẽ clear màn hình để dễ nhìn. Sau đó chạy **df -k**, và đây là những hệ thống file hiện đang gắn vào instance này.

Tiếp theo chúng ta sẽ mount hệ thống file EFS vào thư mục đã tạo. Cách thực hiện là dùng lệnh:

Bash

```
sudo mount /efs/wp-content
```

Cách hoạt động của lệnh này là nó sẽ sử dụng những gì chúng ta vừa định nghĩa trong file **fstab**. Vậy chúng ta đang mount vào thư mục này bất kỳ hệ thống file nào được định nghĩa trong file đó. Đó chính là hệ thống file EFS. Hãy nhấn Enter, sau vài giây nó sẽ trả về prompt, và như vậy là đã mount xong hệ thống file.

Được rồi, chúng ta đã trở lại prompt, và nếu chạy lại **df -k**, chúng ta sẽ thấy thêm một dòng ở dưới cùng. Đây chính là hệ thống file EFS đã được mount vào thư mục này.

Để chứng minh đây thực sự là một hệ thống file mạng, hãy di chuyển vào thư mục đó bằng lệnh:

Bash

```
cd /efs/wp-content/
```

Khi đã ở trong thư mục, chúng ta sẽ tạo một file. Sử dụng **sudo** để có quyền admin, sau đó dùng lệnh **touch** (nếu bạn còn nhớ từ trước trong khóa học, lệnh này chỉ tạo một file rỗng). Chúng ta sẽ đặt tên file là **amazing-test-file.txt**. Nhấn Enter, rồi chạy **ls -la**, bạn sẽ thấy file này đã được tạo trong thư mục. Và dù chúng ta tạo nó trên instance EC2 này, thực tế file đã được ghi lên một hệ thống file mạng.

Để xác minh điều đó, hãy chuyển sang tab còn lại đang mở console EC2 (tab vẫn đang ở màn hình Running instances), và bây giờ hãy kết nối vào **InstanceB**. Nhấp chuột phải vào InstanceB, chọn **Connect**. Lại dùng **Instance Connect**, kiểm tra username đúng, rồi nhấn **Connect**.

Bây giờ chúng ta đang ở trên InstanceB. Hãy chạy **df -k** để xác nhận hiện chưa có hệ thống file EFS nào được mount.

Tiếp theo, chúng ta cần cài gói công cụ EFS để có thể mount hệ thống file này. Hãy cài đặt gói đó, clear màn hình cho dễ nhìn. Sau đó tạo thư mục sẽ dùng để mount hệ thống file (dùng đúng lệnh như trên InstanceA).

Tiếp theo cần chỉnh sửa file **fstab** để thêm cấu hình hệ thống file. Chúng ta sẽ làm bằng lệnh:

Bash

```
sudo nano /etc/fstab
```

Nhấn Enter. Nhớ rằng đây là InstanceB nên sẽ chưa có dòng mà chúng ta đã thêm trên InstanceA. Hãy xuống cuối file, dán đoạn placeholder, rồi thay thế **File system ID** ở đầu bằng ID thực tế. Xóa phần đó đi, chỉ để lại dấu hai chấm và dấu gạch chéo.

Quay lại console EFS, sao chép **File system ID** vào clipboard, quay lại instance này và dán vào.

Mọi thứ trông ổn. Lưu file bằng **Ctrl + O**, nhấn Enter, thoát bằng **Ctrl + X**. Chúng ta trở lại prompt.

Clear màn hình. Chúng ta lại dùng lệnh **sudo mount /efs/wp-content** để mount hệ thống file EFS lên instance này, và nó sẽ sử dụng cấu hình vừa định nghĩa trong file **fstab**. Nhấn Enter. Sau vài giây bạn sẽ được đưa về prompt. Chúng ta có thể kiểm tra xem đã mount thành công chưa bằng **df -k**.

Nhìn có vẻ đã mount rồi — nó nằm ở dòng dưới cùng.

Bây giờ nếu chúng ta di chuyển vào thư mục đó — **cd /efs/wp-content/** và nhấn Enter — chúng ta đang ở trong thư mục, và nếu liệt kê nội dung bằng **ls -la**, chúng ta sẽ thấy file **amazing-test-file.txt** đã được tạo trên InstanceA. Điều này chứng minh đây là một hệ thống file mạng chia sẻ, nơi bất kỳ file nào được thêm trên một instance đều có thể nhìn thấy từ tất cả các instance khác.

Vậy **EFS** là một hệ thống file **multi-user, dựa trên mạng**, có thể được mount trên cả các instance EC2 Linux lẫn các máy chủ vật lý hoặc ảo on-premises chạy Linux.

Đây chỉ là một ví dụ đơn giản về cách sử dụng EFS. Hiện tại chúng ta đã hoàn thành mọi thứ cần thiết trong bài demo này, vậy nên chúng ta chỉ cần dọn dẹp toàn bộ hạ tầng đã sử dụng. Để làm điều đó, hãy quay lại console EFS. Chúng ta sẽ xóa hệ thống file này. Nên đã chọn sẵn, hãy chọn **Delete**. Bạn sẽ cần xác nhận bằng cách dán **File system ID**. Hãy dán ID của bạn vào rồi chọn **Confirm**.

Quá trình xóa có thể mất một thời gian, bạn cần đợi nó hoàn tất.

Khi đã xóa xong, chúng ta sẽ chuyển sang console **CloudFormation**. Bạn vẫn nên giữ tab này mở. Nếu không, hãy gõ **CloudFormation** vào ô tìm kiếm phía trên rồi vào console CloudFormation.

Bạn vẫn nên thấy stack tên **implementing-efs** — đây là stack bạn đã tạo từ đầu bằng one-click deployment. Hãy chọn stack này, rồi nhấn **Delete**, và xác nhận việc xóa. Khi quá trình xóa hoàn tất, toàn bộ hạ tầng chúng ta đã tạo trong bài demo này sẽ biến mất.

Tôi hy vọng đây là một bài demo thú vị và bổ ích, giúp bạn có trải nghiệm thực tế khi làm việc với EFS. Tại thời điểm này, đó là tất cả những gì bạn cần làm trong bài demo này. Hãy hoàn thành video, và khi sẵn sàng, tôi rất mong được gặp bạn ở bài tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Mount Target Available & kết nối EFS từ nhiều AZ
- Cài đặt **amazon-efs-utils** trên EC2
- Tạo thư mục mount /efs/wp-content
- Cấu hình **/etc/fstab** với _netdev,tls
- Mount EFS & kiểm tra bằng df -k
- Chứng minh chia sẻ file giữa InstanceA và InstanceB
- Cleanup: Delete EFS File System + CloudFormation stack

**Notes (Chi tiết):**

- Sau khi **3 mount target** ở trạng thái **Available**, có thể mount EFS từ bất kỳ AZ nào trong VPC.
- Trên **InstanceA**:
    - Tạo thư mục: sudo mkdir -p /efs/wp-content
    - Cài gói: sudo dnf -y install amazon-efs-utils
    - Thêm dòng vào /etc/fstab: fs-xxxxxxxx:/ /efs/wp-content efs _netdev,tls 0 0
    - Mount: sudo mount /efs/wp-content
    - Tạo file test: touch amazing-test-file.txt
- Trên **InstanceB**: thực hiện **y hệt** các bước trên → file amazing-test-file.txt xuất hiện ngay → chứng minh EFS là **shared network file system**.
- EFS hỗ trợ mount trên cả EC2 Linux và server Linux on-premises.
- **Cleanup**:
    1. Xóa File System trên console EFS (cần dán đúng File System ID để Confirm).
    2. Xóa stack CloudFormation tên **implementing-efs**.

**Summary (Tóm tắt ngắn):** Bài học hướng dẫn cách **mount EFS** lên hai EC2 instance Linux (InstanceA & InstanceB) bằng cách cài **amazon-efs-utils**, tạo thư mục, cấu hình **/etc/fstab** và mount. Việc tạo file trên một instance và thấy ngay trên instance kia chứng minh EFS là hệ thống file mạng chia sẻ multi-AZ. Kết thúc bằng quy trình dọn dẹp hoàn toàn (xóa EFS + xóa CloudFormation stack).