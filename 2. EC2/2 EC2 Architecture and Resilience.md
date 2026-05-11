Bài học này sẽ tóm tắt kiến trúc của EC2
Nhìn cụ thể vào máy chủ EC2, chúng được thiết kế vật lý như thế nào, tại sao chúng có khả năng phục hồi ở mức AZ, và khả năng phục hồi của EBS (Elastic Block Store) là nhân tố quan trọng trong quyết định của SA.

### EC2 Architecture

#### Các điểm chính
EC2 instances là virtual machine (OS + Resource)
EC2 instances chạy trên EC2 hosts và chúng là máy chủ vật lý thật (Physical Service Hardware)
Máy chủ này (EC2 hosts) có thể là Shared Hosts (Chia sẻ) hoặc Dedicated Hosts (Dành riêng)
Shared Host: Bạn chi sẻ máy chủ với nhiều dùng khác, bạn chỉ trả tiền cho instances bạn chạy
Dedicated Host: Bạn sở hữu nguyên máy chủ, bạn trả tiền cho toàn bộ máy chủ
EC2 là một dịch vụ có khả năng phục hồi (resilient) ở mức AZ lý do là hosts thì chạy trong 1 AZ nên AZ lỗi, Host lỗi, Instance lỗi.

#### Kiến trúc:
Chúng ta có region us-east-1, trong region có hai AZ AZ-A, AZ-B, trong AZ-A chúng ta có hai subnet sn-A, sn-B.
Trong mỗi AZ chúng ta có một EC2 host, mỗi EC2 host chạy trong 1 AZ
EC2 hosts có vài phần cứng nội bộ logically CPU và memory, chúng cũng có ổ cứng cục bộ gọi là Instance Store, Instance store chỉ lưu dữ liệu tạm thời, dữ liệu sẽ mất khi instance stop hoặc terminate, host bị lỗi, instance bị reboot. Nếu instance bị di chuyển từ host này sang host khác thì dữ liệu sẽ mất.
Ngoài ra chúng ta có hai loại networking: Storage networking mạng cho lưu trữ, **Data networking** mạng cho dữ liệu.
Khi instance được cấp phát vào một subnet cụ thể trong VPC thì thực chất là: Một **Primary Elastic Network Interface (ENI)** được khởi tạo trong subnet, và nó ánh xạ trực tiếp tới phần cứng mạng vật lý trên EC2 host.
ENI (Elastic Network Interface) có thể coi là một card mạng ảo (virtual NIC) được AWS tạo ra và gắn trực tiếp vào subnet mà bạn chọn.
Instances có thể có nhiều network interfaces thậm chí trong các subnet khác nhau miễn là chúng thuộc 1 AZ. Mọi thứ về EC2 thì tập chung xung quanh kiến trúc này, sự thật là nó chạy trong 1 AZ cụ thể.
EC2 có thể sử dụng remote storage, cho nên EC2 có thể kết nối tới **Elastic Block Store (EBS)**, EBS thì cũng chạy bên trong một AZ cụ thể, cho nên dịch vụ chạy trong AZ-A thì khác với dịch vụ chạy trong AZ-B và bạn không thể truy cập chúng qua các AZ khác nhau, EBS cho bạn cấp phát volumes (ổ cứng), và những ổ cứng lưu trữ bền vững, và chúng có thể cấp phát cho instances trong cùng 1 AZ.
Tại sao AZ lại quan trọng: Bởi vì hosts chạy trong 1 AZ, Network thì thuộc 1 AZ, Persistent Storage cũng thuộc 1 AZ. Nếu 1 AZ trong AWS gặp sự cố lớn nó sẽ ảnh hưởng đến toàn những thữ trên.
Nếu 1 Instance chạy trên 1 host cụ thể, nếu bạn restart nó vẫn sẽ ở trong host đó. Instance sẽ vẫn ở trong host cho đến khi 1 trong hai thứ xảy ra:  đầu tiên là host bị lỗi, hoặc nó bị tắt cho việc bảo trì vì 1 lý do nào đó cùa AWS, thứ hai là một instance đã stopped sau đó started trở lại, nó khác với việc restarting, đó là việc bắt buộc 1 instance stopped và sau đó started nó khác với việc restart. Khi đó instance sẽ được chạy ở 1 host khác và host đó vẫn sẽ cùng AZ với host cũ.
Instance không thể di chuyển tự nhiên giữa các AZ, tất cả mọi thứ liên quan đến chúng hardware, network, storage bị khóa chặt bên trong 1 AZ. Có một cách để bạn di chuyển tuy nhiên nó về cơ bản là sao chép instance sau đó tạo một cái mới ở AZ khác, cái này sẽ được trao đổi ở phần snapshot và AMIS.
Cái mà bạn không thể làm đó là kết nối Network Interface hoặc EBS storage được đặt tại một AZ tới một EC2 instance ở AZ khác, EC2 và EBS cả hai là AZ services chúng được cô lập trong AZ, bạn không thể kết nối giữa các AZ với instance và EBS volumes.
Instance chạy trong host thì chia sẻ tài nguyền của host và instance với nhiều kích cỡ khác nhau có thể cùng chia sẻ chung 1 host nhưng về cơ bản instance thuộc cùng 1 kiểu và cùng thế hệ sẽ cùng chiếm 1 host.
Khi bạn nghĩ về EC2 host hãy nghĩ rằng nó thuộc 1 một năm nhất định bao gồm 1 thế hệ CPU, RAM về Disk nhất định và instances cũng được tạo ra với nhiều thế hệ khác nhau, những phiên bản khác nhau sẽ xử dụng các loại CPU, RAM, DISK cụ thể, cho nên hai instance thuộc hai thế hệ khác nhau sẽ chạy ở hai host khác nhau. Cho nên host sẽ có nhiều instances của nhiều customer khác nhau thuộc cùng một type nhưng khác nhau về sizes

