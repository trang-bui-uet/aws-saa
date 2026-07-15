**Chào mừng đến với bài học này**, nơi tôi muốn cung cấp một phần giới thiệu lý thuyết ngắn gọn về **ACID** và **BASE** – hai mô hình giao dịch cơ sở dữ liệu mà bạn có thể gặp trong kỳ thi cũng như trong thực tế. Có thể nghe hơi trừu tượng, nhưng nó xuất hiện trong kỳ thi, và tôi hứa rằng trong thực tế, việc nắm vững kiến thức này chính là một **siêu năng lực về cơ sở dữ liệu**. Hãy bắt đầu ngay.

**ACID** và **BASE** đều là các từ viết tắt, và tôi sẽ giải thích ý nghĩa của chúng ngay sau đây, nhưng cả hai đều là **mô hình giao dịch cơ sở dữ liệu**. Chúng định nghĩa một số khía cạnh về các giao dịch đến và từ cơ sở dữ liệu, đồng thời chi phối cách hệ thống cơ sở dữ liệu được thiết kế.

Ở mức nền tảng nhất, có một định lý khoa học máy tính gọi là **định lý CAP**, viết tắt của **Consistency** (Tính nhất quán), **Availability** (Tính sẵn sàng) và **Partition tolerance** (Khả năng chịu phân vùng). Hãy cùng khám phá nhanh từng khái niệm vì chúng thực sự quan trọng.

- **Consistency** (Tính nhất quán) nghĩa là mọi lần đọc từ cơ sở dữ liệu sẽ nhận được lần ghi mới nhất, hoặc sẽ nhận được lỗi.
- Ngược lại, **Availability** (Tính sẵn sàng) nghĩa là mọi yêu cầu sẽ nhận được phản hồi không lỗi, nhưng **không đảm bảo** rằng phản hồi đó chứa lần ghi mới nhất – điều này rất quan trọng.
- **Partition tolerance** (Khả năng chịu phân vùng) nghĩa là hệ thống có thể được tạo thành từ nhiều phân vùng mạng, và hệ thống vẫn tiếp tục hoạt động ngay cả khi có một số thông điệp bị mất hoặc lỗi giữa các nút mạng.

**Định lý CAP** nêu rõ rằng bất kỳ sản phẩm cơ sở dữ liệu nào cũng chỉ có khả năng cung cấp tối đa **hai trong ba yếu tố** này. Một lý do là nếu bạn tưởng tượng một cơ sở dữ liệu với nhiều nút khác nhau, tất cả đều nằm trên mạng. Hãy tưởng tượng nếu giao tiếp bị lỗi giữa một số nút hoặc nếu bất kỳ nút nào bị lỗi. Khi đó, bạn có hai lựa chọn nếu ai đó đọc từ cơ sở dữ liệu đó:

- Hủy bỏ thao tác → giảm tính sẵn sàng nhưng đảm bảo tính nhất quán,
- hoặc tiếp tục thao tác → tăng tính sẵn sàng nhưng rủi ro về tính nhất quán.

Vì vậy, như tôi vừa đề cập, người ta thường coi là **không thể** xây dựng một nền tảng cơ sở dữ liệu cung cấp nhiều hơn hai trong ba yếu tố này. Nếu bạn có hệ thống cơ sở dữ liệu nhiều nút và có mạng liên quan, thì nói chung bạn phải chọn giữa **tính nhất quán** hoặc **tính sẵn sàng**, và các mô hình giao dịch **ACID** và **BASE** lựa chọn các sự đánh đổi khác nhau. **ACID tập trung vào tính nhất quán**, còn **BASE tập trung vào tính sẵn sàng**.

Có một số sắc thái và chi tiết bổ sung, nhưng đây chỉ là phần giới thiệu ở mức cao. Tôi chỉ đề cập những gì cần thiết cho kỳ thi. Hãy cùng bước nhanh qua các sự đánh đổi của từng mô hình, bắt đầu với **ACID**.

**ACID** nghĩa là các giao dịch là:

