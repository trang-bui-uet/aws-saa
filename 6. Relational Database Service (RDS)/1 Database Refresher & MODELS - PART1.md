Chào mừng bạn quay lại. Và trong bài học kỹ thuật đầu tiên của phần này trong khóa học, tôi muốn cung cấp một bài học cơ bản nhanh về **cơ sở dữ liệu**.

Nếu bạn đã có kinh nghiệm về cơ sở dữ liệu, thì bạn có thể phát video ở tốc độ siêu nhanh và coi bài học này như một sự xác nhận tốt cho những kỹ năng bạn đã có. Nếu bạn chưa có kinh nghiệm về cơ sở dữ liệu thì cũng không sao. Bài học này sẽ giới thiệu vừa đủ kiến thức để bạn vượt qua khóa học, và tôi sẽ bổ sung thêm tài liệu đọc để giúp bạn nắm vững kiến thức cơ bản về cơ sở dữ liệu nói chung.

Bây giờ chúng ta có khá nhiều nội dung cần học, vậy nên hãy bắt đầu ngay thôi.

**Cơ sở dữ liệu** là các hệ thống dùng để **lưu trữ và quản lý dữ liệu**. Nhưng có rất nhiều loại hệ thống cơ sở dữ liệu khác nhau, và có những khác biệt quan trọng về cách dữ liệu được lưu trữ vật lý trên đĩa, cách chúng được quản lý trên đĩa và trong bộ nhớ, cũng như cách hệ thống truy xuất dữ liệu và trình bày cho người dùng.

Hệ thống cơ sở dữ liệu được chia rất rộng thành **quan hệ (relational)** và **phi quan hệ (non-relational)**. Các hệ thống quan hệ thường được gọi là **SQL** hoặc **SQL**. Thực ra cách gọi này không chính xác vì **SQL** là một ngôn ngữ dùng để lưu trữ, cập nhật và truy xuất dữ liệu. Nó được gọi là **Structured Query Language** (Ngôn ngữ Truy vấn Có Cấu trúc), và là một tính năng của hầu hết các nền tảng cơ sở dữ liệu quan hệ. Về mặt nghiêm ngặt, nó khác với thuật ngữ **hệ quản trị cơ sở dữ liệu quan hệ (RDBMS)**, nhưng hầu hết mọi người dùng hai thuật ngữ này thay thế cho nhau. Vì vậy nếu bạn thấy hoặc nghe thuật ngữ **SQL** hoặc **RDBMS**, chúng đều đang đề cập đến các nền tảng cơ sở dữ liệu quan hệ. Hầu hết mọi người dùng chúng thay thế cho nhau.

Một trong những đặc điểm nhận dạng chính của hệ thống cơ sở dữ liệu quan hệ là chúng có **cấu trúc** cho dữ liệu, cả bên trong và giữa các bảng cơ sở dữ liệu. Và tôi sẽ đề cập đến điều đó ngay sau đây. Cấu trúc của một bảng cơ sở dữ liệu được gọi là **schema**, và với hệ thống cơ sở dữ liệu quan hệ, schema này là **cố định** hoặc **cứng nhắc**. Nghĩa là nó được định nghĩa trước khi bạn đưa bất kỳ dữ liệu nào vào hệ thống. Schema định nghĩa tên của các thứ, giá trị hợp lệ của các thứ, và các kiểu dữ liệu được lưu trữ ở đâu. Quan trọng hơn, với hệ thống cơ sở dữ liệu quan hệ, còn có **mối quan hệ cố định** giữa các bảng. Mối quan hệ này cũng cố định và được định nghĩa trước khi bất kỳ dữ liệu nào được nhập vào hệ thống.

Còn **NoSQL** thì sao? Trước hết hãy làm rõ một điều. **NoSQL không phải là một thứ duy nhất**. Như tên gọi cho thấy, NoSQL là tất cả những gì không phù hợp với khuôn mẫu SQL — tất cả những gì không phải là quan hệ. Nhưng điều đó đại diện cho một tập hợp lớn các mô hình cơ sở dữ liệu thay thế, mà tôi sẽ đề cập trong bài học này. Một khác biệt lớn phổ biến áp dụng cho hầu hết các mô hình cơ sở dữ liệu NoSQL là, nhìn chung, chúng có khái niệm về schema **lỏng lẻo hơn nhiều**. Hầu hết chúng đều có schema yếu hoặc không có schema, và mối quan hệ giữa các bảng cũng được xử lý rất khác. Cả hai điều này đều ảnh hưởng đến các tình huống mà một mô hình cụ thể phù hợp, và đó là điều bạn cần hiểu ở mức độ cao cho kỳ thi, cũng như khi bạn chọn mô hình cơ sở dữ liệu để sử dụng trong thực tế.