#### Hãy trả lời câu hỏi: EC2 good for?
Loại tình huống bạn sẽ sử dụng EC2 là gì?
- Khi ứng dụng cần **hệ điều hành cụ thể** (Windows, Linux phiên bản cũ…).
- Cần **runtime** và **cấu hình** nhất định.
- Đội ngũ IT nội bộ quen dùng cách truyền thống.
- Nhà cung cấp phần mềm chỉ hỗ trợ chạy trên server thông thường (không hỗ trợ container/serverless).
Đây là lý do EC2 vẫn rất phổ biến dù có Lambda, Fargate, ECS… ra đời.

Và nó cũng rất tuyệt vời cho nhu cầu tính toán cần chạy thời gian dài, có nhiều service trong AWS cung cấp khả năng tính toán tuy nhiên nhiều trong số chúng gặp phải giới hạn thời gian, vì vậy bạn không thể để những dịch vụ này chạy mãi cho 1 hoặc 2 năm. Với EC2 nó thiết kế cho bền vững cho nhu cầu chạy tính toán thời gian dài. Cho nên nếu bạn có một ứng dụng cần chạy 24/7 thì và cần chạy trên một hệ điều hành linux or window thì EC2 là lựa chọn hiển nhiên cho việc này.

Nếu bạn có bất kỳ ứng dụng dạng Server chạy chờ kết nối tới thì EC2 là lựa chọn cho việc này.

EC2 thì hoàn hảo cho dịch vụ nào cần khả năng tăng đột biến (brust) hoặc ổn định liên tục (steady state), có nhiều loại EC2 khác nhau phù hợp với mức tải bình thường thấp, kèm theo những lần tăng đột biến, cũng nhưng trạng thái tải ổn định lâu dài. 

EC2 hoàn hảo cho ứng dụng monolithic stack, nếu ứng dụng monolithic yêu cầu những thành phần có thể là database, middleware, có thể là những thành phần runtime hoặc yêu cầu traditional OS, EC2 là thứ đầu tiên bạn nên nghĩ tới.

EC2 cũng phù hợp với việc chuyển đổi các ứng dụng cũ on on-permises (VM ware, Physical server) sang AWS, những ứng dụng yêu cầu môi trường máy ảo, server thông thường, hoặc khi bạn thự hiện dựng một môi trường dự phòng (Disaster Recovery) cho các hệ thống truyền thống.

