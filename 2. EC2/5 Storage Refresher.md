Trong bài này chúng ta sẽ tóm tắt nhanh một vài thông tin quan trọng liên quan đến lưu trữ.
Bao gồm:
- Block Storage
- File Storage
- Object Storage
- IO Block Size
- IOPS
- Throughput

Trong bài kiểm tra mong đợi bạn biết loại storage thích hợp cho một tình huống cụ thể. Cho nên trước khi chúng ta sang các bài học cụ thể về AWS storage, tôi muốn tóm tắt nhanh.

Hãy bắt đầu bằng cách nói về một vài thuật ngữ quan trọng về Storage.
**Thuật ngữ quan trọng**:
1: **Direct (local) Attached Storage** - Storage ở trên EC2 host, đây là ổ cứng kết nối trực tiếp với Ec2 host. Nó gọi là Instance Store, lưu trữ gắn trực tiếp thông thường siêu nhanh bởi vì nó gắn trực tiếp vào phần cứng. Nhưng nó lại chịu thiệt hại từ một vài vấn đề. Nếu ổ cứng, phần cứng lỗi storage có thể bị mất. Nếu một EC2 instance di chuyển giữa các host storage có thể mất. Giải pháp thay thế là lưu trữ gắn qua mạng (Network attached storage)
2: **Network Attached Storage**: Nơi mà các volume được tạo ra, và được gắn tới một thiết bị qua mạng. Trong môi trường tại chỗ (on permises) loại này sử dụng các giao thức như là ISCSI hoặc Fiber channel, trong AWS nó sử dụng một sản phẩm gọi là Elastic Block Store (EBS), Network Storage nói chung là có khả năng phục hồi cao, và nó tách biệt khỏi phần cứng của instance, cho nên network storage có thể vượt qua các sự cố khi EC2 host bị ảnh hưởng.
3: **Ephemeral storage**: Nó là một bộ nhớ tạm, không cần tồn tại lâu, là loại mà bạn không thể tin tưởng sự bền vững.
4: **Persistent storage**: Lưu trữ tồn tại như một thực thể riêng biệt, nó vẫn tồn tại ngay cả sau khi thiết bị nó gắn vào không còn nữa (instance).

Một ví dụ của Ephemeral storage là Instance Store, đây là ổ cứng vật lý gắn trực tiếp vào EC2 host, nó không bền vững.
Ngược lại là Persistent storage (lưu trữ bền vững) như EBS. Hãy nhớ kỹ điều này. Bạn sẽ gặp các câu hỏi kiểm tra kiến thức về việc phần biệt loại storage nào là Ephemeral storage và loại nào là persistent storage.