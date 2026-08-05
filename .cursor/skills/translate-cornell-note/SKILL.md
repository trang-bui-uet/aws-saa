---
name: translate-cornell-note
description: Dịch nội dung sang tiếng Việt (giữ đủ ý, dễ đọc, bôi đậm phần quan trọng) rồi thêm Cornell Note phía dưới. Dùng khi người dùng yêu cầu dịch bài học, tóm tắt theo Cornell, hoặc nhắc đến dịch tiếng Việt + Cornell Note.
---

# Dịch tiếng Việt + Cornell Note

## Khi nào dùng

Áp dụng khi người dùng yêu cầu:
- Dịch bài học / ghi chú / tài liệu sang tiếng Việt
- Tóm tắt theo Cornell Note
- Kết hợp dịch + Cornell Note

## Quy trình bắt buộc

1. Đọc toàn bộ nội dung nguồn.
2. Dịch sang **tiếng Việt**.
3. Ngay bên dưới bản dịch, thêm khối **Tóm tắt theo Cornell Note**.

Không bỏ bước nào. Không chỉ dịch mà thiếu Cornell Note. Không chỉ Cornell Note mà thiếu bản dịch.

## Quy tắc dịch

- **Không bỏ ý**: Giữ đủ ý, khái niệm, bước thực hành, tên dịch vụ/AWS resource, lệnh, và cảnh báo.
- **Dễ đọc**: Dùng heading, danh sách, đoạn ngắn; tách bước rõ ràng.
- **Bôi đậm nội dung quan trọng**: Chỉ bôi đậm thuật ngữ then chốt, hành động chính, tên tài nguyên, kết quả cần kiểm tra — không bôi đậm cả câu dài.
- Giữ nguyên tên kỹ thuật phổ biến khi cần (ví dụ: `EC2`, `CloudFormation`, `User Data`, `Base64`, `Public IP`).
- Có thể giữ thuật ngữ gốc trong ngoặc sau lần xuất hiện đầu nếu giúp hiểu nhanh.
- Không thêm kiến thức ngoài nguồn trừ khi người dùng yêu cầu giải thích thêm.

## Định dạng đầu ra

Luôn trả về theo cấu trúc sau:

```markdown
# [Tiêu đề bản dịch]

[Nội dung đã dịch — format rõ, bôi đậm phần quan trọng]

---

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- [ý chính 1]
- [ý chính 2]
- [ý chính 3]
- ...

**Notes (Chi tiết):**

- [chi tiết gắn với cues; bôi đậm thuật ngữ/hành động quan trọng]
- ...

**Summary (Tóm tắt ngắn):** [1–3 câu nắm trọn bài; bôi đậm điểm then chốt]
```

## Hướng dẫn viết Cornell Note

### Cues
- 4–8 gạch đầu dòng.
- Là từ khóa / ý chính / chủ đề nhớ nhanh — không viết đoạn dài.
- Ưu tiên khái niệm, so sánh, quy trình, cleanup, kết quả kiểm tra.

### Notes
- Chi tiết hóa cues: giải thích ngắn, bước làm, điều kiện, kết quả.
- Mỗi ý Notes nên gắn với ít nhất một Cue.
- Bôi đậm thuật ngữ và hành động quan trọng.

### Summary
- 1–3 câu.
- Trả lời: bài này dạy gì, làm bằng cách nào, kết thúc ở đâu (ví dụ cleanup / kiểm chứng).

## Ví dụ định dạng Cornell Note (dùng đúng mẫu này)

**Tóm tắt theo Cornell Note**

**Cues (Từ khóa / Ý chính):**

- User Data + Base64 trong CloudFormation
- One-click deployment với template bootstrap-CFN
- So sánh bootstrap thủ công vs CloudFormation
- Custom login banner & WordPress install
- Account cleanup: Terminate instance + Delete stacks

**Notes (Chi tiết):**

- **User Data** có thể dùng trong CloudFormation bằng cách mã hóa **Base64**.
- Template CloudFormation tự động launch EC2 + chạy script user data giống hệt cách thủ công.
- Sau khi stack **Create complete** → kiểm tra Public IP → truy cập WordPress và Instance Connect để xem banner.
- **Cleanup**: Terminate instance thủ công (a4l-manual-wordpress) → Delete cả 2 CloudFormation stack (bootstrap & bootstrap-CFN).

**Summary (Tóm tắt ngắn):** Bài học hướng dẫn cách **bootstrap EC2 instance** bằng **User Data** theo hai cách: **thủ công** và **qua CloudFormation template**. Sử dụng base64 để nhúng script tự động cài WordPress và custom banner. Kết thúc bằng quy trình dọn dẹp tài khoản để trả về trạng thái ban đầu.

## Checklist trước khi trả kết quả

- [ ] Đã dịch đủ ý, không cắt nội dung quan trọng
- [ ] Format dễ đọc (heading / list / đoạn ngắn)
- [ ] Đã bôi đậm nội dung quan trọng (không over-bold)
- [ ] Có đủ 3 phần Cornell: Cues, Notes, Summary
- [ ] Cornell Note nằm **bên dưới** bản dịch
- [ ] Nhãn phần đúng như mẫu: `**Tóm tắt theo Cornell Note**`, `**Cues (Từ khóa / Ý chính):**`, `**Notes (Chi tiết):**`, `**Summary (Tóm tắt ngắn):**`