Trước khi nói về các mô hình cơ sở dữ liệu khác nhau, tôi muốn minh họa trực quan cho bạn cách các hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) hoặc hệ thống SQL khái niệm hóa dữ liệu mà bạn lưu trữ trong chúng. Hãy xem xét một ví dụ về cơ sở dữ liệu thú cưng đơn giản. Bạn có ba người, và với ba người đó, bạn muốn ghi lại những thú cưng mà những người này sở hữu.

Thành phần chính của bất kỳ hệ thống cơ sở dữ liệu dựa trên SQL nào là **bảng (table)**. Mỗi bảng có các **cột**, và chúng được gọi là **thuộc tính (attributes)**. Cột có một tên — tên thuộc tính — và sau đó trong mỗi hàng của bảng đó, mỗi cột phải có một giá trị, và giá trị này được gọi là **giá trị thuộc tính**. Ví dụ trong bảng này, các cột là **Fname** (tên), **Lname** (họ), và **Age** (tuổi). Và với mỗi hàng 1, 2 và 3, hàng đó có một giá trị thuộc tính cho mỗi cột. Vì vậy mỗi thuộc tính (là các cột) đều có một giá trị thuộc tính trong mỗi hàng.

Nhìn chung, cách dữ liệu được mô hình hóa trong hệ quản trị cơ sở dữ liệu quan hệ hoặc hệ thống SQL là dữ liệu có liên quan với nhau được lưu trữ trong một bảng. Trong trường hợp này, tất cả dữ liệu về con người được lưu trong một bảng. Mỗi hàng trong bảng phải có thể nhận dạng duy nhất, vì vậy chúng ta định nghĩa thứ được gọi là **khóa chính (primary key)**. Khóa này là duy nhất trong bảng, và mỗi hàng của bảng đó phải có một giá trị duy nhất cho thuộc tính này. Hãy chú ý trong bảng này mỗi hàng đều có giá trị duy nhất — 1, 2 và 3 — cho khóa chính này.

Với mô hình cơ sở dữ liệu này, chúng ta cũng có một bảng tương tự cho động vật. Chúng ta có Whiskers và Woofy, và chúng cũng có một khóa chính được định nghĩa, đó là **animal hoặc AID**. Và khóa chính trên bảng này cũng phải có giá trị duy nhất trong mọi hàng của bảng. Trong trường hợp này, Whiskers là animal ID 1 và Woofy là animal ID 2.

Mỗi bảng trong hệ quản trị cơ sở dữ liệu quan hệ có thể có các thuộc tính khác nhau. Nhưng đối với một bảng cụ thể, mọi hàng trong bảng đó cần phải có một giá trị được lưu trữ cho mọi thuộc tính trong bảng đó. Hãy xem bảng động vật có Name và Types, trong khi bảng con người có First Name, Last Name và Age. Nhưng hãy chú ý rằng ở cả hai bảng, với mọi hàng, mọi thuộc tính đều phải có một giá trị.

Vì hệ thống SQL là quan hệ, chúng ta thường định nghĩa **mối quan hệ** giữa các bảng. Đây là một **bảng nối (join table)**. Nó giúp dễ dàng có các mối quan hệ nhiều-nhiều. Một người có thể có nhiều động vật, và mỗi động vật có thể có nhiều người hầu. Một bảng nối có cái được gọi là **khóa tổng hợp (composite key)**, là khóa được tạo thành từ hai phần. Và với khóa tổng hợp, chúng phải duy nhất khi kết hợp với nhau. Hãy chú ý hàng thứ hai và thứ ba có cùng animal ID. Điều đó ổn vì human ID khác nhau. Miễn là khóa tổng hợp trong toàn bộ là duy nhất thì vẫn ổn.

Các khóa trong các bảng khác nhau chính là cách các mối quan hệ giữa các bảng được định nghĩa. Trong ví dụ này, bảng con người có mối quan hệ với bảng nối. Nó cho phép mỗi người có nhiều động vật, và mỗi động vật có nhiều người. Trong ví dụ này, animal ID 2 (là Woofy) được liên kết với human ID 2 và 3 (là Julie và James). Cả hai đều là người hầu của Woofy vì chú chó đó cần gấp đôi số bánh thưởng ngon lành.

