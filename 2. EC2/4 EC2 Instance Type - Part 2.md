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