- **Atomic** (Nguyên tử)
- **Consistent** (Nhất quán)
- **Isolated** (Cô lập)
- và cuối cùng là **Durable** (Bền vững)

Và hãy giải quyết ngay phần “sức mạnh kỳ thi”: Nói chung, nếu bạn thấy nhắc đến **ACID**, thì rất có thể đang đề cập đến bất kỳ cơ sở dữ liệu **RDS** nào – chúng thường dựa trên **ACID**. Và **ACID hạn chế khả năng mở rộng** của cơ sở dữ liệu. Tôi sẽ giải thích một số lý do. Tôi sẽ giữ ở mức cao, nhưng đã đính kèm một số liên kết nếu bạn muốn đọc thêm. Trong bài học này, tôi chỉ tập trung vào những gì **cực kỳ quan trọng** cho kỳ thi.

Hãy xem xét từng thành phần:

- **Atomic** (Nguyên tử) nghĩa là đối với một giao dịch, **tất cả các phần** của giao dịch đều thành công, hoặc **không có phần nào** thành công. Hãy tưởng tượng bạn điều hành một ngân hàng và muốn chuyển 10 đô từ Tài khoản A sang Tài khoản B. Giao dịch đó có hai phần: phần một trừ 10 đô khỏi Tài khoản A, phần hai cộng 10 đô vào Tài khoản B. Bạn không muốn tình huống phần thứ nhất hoặc thứ hai thành công một mình trong khi phần kia thất bại. **Hoặc cả hai phần đều thành công, hoặc không phần nào được áp dụng** – đó chính là ý nghĩa của Atomic.
- **Consistent** (Nhất quán) nghĩa là các giao dịch được áp dụng lên cơ sở dữ liệu sẽ đưa cơ sở dữ liệu từ **một trạng thái hợp lệ** sang **một trạng thái hợp lệ khác**. Không được phép có trạng thái trung gian. Trong các cơ sở dữ liệu quan hệ, thường có liên kết giữa các bảng – một mục trong bảng này phải có mục tương ứng trong bảng kia, hoặc giá trị phải nằm trong một khoảng nhất định. Thành phần này đơn giản nghĩa là mọi giao dịch phải đưa cơ sở dữ liệu từ trạng thái hợp lệ này sang trạng thái hợp lệ khác theo đúng các quy tắc của cơ sở dữ liệu đó.
- **Isolated** (Cô lập) nghĩa là vì các giao dịch thường được thực thi song song, chúng **không được can thiệp lẫn nhau**. Isolation đảm bảo rằng việc thực thi đồng thời các giao dịch sẽ để lại cơ sở dữ liệu ở cùng trạng thái như thể các giao dịch được thực thi tuần tự. Điều này rất quan trọng để cơ sở dữ liệu có thể chạy nhiều giao dịch khác nhau cùng lúc (từ các ứng dụng hoặc người dùng khác nhau). Mỗi giao dịch cần được thực thi đầy đủ như thể nó là giao dịch duy nhất đang chạy trên cơ sở dữ liệu đó – chúng không được can thiệp lẫn nhau.
- Cuối cùng là **Durable** (Bền vững), nghĩa là một khi giao dịch đã được **commit**, nó sẽ **vẫn được commit** ngay cả trong trường hợp hệ thống gặp sự cố. Một khi cơ sở dữ liệu báo cho ứng dụng rằng giao dịch đã hoàn tất và thành công, dữ liệu đó được lưu trữ ở nơi mà sự cố hệ thống, mất điện hoặc khởi động lại máy chủ/nút cơ sở dữ liệu sẽ **không ảnh hưởng** đến dữ liệu.

Hầu hết các nền tảng cơ sở dữ liệu quan hệ sử dụng giao dịch dựa trên **ACID**. Đó là lý do các tổ chức tài chính thường sử dụng chúng, vì nó áp dụng một hình thức quản lý dữ liệu và giao dịch rất chặt chẽ. Nhưng chính vì những quy tắc cứng nhắc này, nó **hạn chế khả năng mở rộng**.

Tiếp theo là **BASE**, và **BASE** viết tắt của:

