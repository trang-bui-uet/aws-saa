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

Đây là những kịch bản **high-level** và những sự kiện quan trọng này sẽ giúp bạn rất tốt trong kỳ thi. Nó sẽ giúp bạn chọn giữa **Instance Store volumes** và **EBS** trong hầu hết các tình huống thi phổ biến, nhưng bây giờ tôi muốn đề cập đến một số sự kiện và **con số cụ thể** mà bạn cần nắm rõ. Bây giờ nếu bạn gặp các câu hỏi trong kỳ thi mà... 3p56s.