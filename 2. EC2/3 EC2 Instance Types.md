#### Giới thiệu
Trong bài này sẽ thảo luận những cân nhắc cần thiết khi một SA chọn EC2 instances cho một nhiệm vụ nhất định.
Tôi sẽ giải thích về các loại EC2 instance, nói về cách giải mã tên các loại instance (family, generation, features & size)

Trước đó tôi đã mô tả EC2 instance như là OS và các tài nguyên được cấp phát, bằng cách chọn loại instance và kích thước instance bạn có khả năng kiểm soát chi tiết về cấu hình tài nguyên đó. Chọn một lượng tài nguyên thích hợp và năng lực của instance có thể có ý nghĩa khác nhau giữa hệ thống hiệu năng tốt và một hệ thống gây ra trải nghiệm tệ cho khách hàng.

Hiểu về các loại instance là thứ sẽ dẫn dắt quá trình ra quyết định của bạn, cho một ví dụ, hai người dùng AWS khác nhau có thể chọn hai instance type khác nhau cho cùng một mục đính, bài học quan trọng rút ra sẽ là bạn không đưa ra bất kỳ quyết định tệ nào và bạn có nhận thức về các điểm mạnh và điểm yếu của các loại instance khác nhau. Tôi thỉnh thoảng thấy tính năng này xuất hiện trong bài thi dưới dạng bạn trình bày một vấn đề hiệu suất. và một câu trả lời là thay đổi instance type. Vì vậy bạn cần biết liệu một C instance có tốt hơn M instance trong một tình hướng cụ thể.

#### EC2 Instance
Ở mức khái quát (high level) khi bạn chọn loại instance bạn ảnh hưởng đến vài thứ sau.
**Đầu tiên**: Về mặt logic, là lượng **tài nguyên thô** mà bạn nhận được (CPU, memory, disk amount and type), 
**Thứ hai**: Ngoài lượng tài nguyên thô, còn là **tỷ lệ tài nguyên**: một số instance cho bạn nhiều hơn về một loại tài nguyên và ít hơn về loại còn lại. Ví dụ loại instance phù hợp với các ứng dụng tính toán thường cho bạn nhiều CPU hơn nhưng ít bộ nhớ hơn với cùng một chi phí.
**Thứ ba**: Instance type ảnh hưởng đến lượng **băng thông mạng** dành cho storage và khả năng truyền dữ liệu mà bạn nhận được, yêu tố này rất quan trọng. Khi chúng ta nói về EBS đây là một dịch vụ lưu trữ dựa trên network của AWS. Trong một số tình huống bạn có thể cấp phát volume với hiệu suất rất cao, nhưng nếu bạn không chọn instance phù hợp và chọn loại không cung cấp đủ băng thông mạng cho storage thì chúng instance đó sẽ trở thành nút thắt cổ chai. Vì vậy bạn cần nắm rõ các mực hiệu suất khác nhau mà từng loại instance mang lại.
**Thứ 4**: Việc lựa chọn instance type cũng ảnh hướng đến **kiến trúc phần cứng** mà instance chạy trên đó và có thể cả nhà cung cấp. Ví dụ bạn cân nhắc giữa x86 và Arm, bạn có thể chọn instance type cung cấp CPU Intel hoặc AMD.
Việc lựa chọn loại instance có thể ảnh hưởng một cách chi tiết đến loại phần cứng mà bạn muốn.
**Thứ 5**: Việc chọn instance type phù hợp còn ảnh hưởng đến các tính năng bổ sung mà bạn nhận được có thể là GPU, FPGA (Field Programmable Gateway Arrays) bạn có thể hình dung chúng như một loại CPU đặc biệt mà bạn có thể lập trình phần cứng để thực hiện đúng theo ý muốn.