Tất cả các khóa và mối quan hệ này đều được định nghĩa trước. Việc này được thực hiện bằng schema. Nó cố định, và rất khó thay đổi sau khi dữ liệu đầu tiên được đưa vào. Việc schema quá cố định và phải được khai báo trước khiến hệ thống SQL hoặc quan hệ khó lưu trữ bất kỳ dữ liệu nào có mối quan hệ thay đổi nhanh chóng. Một ví dụ tốt cho điều này là mạng xã hội như Facebook, nơi các mối quan hệ thay đổi liên tục.

Đây là một ví dụ đơn giản về hệ thống cơ sở dữ liệu quan hệ. Nó thường có nhiều bảng. Một bảng lưu trữ dữ liệu có liên quan, ví dụ con người và động vật. Các bảng có schema cố định, có thuộc tính, có hàng. Mỗi hàng có giá trị khóa chính duy nhất và phải chứa một số giá trị cho tất cả các thuộc tính trong bảng. Và sau đó các bảng đó có mối quan hệ với nhau, cũng cố định và được định nghĩa trước. Đây chính là SQL. Đây chính là mô hình hóa cơ sở dữ liệu quan hệ.

Được rồi, đây là phần cuối của phần một của bài học này. Nội dung hơi dài một chút nên tôi muốn thêm một khoảng nghỉ. Đây là cơ hội để bạn nghỉ ngơi hoặc lấy một ly cà phê. Phần hai sẽ tiếp tục ngay từ điểm kết thúc của phần một. Vậy hãy hoàn thành video, và khi bạn sẵn sàng, hãy tham gia cùng tôi ở phần hai.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Cơ sở dữ liệu: hệ thống lưu trữ & quản lý dữ liệu
- Phân loại: Relational (SQL/RDBMS) vs Non-relational (NoSQL)
- Schema cố định trong RDBMS
- Bảng, thuộc tính, hàng, khóa chính (Primary Key)
- Bảng nối (Join table) & khóa tổng hợp (Composite Key)
- Mối quan hệ cố định giữa các bảng
- Hạn chế của schema cứng với dữ liệu thay đổi nhanh (ví dụ mạng xã hội)

**Notes (Chi tiết):**

- **Cơ sở dữ liệu** lưu trữ và quản lý dữ liệu, có sự khác biệt về cách lưu trữ vật lý, quản lý bộ nhớ và truy xuất dữ liệu.
- Hệ thống được chia thành **quan hệ (relational)** và **phi quan hệ (non-relational)**.
- **SQL** là ngôn ngữ truy vấn có cấu trúc, thường bị dùng thay thế cho **RDBMS** (hệ quản trị cơ sở dữ liệu quan hệ).
- Đặc điểm chính của RDBMS: có **schema cố định**, định nghĩa trước tên, kiểu dữ liệu, giá trị hợp lệ và **mối quan hệ giữa các bảng**.
- **NoSQL** không phải một mô hình duy nhất, mà là tất cả những gì không phải quan hệ; thường có schema lỏng lẻo hoặc không có schema.
- Ví dụ mô hình quan hệ: bảng **Humans** (Fname, Lname, Age + Primary Key), bảng **Animals** (Name, Type + Primary Key AID), và **Join table** dùng **Composite Key** để tạo quan hệ nhiều-nhiều.
- Mỗi hàng phải có giá trị cho mọi thuộc tính; khóa chính phải duy nhất; khóa tổng hợp phải duy nhất khi kết hợp.
- Schema cố định khiến RDBMS khó thích ứng với dữ liệu có mối quan hệ thay đổi liên tục (ví dụ Facebook).

**Summary (Tóm tắt ngắn):** Bài học giới thiệu kiến thức nền tảng về **cơ sở dữ liệu**, tập trung vào sự khác biệt giữa **hệ thống quan hệ (SQL/RDBMS)** và **NoSQL**. RDBMS sử dụng **schema cố định**, bảng có thuộc tính, hàng và **khóa chính**, cùng với **bảng nối** để tạo mối quan hệ. Schema cứng nhắc phù hợp với dữ liệu ổn định nhưng kém linh hoạt với dữ liệu thay đổi nhanh.