### Giới thiệu về **cfn-init**

Trong bài trước, CloudFormation xử lý **User Data** bằng cách truyền dữ liệu đã mã hóa **Base64** vào instance, sau đó hệ điều hành chạy nó như một shell script.

Bây giờ, có một cách mạnh mẽ hơn để cấu hình EC2 instance: **cfn-init**. Đây là **helper script** chính thức của AWS, được cài sẵn trên Amazon Linux 2 và các OS khác.

**cfn-init** không chỉ là script đơn giản mà giống như một **hệ thống quản lý cấu hình** (configuration management).

- **User Data** là **procedural** (chạy từng lệnh theo thứ tự).
- **cfn-init** có thể chạy procedural nhưng cũng hỗ trợ **desired state** (trạng thái mong muốn).

Bạn chỉ cần mô tả **trạng thái mong muốn** của instance, **cfn-init** sẽ tự động thực hiện các bước cần thiết để đạt được trạng thái đó.

### Những tính năng mạnh mẽ của **cfn-init**

- Cài đặt **packages** với phiên bản cụ thể (nếu đã đúng phiên bản thì không làm gì).
- Quản lý **users** và **groups** trên OS.
- Tải source code, giải nén, hỗ trợ authentication.
- Tạo **files** với nội dung, quyền (permissions) và owner cụ thể.
- Chạy **commands** và kiểm tra điều kiện sau khi chạy.
- Quản lý **services** (start, enable, restart).

**cfn-init** được gọi từ **User Data**, nhưng lấy cấu hình từ **CloudFormation stack** thông qua phần **Metadata** với **AWS::CloudFormation::Init** trong template.

### Kiến trúc hoạt động của cfn-init

1. CloudFormation template chứa **logical resource** (EC2Instance) với **Metadata > AWS::CloudFormation::Init**.
2. Khi stack tạo instance, **User Data** chạy lệnh **cfn-init** (với stack ID và region được tự động thay thế).
3. **cfn-init** giao tiếp với CloudFormation service để lấy cấu hình **desired state**.
4. Thực hiện cấu hình và đưa instance về trạng thái mong muốn.

**Ưu điểm lớn**: Hỗ trợ **stack updates** – nếu metadata thay đổi, có thể chạy lại **cfn-init** để cập nhật instance mà không cần recreate.

### Creation Policy và cfn-signal

**User Data** chỉ chạy một lần khi launch. CloudFormation không biết bên trong instance thành công hay thất bại.

**Creation Policy** giải quyết vấn đề này:

- Thêm vào logical resource với **timeout** (ví dụ: 15 phút).
- CloudFormation sẽ **chờ signal** từ instance thay vì chỉ dựa vào EC2 báo provision thành công.

**cfn-signal** được chạy ở cuối User Data:

- Gửi tín hiệu **success** hoặc **failure** dựa trên exit code của **cfn-init** (-e $?).
- Nếu thành công → resource chuyển sang **CREATE_COMPLETE**.
- Nếu lỗi hoặc timeout → stack báo lỗi.

Tính năng này rất quan trọng khi bootstrapping phức tạp, đặc biệt với EC2 và Auto Scaling Groups.

### Kết luận

Bài học chuẩn bị cho demo sử dụng template WordPress đã nâng cấp với **cfn-init** + **Creation Policy**. Đây là bước tiến lớn so với User Data thuần túy, giúp bootstrapping đáng tin cậy và có thể update.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- **cfn-init** vs User Data (procedural vs desired state)
- Tính năng cfn-init: packages, files, users, services, commands
- Metadata **AWS::CloudFormation::Init**
- cfn-init trong User Data + stack ID/region
- **Creation Policy** + **cfn-signal** (timeout, success/failure)
- Hỗ trợ stack updates

**Notes (Chi tiết):**

- **cfn-init** là helper script mạnh mẽ, lấy cấu hình từ CloudFormation template qua **Metadata**, thực thi **desired state** (cài Apache đúng version, quản lý service, tạo file…).
- Lệnh cfn-init trong **User Data** được CloudFormation tự động inject giá trị stack/region.
- **Creation Policy** buộc CloudFormation chờ **cfn-signal** trước khi đánh dấu CREATE_COMPLETE.
- **cfn-signal -e $?** báo kết quả của cfn-init lên stack (thành công hoặc lỗi).
- Timeout (ví dụ 15 phút) tránh treo stack nếu bootstrapping thất bại.
- Khác biệt lớn với User Data: hỗ trợ update và xác thực kết quả thực sự.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu **cfn-init** như một công cụ configuration management mạnh mẽ hơn **User Data**, kết hợp với **Creation Policy** và **cfn-signal** để CloudFormation biết chính xác instance đã bootstrap thành công hay chưa. Đây là nền tảng quan trọng cho bootstrapping phức tạp và stack updates trong AWS.