#### EC2 Categories
Chúng ta có 5 loại EC2 instance chính 
1: **General Purpose**: Đây nên luôn là điểm khởi đầu của bạn, các instance thuộc loại này được thiết kế cho các workload ổn định, chúng có tỷ lệ tài nguyên khá cân bằng. Chỉ khi bạn có yêu cầu cụ thể thì mới chuyển sang các loại khác.
2: **Computed Optimized**: Các instance thuộc category này được thiết kế cho các công việc như xử lý media, tính toán hiệu xuất cao, mô hình hóa khoa học, gaming, machine learning..., chúng cung cấp truy cập vào những CPU hiệu năng cao mới nhất. Chúng thường có tỷ lệ nghiêng về CPU nhiều hơn RAM ở cùng một mức giá.
3: **Memory Optimized**: Về cơ bản ngược lại với Computed Optimized. Nó cung cấp lượng bộ nhớ rất lớn cho cùng một mức chi phí. Loại này rất lý tưởng cho các ứng dụng cần làm việc với tập dữ liệu lớn trong bộ nhớ, chẳng hạn in memory caching hoặc một số loại workload database đặc thù khác.
4: **Accelerated Computing**: Tính toán chuyên biệt hiệu năng cao, chính là nơi các tính năng chuyên biệt phát huy tác dụng, như GPU chuyên dụng dùng cho xử lý xong xong và mô hình hóa, hoặc phần cứng lập trình tùy chỉnh như FPGA, những loại này khá là chuyên biệt, nhưng nếu bạn đang gặp đúng tình huống cần chúng, thì bạn sẽ biết rõ mình cần chúng. Vì vậy khi có yêu cầu chuyên biệt, instance type bạn chọn thường thuộc loại Accelerated Computing.
5: **Storage Optimized**: Các instance dạng này thường cung cấp lượng lưu trữ cục bộ (local storage) siêu nhanh và dung lượng lớn, được thiết kế hoặc để đạt tốc độ truyền dữ liệu tuần tự cao (sequential tranfer rates) hoặc để cung cấp tốc độ nhập xuất siêu lớn IO operations per second (IOPS). loại này phù hợp cho ứng dụng có nhu cầu cao về sequential và ramdom IO. Ví dụ data warehouse, elastic search, và một số loại workload phân tích.
Sequential IO có nghĩa là xem phim 4k, copy file, 

#### Decoding EC2 Types
Một trong những thứ dễ gây nhầm lẫn nhất về EC2 là cách đặt tên của instance type.
**Ví dụ: R5dn.8xlarge**
Nếu thành viên nào trong đội vận hành hỏi bạn cần instance type nào hãy dùng tên đầy đủ bạn sẽ truyền đạt chính xác, không mơ hồ điều bạn cần. Dù phải nói R5dn.8xlarge hơi dài nhưng nó rất chính xác.
**Chữ cái đầu tiên là họ instance (Instance Family)** có rất nhiều họ như T, M, R family... và còn nhiều nữa. Mỗi family được thiết kế cho 1 hoặc một vài workload tính toán cụ thể.
Không ai mong bạn nhớ hết toàn bộ các family, nhưng nếu bạn bắt đầu ghi nhớ nhứng family quan trọng, thì bạn sẽ có lợi thế rất lớn trong kỳ thi khi phải nhận xét instance type có phù hợp không.
**Phần tiếp theo là Generation (thế hệ):** AWS liên tục cải tiến. Ví dụ nếu bạn thấy R5 hoặc C4: 4 hoặc 5 là thế hệ, thế hệ sau thường đi kèm phần cứng tốt hơn và giá trị hiệu năng trên giá tiền tốt hơn. Thường AWS luôn chọn thế hệ mới nhất, nó hầu như luôn luôn cung cấp giá tốt nhất về hiệu năng.
Lý do duy nhất khiến bạn không dùng ngay thế hệ mới nhất là ví nó chưa sẵn có ở region của bạn đang dùng. hoặc doanh nghiệp bạn có quy trình kiểm duyệt nghiêm ngặt trước khi được phê duyệt sử dụng instance mới.
**Size (Kích thước)**: Bây giờ chuyển sang phần bên kia, chúng ta có Size (kích thước), ví dụ ở đây là **8xlarge** (eight extra large) đây chính là kích cỡ instance.
Trong cùng một family và generation luôn có nhiều kích thước khác nhau. Kích thước quyết định instance được cấp bao nhiêu CPU và RAM. Giữa các size thường có mối quan hệ logic và thường là tuyến tính. Tùy thuộc vào family và generation size nhỏ nhất có thể là nano, micro, small, medium, large, xlarge, 2xlarge, 4xlarge, 8xlarge... và tiếp tục tăng. Lưu ý là thường có phí premium nếu dùng size lớn. Vì vậy nhiều khi scale hệ thống bằng cách dùng nhiều instance size nhỏ sẽ tốt hơn. Chúng ta sẽ nói kỹ về phần này ở High Available và Scaling, chỉ cần nhớ rằng với cùng 1 family và generation chúng ta sẽ có nhiều size khác nhau.
