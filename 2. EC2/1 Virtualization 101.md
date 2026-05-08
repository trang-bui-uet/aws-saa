## Giới thiệu về ảo hóa
EC2 cung cấp ảo hóa như là một dịch vụ, hay còn gọi là Infrastructure as Service hay Iaas.
Ảo hóa (Virtualization):  Nghĩa là chạy nhiều hệ điều hành trên một thiết bị phần cứng.
Kernel là software trương tác trực tiếp với hardware.
Hệ điều hành và kernel chạy ở chế độ privileged mode (toàn quyền)
Application (ứng dụng) chạy ở chế độ user mode (hạn chế)

Không có ảo hóa thì 1 phần cứng chạy 1 hệ điều hành
Với ảo hóa thì 1 phần cứng sẽ chạy nhiều hệ điều hành

| Ảo hóa<br>![[Pasted image 20260508094948.png]] | Bình thường<br>![[Pasted image 20260508094910.png]] |
| ---------------------------------------------- | --------------------------------------------------- |

## Các cách ảo hóa

- Emulated Virtualization
- Paravirtualization
- Hardware Assisted Virtualization
- SR-IOV

**Giả lập ảo hóa (Emulated Virtualization)**
Có 1 phần mềm Host operating system / Hypervisor chạy ở privileged mode tương tác trực tiếp với phần cứng.
Application và OS đóng gói lại thành các Virtual Machine (Máy ảo)
Mỗi Virtual Machine được cấp phát ảo tài nguyên Bộ nhớ, CPU, RAM...
Hệ điều hành của VM tin rằng chúng đang tương tác trực tiếp vs Memory, Disk nhưng thực tế không phải, chugns chỉ đang tương tác với một phần Memory, Disk được cấp phát bở Hypervisor.
Hypervisor có khả năng cách ly bộ nhớ và tài nguyên. Để các máy ảo không ghi đè lên bộ nhớ và ổ cứng của nhau.
![[Pasted image 20260508101500.png]]

Bất kỳ hành động privileged mode nào mà máy ảo cố gắng thực hiện đều bị hypervisor chặn lại và bên dịch ngay lập tức.
Điều này có nghĩa là hệ điều hành của VM không cần sửa đổi gì, nhưng hiệu suất rất chậm. Thập chí có thể chỉ bằng 1 nửa so với chạy thật.
Ảo hóa giả lập (Emulated Virtualization) từng là một tính năng quan trọng vời thời của nó, nhưng chưa bao giờ đủ nhanh để áp dụng cho các workload đòi hỏi cao về hiệu suất.

**Paravirtualization**
Tuy nhiên, còn một cách ảo hóa khách là Para - Virtualization (Áo hóa bán ảo)
Thay vì dùng Binary Tranlsation, hpervisor chọn cách tiếp cận khác.
Para - Virtualization chỉ hoạt động với một số ít hệ điều hành có thể chỉnh sửa mã nguồn.
Những hệ điều hành này sẽ được sửa đổi ở phần thực hiện privileged mode thay vì gọi trực tiếp phần cứng, chúng sẽ gọi hyper call đến hypervisor.
Nhờ đó hiệu suất tăng mạnh do không cần Binary Translation mà VM OS gọi trực tiếp đến Hypervisor.
Tuy nhiên, nó đòi hỏi phải chỉnh sửa hệ điều hành (Không còn generic nữa)![[Pasted image 20260508103054.png]]

**Hardware Assisted Virtualization**
Sự cải tiến lớn nhất trong công nghệ ảo hóa đến khi phần cứng vật lý bắt đầu ảo hóa.
Điều này cho phép xuất hiện Hardware Virtualization, còn được gọi là Hardware Assisted Virtualization (Ảo hóa được hỗ trợ bởi phần cứng)
Với Hardware Assisted Virtualization, bản thân phần cứng bắt đầu biết ảo hóa.
CPU chứa các lệnh và tính năng đặc biệt cho nên Hypervisor có thể trực tiếp kiểm soát và cấu hình việc này (Intel VT-x, AMD-v)
Do đó chính CPU biết rằng nó đang thực hiện ảo hóa. Nói cách khách CPU hiểu rằng ảo hóa đang tồn tại.
Điều này có nghĩa là Khi VM cố gắng thực hiện Privileged call, chúng sẽ bị CPU chặn lại, vì CPU biết những lệnh này đến từ VM, nhờ vậy toàn bộ hệ thống không bị treo.
Tuy nhiên những lệnh này không thể thực thi trực tiếp vì VM OS vẫn nghĩ chúng đang chạy thẳng trên phần cứng thật.
Do đó chúng được chuyển hướng đến Hypervisor bởi phần cứng, và Hypervisor sẽ xử lý cách thực thi chúng (CPU chuyển cho Hypervisor)
Kết quả là hiệu suất giảm rất ít so với chạy trực tiếp hệ điều hành trên phần cứng.
Đây chính là bước **cải tiến lớn nhất** so với:
- Emulated Virtualization: giảm 50% hoặc hơn.
- Para-Virtualization: nhanh nhưng phải sửa OS.
Tuy nhiên vẫn còn vấn đề: Mặc dù phương pháp này giúp ích rất nhiều nhưng thứ quan trọng nhất đối với một máy ảo thường là cái hoạt động I/O cụ thể là truyền dữ liệu mạng và đọc ghi ổ cứng.
Các máy ảo nghĩ rằng chúng có phần cứng thật (card mạng thật) nhưng thật ra đó chỉ là thiết bị logic, sử dụng driver để kết nối ngược về một phần cứng duy nhất trên máy chủ.
Trừ khi mỗi máy ảo có 1 card mạng riêng, thì luôn luôn có lớp phần mềm ở giữa.
Và khi thực hiện các hoạt động transaction cáo như Network IO hay Disk IO điều này thường ảnh hưởng lớn đến hiệu suất, đồng thời tốn rất nhiều CPU của máy chủ.
![[Pasted image 20260508110621.png]]

**SR-IOV** 
cho phép một card mạng vật lý “chia nhỏ” thành nhiều card ảo độc lập, mỗi card ảo được gán trực tiếp cho một máy ảo. Máy ảo dùng card này gần như như dùng card thật → hiệu suất gần native, độ trễ cực thấp, ít tốn CPU host
![[Pasted image 20260508111719.png]]