- **Basically Available** (Về cơ bản sẵn sàng)
- **Soft state** (Trạng thái mềm)
- **Eventually consistent** (Nhất quán cuối cùng)

Một lần nữa, đây là mức rất cao, và tôi đã đính kèm một số liên kết để bạn đọc thêm. Nghe có vẻ như tôi đang chế giễu mô hình giao dịch này vì một số khái niệm nghe khá kỳ lạ, nhưng hãy kiên nhẫn, tôi sẽ giải thích tất cả.

- **Basically Available** nghĩa là các thao tác đọc và ghi được cung cấp **càng nhiều càng tốt**, nhưng **không có bất kỳ đảm bảo nhất quán nào**. Vì vậy đọc và ghi là kiểu “có thể” hoặc “có lẽ”. Về cơ bản, thay vì ép buộc tính nhất quán ngay lập tức, các cơ sở dữ liệu NoSQL theo mô hình BASE sẽ đảm bảo tính sẵn sàng của dữ liệu bằng cách **phân tán và nhân bản** dữ liệu trên tất cả các nút khác nhau của cơ sở dữ liệu. Trong cơ sở dữ liệu không thực sự có mục tiêu đảm bảo bất cứ điều gì về tính nhất quán – nó cố gắng hết sức để nhất quán, nhưng **không có đảm bảo**.
- **Soft State** là một khái niệm nghe hơi buồn cười một chút. Nó nghĩa là BASE **từ bỏ** khái niệm cơ sở dữ liệu tự ép buộc tính nhất quán của chính nó. Thay vào đó, nó **ủy thác trách nhiệm đó cho nhà phát triển**. Ứng dụng của bạn cần nhận thức được tính nhất quán và trạng thái, rồi làm việc xung quanh cơ sở dữ liệu. Nếu bạn cần tính nhất quán ngay lập tức – nghĩa là một thao tác đọc luôn có quyền truy cập vào tất cả các lần ghi trước đó ngay lập tức – và nếu cơ sở dữ liệu cho phép tùy chọn, thì ứng dụng của bạn cần **yêu cầu rõ ràng**. Nếu không, ứng dụng phải chấp nhận rằng những gì nó đọc có thể **không phải** là những gì một phiên bản khác của ứng dụng đã ghi trước đó. Vì vậy với cơ sở dữ liệu soft state, ứng dụng của bạn cần xử lý khả năng dữ liệu đang đọc **không giống** dữ liệu đã được ghi chỉ vài khoảnh khắc trước.
- Tất cả các khái niệm này khá mơ hồ và có sự chồng chéo, nhưng cuối cùng, **BASE không ép buộc tính nhất quán ngay lập tức**. Nó nghĩa là tính nhất quán **có thể xảy ra… cuối cùng**. Nếu chúng ta chờ đủ lâu, thì những gì chúng ta đọc sẽ khớp với những gì đã được ghi trước đó – **cuối cùng**.

Điều này quan trọng cần hiểu vì theo mặc định, mô hình giao dịch BASE nghĩa là mọi lần đọc từ cơ sở dữ liệu đều là **eventually consistent**. Vì vậy ứng dụng cần chấp nhận rằng lần đọc có thể **không luôn** chứa dữ liệu của các lần ghi trước đó. Nhiều cơ sở dữ liệu có khả năng cung cấp cả đọc eventually consistent và immediately consistent, nhưng ứng dụng phải nhận thức được điều này và **yêu cầu rõ ràng** cơ sở dữ liệu thực hiện đọc nhất quán.

Nghe có vẻ như giao dịch BASE khá tệ, đúng không? Thực ra thì **không**. Các cơ sở dữ liệu sử dụng BASE thực sự **có khả năng mở rộng rất cao** và có thể mang lại hiệu suất cực kỳ tốt, vì chúng không phải lo lắng về tất cả những thứ phiền phức như tính nhất quán bên trong cơ sở dữ liệu – chúng **chuyển trách nhiệm đó sang cho ứng dụng**.

