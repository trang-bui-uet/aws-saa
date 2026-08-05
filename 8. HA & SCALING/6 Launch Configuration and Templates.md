Chào mừng bạn quay lại. Trong bài này, chúng ta sẽ tìm hiểu hai tính năng của EC2: Launch Configurations và Launch Templates. Cả hai làm việc tương tự, nhưng Launch Templates ra đời sau Launch Configurations và có thêm nhiều tính năng/khả năng.

Bài này sẽ khá ngắn vì hai khái niệm này tương đối dễ hiểu. Bài tiếp theo sẽ nói về Auto Scaling Groups, vốn sử dụng Launch Configuration hoặc Launch Template. Vì vậy bài này sẽ giữ trọng tâm rõ ràng.

## Điểm chung ở mức cao

Launch Configurations và Launch Templates về cơ bản làm cùng một việc: cho phép định nghĩa trước cấu hình EC2 instance.

Chúng là các “document” giúp bạn cấu hình sẵn các thứ như:

- AMI sẽ dùng
- Instance type và size
- Cấu hình storage của instance
- Key pair dùng để kết nối instance
- Cấu hình networking và Security Groups
- User Data cung cấp cho instance
- IAM Role gắn vào instance (để cấp quyền)

→ Mọi thứ bạn thường định nghĩa lúc launch một instance đều có thể định nghĩa trong Launch Configurations và Launch Templates.

## Không chỉnh sửa được sau khi tạo

Cả hai đều không editable:

- Bạn định nghĩa một lần → cấu hình bị khóa (locked)
- Launch Templates (mới hơn) hỗ trợ versions
- Launch Configurations thì không có versioning

## Launch Templates có thêm gì?

Launch Templates còn cho phép kiểm soát các tính năng của instance thế hệ mới hơn, ví dụ:

- Tùy chọn CPU T2/T3 Unlimited
- Placement Groups
- Capacity Reservations
- Elastic Graphics

AWS khuyến nghị dùng Launch Templates ở thời điểm hiện tại vì chúng là superset của Launch Configurations: có đủ mọi thứ Launch Configurations có, và nhiều hơn.

## Khác biệt về cách dùng trong kiến trúc

### Launch Configurations

- Chỉ có một mục đích: dùng trong Auto Scaling Groups
- Auto Scaling Groups cung cấp khả năng scale tự động cho EC2
- Launch Configuration cung cấp cấu hình cho các EC2 mà ASG sẽ launch
- Không editable, không versioning
- Muốn chỉnh cấu hình → phải tạo Launch Configuration mới rồi dùng cái mới

### Launch Templates

- Cũng dùng được với Auto Scaling Groups (cung cấp cấu hình EC2)
- Ngoài ra còn dùng để launch EC2 trực tiếp từ Console hoặc CLI
- Ví dụ: Bob có thể định nghĩa cấu hình instance trước, rồi dùng lại mỗi lần launch EC2

Bạn sẽ được tạo và dùng Launch Templates trong các bài demo sau trong phần này. Hiện tại chỉ cần nắm lý thuyết để thấy chúng khớp với nhau thế nào.

---

Đó là toàn bộ nội dung về Launch Configurations và Launch Templates trong bài này. Bài tiếp theo sẽ nói về Auto Scaling Groups — hai thứ này hoạt động cùng nhau để EC2 scale theo tải của hệ thống.

Hãy hoàn thành video này, rồi khi sẵn sàng hãy tiếp tục bài tiếp theo.

---

Tóm tắt theo Cornell Note

Cues (Từ khóa / Ý chính):

- Launch Config / Launch Template = định nghĩa trước cấu hình EC2
- Không editable sau khi tạo
- Launch Template có versioning; Launch Config thì không
- Launch Template = superset (AWS khuyến nghị)
- Launch Config chỉ dùng cho Auto Scaling Group
- Launch Template dùng cho ASG + launch trực tiếp Console/CLI

Notes (Chi tiết):

- Cả hai cho phép cấu hình sẵn: AMI, instance type/size, storage, key pair, networking/SG, User Data, IAM Role.
- Sau khi tạo thì cấu hình bị khóa; muốn đổi Launch Configuration phải tạo mới.
- Launch Template hỗ trợ versions và các tính năng mới hơn (T2/T3 Unlimited, Placement Groups, Capacity Reservations, Elastic Graphics).
- AWS recommend Launch Templates vì có đủ tính năng của Launch Configurations và nhiều hơn.
- Launch Configuration: chỉ phục vụ Auto Scaling Groups.
- Launch Template: dùng cho ASG và launch EC2 trực tiếp từ Console/CLI.

Summary (Tóm tắt ngắn): Bài học so sánh Launch Configurations và Launch Templates — cả hai đều dùng để định nghĩa trước cấu hình EC2, nhưng đều không sửa được sau khi tạo. Launch Templates mới hơn, có versioning, nhiều tính năng hơn, và dùng được cả cho ASG lẫn launch trực tiếp; AWS khuyến nghị dùng Launch Templates thay cho Launch Configurations.