Bài này chúng ta sẽ đi tiếp những yếu tô cần cân nhắc khi một SA chọn EC2 instance cho một công việc cụ thể.

Tổng quan về các loại EC2 instances, sau đó sẽ đi sau vào từng category, liệt kê các loại phổ biến nhất hoặc các thế hệ hiện tại đang sẵn có.

Bảng này quan trọng hãy theo khảo liên tục khi chúng ta học khóa học SA. Để bất cứ khi nào chúng ta nói về một size cụ thể, type, generation của instance. Nếu bạn tham khảo cột Details/Notes bạn sẽ có khả năng để bắt đầu tạo ra các liên tưởng trong đầu giữa type và sau đó là các tính năng bổ sung mà bạn nhận được

![[Pasted image 20260511173224.png]]

Ví dụ, nếu chúng ta nhìn vào **hạng mục General Purpose**, chúng ta có ba mục chính trong hạng mục này. Đó là các loại **A1** và **M6g**, đây là một loại instance cụ thể dựa trên bộ xử lý **ARM**.

A1 sử dụng bộ xử lý **Graviton ARM** do chính AWS thiết kế. Và khi sử dụng các bộ xử lý dựa trên ARM, miễn là bạn có hệ điều hành và ứng dụng có thể chạy trên kiến trúc đó, chúng sẽ **rất hiệu quả**. Nhờ vậy bạn có thể dùng instance nhỏ hơn với chi phí thấp hơn, mà vẫn đạt được mức hiệu suất rất tốt.

Các loại **T3** và **T3a** là **burstable instances** (instance có khả năng bùng nổ). Giả định với những loại instance này là tải CPU bình thường của bạn sẽ khá thấp, và bạn được cấp một lượng **burst credits** cho phép thỉnh thoảng tăng đột biến lên mức cao hơn, sau đó quay về mức CPU thấp bình thường.

Vì vậy loại instance T3 và T3a rất phù hợp cho những máy có tải bình thường thấp nhưng thỉnh thoảng có burst. Ngoài ra chúng cũng **rẻ hơn khá nhiều** so với các loại general purpose instance khác.

Tiếp theo chúng ta có **M5, M5a và M5n**. M5 là điểm khởi đầu của bạn. **M5a** sử dụng kiến trúc **AMD**, trong khi M5 thông thường sử dụng **Intel**. Đây là những instance general purpose ổn định (steady state).

Vậy nếu bạn không có nhu cầu burst, đang chạy một loại application server nào đó đòi hỏi CPU ổn định liên tục, thì bạn có thể dùng loại **M5**. Ví dụ như một mail server Exchange được sử dụng nhiều, chạy ở mức **60% CPU** bình thường — đây sẽ là ứng cử viên tốt cho M5.

Nhưng nếu bạn có **Domain Controller** hoặc **email relay server** thường chỉ chạy ở mức **2-3% CPU**, thỉnh thoảng burst lên 20-30-40%, thì bạn nên dùng loại **T**."

Chúng ta có category **Compute Optimized** với C5 và C5n, và chúng rất phù hợp cho media encoding, scientific modeling, gaming servers, general machine learning.

**Memory Optimized**, chúng ta bắt đầu với R5 và R5a. Nếu bạn muốn dùng các ứng dụng in-memory thực sự lớn, thì có X1 và X1e. Nếu muốn dung lượng memory cao nhất trong tất cả các instance của AWS, thì có High Memory series. Bạn cũng có Z1d đi kèm large memory và NVMe storage.

Tiếp theo là **Accelerated Computing**, đây là những instance đi kèm với các tính năng bổ sung. Ví dụ P3 type và G4 type đi kèm với các loại GPU khác nhau. P type rất mạnh về parallel processing và machine learning. G type khá tốt cho machine learning, và tốt hơn nhiều cho các workload đồ họa nặng (graphics intensive). Bạn có F1 type đi kèm Field Programmable Gate Arrays (FPGA), rất phù hợp cho genomics, financial analysis, big data, hoặc bất kỳ việc gì bạn muốn lập trình phần cứng để thực hiện nhiệm vụ cụ thể.

Bạn có Inf1 type khá mới, được thiết kế chuyên biệt cho machine learning inference, recommendation, forecasting analysis, voice conversation, bất kỳ thứ gì liên quan đến machine learning, hãy cân nhắc dùng loại này.

Và cuối cùng là **Storage Optimized**, những instance này đi kèm high-speed local storage. Tùy loại bạn chọn, bạn có thể đạt high throughput hoặc maximum IOPS, hoặc ở mức ở giữa.

Vì vậy hãy giữ phần này ở nơi dễ tìm, in ra hoặc lưu điện tử. Khi học qua khóa học và dùng các instance type khác nhau, hãy mở ra tham khảo và bắt đầu tạo sự liên kết trong đầu: category là gì, instance type nào thuộc category đó, và chúng mang lại lợi ích gì.

Một lần nữa, đừng lo phải học thuộc lòng hết tất cả để thi. Bạn không cần làm vậy. Mình sẽ vẽ ra và giải thích bất cứ thứ gì bạn cần trong suốt khóa học. Chỉ cần cố gắng nắm được chữ cái nào thuộc category nào. Nếu mình đưa ra T type, C type hay R type, bạn có thể liên tưởng ngay ra category tương ứng — đó đã là một bước tiến rất lớn rồi.

**Có các cách để liên tưởng từ type ra loại category nào:**
C: Computed
R: Ram
I: IO
D: Desen Storage (HDD) lưu trữ mật độ cao, Data warehouse
G: GPU
P: Parallel Processing

Tuy nhiên, điều quan trọng cần hiểu là cách chọn loại instance cho một tình huống tính toán cụ thể.

Trang hữu ích tác giải xem liên tục
1: Trang tài liệu AWS [https://aws.amazon.com/ec2/instance-types/](https://aws.amazon.com/ec2/instance-types/). Bạn sẽ thấy các trường hợp sử dụng instance type, các tính năng đặc biệt, danh sách mỗi instance size, và chính xác những tài nguyên được cấp phát, và các ghi chú đặc biệt bạn cần biết.
2: Website thứ hai là một thứ tương tự: [https://ec2instances.info/](https://ec2instances.info/). nó cung cấp một danh sách được sắp xếp bạn có thể filter cho bạn cái nhìn overview về chính xác instance type cung cấp những gì.