**DynamoDB** trong AWS là một ví dụ về cơ sở dữ liệu thường hoạt động theo kiểu **BASE**. Nó cung cấp cả đọc eventually consistent và immediately consistent, nhưng ứng dụng của bạn phải nhận thức được điều đó. Ngoài ra, DynamoDB còn cung cấp một số tính năng bổ sung mang lại chức năng **ACID**, chẳng hạn như **DynamoDB Transactions** – đây cũng là điều cần ghi nhớ.

Đối với kỳ thi cụ thể, tôi có một số **mặc định hữu ích**:

- Nếu bạn thấy thuật ngữ **BASE**, bạn có thể an toàn giả định rằng nó đang đề cập đến cơ sở dữ liệu kiểu **NoSQL**.
- Nếu bạn thấy thuật ngữ **ACID**, bạn có thể an toàn giả định mặc định rằng nó đang đề cập đến cơ sở dữ liệu **RDS**.
- Nhưng nếu bạn thấy **NoSQL** hoặc **DynamoDB** được đề cập cùng với **ACID**, thì có thể đang nói đến **DynamoDB Transactions** – hãy ghi nhớ điều này.

Đó là tất cả những gì tôi muốn đề cập trong bài học mức cao này về các mô hình giao dịch khác nhau. Chủ đề này tương đối lý thuyết và khá sâu, và có rất nhiều tài liệu đọc thêm. Nhưng tôi chỉ muốn đề cập những điều thiết yếu mà bạn cần cho kỳ thi, và tôi đã bao phủ tất cả các sự thật đó trong bài học này. Đến đây là kết thúc bài học. Cảm ơn bạn đã theo dõi, hãy hoàn thành video, và khi sẵn sàng, tôi mong được gặp bạn ở bài học tiếp theo.

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- Định lý CAP (Consistency – Availability – Partition tolerance)
- ACID vs BASE: sự đánh đổi Consistency vs Availability
- ACID = Atomic + Consistent + Isolated + Durable
- BASE = Basically Available + Soft state + Eventually consistent
- RDS → ACID (hạn chế mở rộng)
- DynamoDB → BASE (có thể mở rộng cao) + hỗ trợ Transactions (ACID)
- Mẹo thi: BASE = NoSQL, ACID = RDS, DynamoDB + ACID = Transactions

**Notes (Chi tiết):**

- **Định lý CAP**: Cơ sở dữ liệu chỉ có thể đảm bảo tối đa **2/3** yếu tố. Khi có mạng/phân vùng, phải chọn giữa **Consistency** hoặc **Availability**.
- **ACID** ưu tiên Consistency:
    - **Atomic**: Tất cả hoặc không có gì (ví dụ chuyển tiền ngân hàng).
    - **Consistent**: Đưa DB từ trạng thái hợp lệ này sang trạng thái hợp lệ khác.
    - **Isolated**: Các giao dịch song song không can thiệp lẫn nhau.
    - **Durable**: Sau khi commit thì dữ liệu không mất dù hệ thống lỗi.
- **BASE** ưu tiên Availability:
    - **Basically Available**: Đọc/ghi càng nhiều càng tốt, không đảm bảo consistency.
    - **Soft state**: DB không tự ép consistency → ứng dụng phải xử lý.
    - **Eventually consistent**: Dữ liệu sẽ nhất quán… cuối cùng.
- DynamoDB mặc định hoạt động theo kiểu BASE nhưng hỗ trợ **DynamoDB Transactions** để có tính năng ACID.

**Summary (Tóm tắt ngắn):** Bài học giới thiệu hai mô hình giao dịch **ACID** và **BASE** dựa trên **định lý CAP**. **ACID** (dùng trong RDS) đảm bảo tính nhất quán cao nhưng hạn chế khả năng mở rộng. **BASE** (dùng trong NoSQL/DynamoDB) ưu tiên tính sẵn sàng và khả năng mở rộng, chấp nhận tính nhất quán cuối cùng. Khi thi, hãy nhớ: BASE = NoSQL, ACID = RDS, và DynamoDB có thể hỗ trợ cả hai thông qua Transactions.