Trong bài học ôn thi này, tôi sẽ chia sẻ một số gợi ý về **khi nào nên chọn EBS thay vì EC2 Instance Store** và ngược lại.
Vì các câu hỏi thi thường xuyên kiểm tra khả năng phân biệt và lựa chọn giữa hai loại lưu trữ này, nên việc hiểu rõ các tiêu chí là cực kỳ quan trọng để đạt kết quả cao.

Chào mừng bạn đến với bài học này, nơi tôi muốn **tóm tắt ngắn gọn** một số tình huống mà bạn sẽ **chọn sử dụng EBS thay vì Instance Store volumes**.

Tôi cũng muốn đề cập đến những tình huống **Instance Store phù hợp hơn EBS** và những trường hợp **phụ thuộc vào ngữ cảnh**. Vì luôn tồn tại những tình huống mà **cả hai đều có thể dùng được** hoặc **không loại nào phù hợp**.

Bây giờ chúng ta có rất nhiều nội dung cần học nên hãy bắt đầu ngay.

Tôi muốn **xin lỗi ngay từ đầu**. Bạn cũng biết rồi, tôi rất ghét những bài học chỉ toàn nói về **sự kiện, số liệu, con số và các viết tắt**. Tôi gần như luôn thích dùng **biểu đồ, ví dụ thực tế, kiến trúc thực tế và cách triển khai**. Tuy nhiên, đôi khi chúng ta vẫn cần đi qua các con số và sự kiện, và đây là một trong những lúc như vậy. Xin lỗi, nhưng chúng ta buộc phải làm vậy.

Vì vậy bài học này sẽ tập trung vào **một số điểm tình huống hữu ích**, một số **giá trị tối thiểu và tối đa**, cùng các tình huống giúp bạn **quyết định giữa Instance Store volumes và EBS**, và những nội dung này sẽ rất hữu ích **cả cho kỳ thi lẫn sử dụng thực tế**.

**Quy tắc mặc định đầu tiên**: Nếu bạn cần **persistent storage** (lưu trữ bền vững) thì bạn **nên mặc định chọn EBS**, hoặc cụ thể hơn là **mặc định tránh Instance Store volumes**.

**Instance Store volumes không phải là persistent storage**. Có rất nhiều lý do khiến dữ liệu có thể bị mất: **hardware failure**, instance reboot, maintenance, bất kỳ thứ gì khiến instance di chuyển giữa các host đều có thể ảnh hưởng đến **Instance Store volumes**, và đây là kiến thức **rất quan trọng** cần nắm cho cả kỳ thi lẫn thực tế.

Nếu bạn cần **resilient storage** (lưu trữ có khả năng phục hồi cao) thì **nên tránh Instance Store volumes** và **mặc định dùng EBS**. Một lần nữa, nếu hardware fail, instance di chuyển, host fail, bất kỳ sự cố nào thuộc loại này đều có thể gây mất dữ liệu trên Instance Store volumes vì chúng **không resilient**.

**EBS** cung cấp phần cứng **resilient trong cùng một Availability Zone**, và bạn còn có khả năng **snapshot volume ra S3**, vì vậy **EBS là lựa chọn tốt hơn rất nhiều** nếu bạn cần tính resilient.

Tiếp theo, nếu bạn có dữ liệu lưu trữ cần **được tách biệt khỏi vòng đời của instance**, thì hãy sử dụng **EBS**.

Vì vậy nếu bạn cần một volume có thể attach vào một instance, sử dụng nó một thời gian, detach ra và sau đó reattach vào instance khác, thì **EBS** chính là thứ bạn cần.

Đây là những tình huống mà việc sử dụng **EBS** sẽ hợp lý hơn rất nhiều.

Với bất kỳ điều nào tôi đã đề cập ở trên, nó khá **rõ ràng**. Hãy **sử dụng EBS**. Hay nói cách khác là **tránh Instance Store volumes**.

Bây giờ có một số tình huống mà không rõ ràng lắm và bạn cần phải chú ý đến chúng trong kỳ thi. Giả sử rằng bạn cần **resilience** nhưng ứng dụng của bạn hỗ trợ **built-in replication**. Lúc đó bạn có thể sử dụng rất nhiều Instance Store volumes trên nhiều instance và theo cách đó bạn vẫn nhận được **lợi ích hiệu năng** của Instance Store volumes mà **không có rủi ro tiêu cực**.

Một tình huống khác mà nó phụ thuộc là khi bạn cần **high performance**. Ở một mức độ nào đó, và tôi sẽ nói chi tiết về các mức hiệu năng khác nhau ngay sau đây, cả **EBS** lẫn **Instance Store volumes** đều có thể cung cấp high performance. Tuy nhiên đối với **super high performance** thì bạn sẽ cần mặc định sử dụng **Instance Store volumes**.

Cuối cùng, **Instance Store volumes** được bao gồm trong giá của nhiều loại EC2 instance nên việc tận dụng chúng là hợp lý. Nếu **cost** là mối quan tâm chính thì bạn nên xem xét sử dụng **Instance Store volumes**.

Đây là những kịch bản **high-level** và những sự kiện quan trọng này sẽ giúp bạn rất tốt trong kỳ thi. Nó sẽ giúp bạn chọn giữa **Instance Store volumes** và **EBS** trong hầu hết các tình huống thi phổ biến, nhưng bây giờ tôi muốn đề cập đến một số sự kiện và **con số cụ thể** mà bạn cần nắm rõ. Bây giờ nếu bạn gặp các câu hỏi trong kỳ thi mà... 

Nếu trong bài thi bạn thấy các câu hỏi **chỉ tập trung vào hiệu quả chi phí** và bạn nghĩ cần dùng **EBS**, thì nên mặc định chọn **ST1 hoặc SC1** vì chúng **rẻ hơn**. Đây là các loại lưu trữ **ổ đĩa cơ học**, nên sẽ rẻ hơn so với các EBS volume dựa trên **SSD**.

Nếu câu hỏi đề cập đến **throughput** hoặc **streaming**, thì nên mặc định chọn **ST1**, trừ khi câu hỏi nhắc đến **boot volume**. **ST1 và SC1 không dùng được làm boot volume**, nên bạn không thể dùng hai loại lưu trữ cơ học này để **khởi động EC2 instance**. Đây là điểm **rất quan trọng cần nhớ cho kỳ thi**.

Tiếp theo là một số mức hiệu năng chính. **GP2 và GP3** đều có thể cung cấp tối đa **16.000 IOPS trên mỗi volume**. Với **GP2**, hiệu năng dựa trên **kích thước của volume**. Với **GP3**, bạn có **3.000 IOPS mặc định** và có thể **trả thêm tiền** để tăng hiệu năng. Tuy nhiên, với cả **GP2 và GP3**, hiệu năng tối đa trên mỗi volume là **16.000 IOPS**. Hãy ghi nhớ con số này cho các câu hỏi trong bài thi.

**IO1 và IO2** có thể cung cấp tối đa **64.000 IOPS**. Vì vậy, nếu bạn cần từ **16.000 IOPS đến 64.000 IOPS** trên một volume, bạn cần chọn **IO1 hoặc IO2**.

Dấu hoa thị được nhắc đến vì có một loại volume mới gọi là **IO2 Block Express**, có thể cung cấp tối đa **256.000 IOPS trên mỗi volume**. Tuy nhiên, cần nhớ rằng các mức hiệu năng cao này chỉ có thể đạt được khi bạn dùng các **instance EC2 kích thước lớn hơn**.

Các thông tin này tập trung vào **hiệu năng tối đa có thể đạt được với EBS volume**. Bạn cần đảm bảo ghép EBS volume với một **EC2 instance đủ lớn**, có khả năng cung cấp các mức hiệu năng đó.

Một lựa chọn thường xuất hiện trong bài thi là bạn có thể lấy nhiều **EBS volume riêng lẻ** và tạo một bộ **RAID 0** từ các EBS volume đó. Bộ **RAID 0** này sau đó có thể đạt được **hiệu năng tổng hợp của tất cả các volume riêng lẻ**.

Nhưng mức này chỉ lên đến **260.000 IOPS**, vì đây là **IOPS tối đa có thể đạt được trên mỗi instance**.

Vì vậy, dù bạn kết hợp bao nhiêu volume với nhau, bạn vẫn luôn phải chú ý đến **hiệu năng tối đa có thể đạt được trên EC2 instance**. Hiện tại, mức hiệu năng cao nhất có thể đạt được khi dùng **EC2 + EBS** là **260.000 IOPS**.

Để đạt được mức này, bạn cần dùng **instance kích thước lớn** và có **đủ EBS volume** để tận dụng toàn bộ dung lượng hiệu năng đó. Bạn cần ghi nhớ cả **hiệu năng mà từng volume cung cấp** lẫn **hiệu năng tối đa của chính instance**. Hiện tại, giới hạn tối đa là **260.000 IOPS**.

Nếu bạn cần nhiều hơn **260.000 IOPS** và ứng dụng của bạn có thể chấp nhận loại lưu trữ **không bền vững / không persistent**, thì bạn có thể dùng **instance store volumes**.

**Instance store volumes** có thể cung cấp mức hiệu năng cao hơn nhiều. Nếu chọn đúng **loại instance** và dùng các **instance store volume gắn kèm**, bạn có thể đạt tới **hàng triệu IOPS**. Tuy nhiên, cần luôn nhớ rằng loại lưu trữ này **không persistent**.

Điều đó có nghĩa là bạn đang đánh đổi **tính bền vững của dữ liệu** để lấy **hiệu năng cao hơn rất nhiều**.

Một lần nữa, dù không khuyến khích học thuộc lòng quá nhiều, lời khuyên là bạn nên cố gắng **ghi nhớ tất cả các con số này**. Slide này sẽ được đưa vào tài liệu học tập trong khóa học hoặc GitHub repository.

Bạn có thể **in ra**, **chụp màn hình**, hoặc đưa vào **ghi chú điện tử** tùy theo phương pháp học của bạn. Bạn cần nhớ tất cả **sự kiện và con số** trong bài học này, vì nếu nhớ được chúng, việc trả lời các câu hỏi liên quan đến **hiệu năng** trong kỳ thi sẽ dễ hơn rất nhiều.

Thông thường, việc học thuộc **số liệu thô** không phải là cách học hiệu quả, nhưng đây là **một ngoại lệ trong AWS**. Vì vậy, hãy cố gắng ghi nhớ các **mức hiệu năng khác nhau** và **công nghệ cần dùng để đạt được từng mức hiệu năng đó**.