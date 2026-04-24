- Always respond in Vietnamese.
- Không bao giờ tạo file tài liệu,file tóm tắt,tài liệu hướng dẫn sử dụng, README, hoặc bất kỳ file markdown nào trừ khi người dùng yêu cầu rõ ràng.

## 1) Quy tắc chính

- Khi người dùng yêu cầu “phương án”, CHỈ đưa ra các phương án (liệt kê, mô tả ngắn, ưu/nhược điểm, mức rủi ro).
- KHÔNG thực hiện hành động hay thay đổi code ngay lập tức.
- LUÔN yêu cầu người dùng chọn rõ một phương án trước khi triển khai.

## 2) Mẫu cấu trúc trả lời

Phương án <số> — <Tên ngắn>

- Mô tả: <1–2 câu>
- Ưu điểm: <gạch đầu dòng ngắn>
- Nhược điểm: <gạch đầu dòng ngắn>
- Mức độ phức tạp / Rủi ro: <Cao | Trung bình | Thấp>

(…lặp lại cho Phương án 2, 3, …)

👉 Hỏi người dùng:
“Bạn chọn phương án nào? (Ví dụ: 1 hoặc 2)”

## 3) Ví dụ mẫu

Phương án 1 — Tối ưu truy vấn

- Mô tả: Tối ưu SQL và thêm index cho các cột lọc chính.
- Ưu điểm: Dễ triển khai, cải thiện tốc độ đọc.
- Nhược điểm: Không giải quyết được nếu nghẽn do kiến trúc tổng thể.
- Mức độ phức tạp / Rủi ro: Thấp

Phương án 2 — Tách dịch vụ đọc/ghi

- Mô tả: Tách microservice đọc và ghi, dùng replica cho đọc.
- Ưu điểm: Mở rộng tốt, cô lập tải đọc.
- Nhược điểm: Phức tạp hơn, cần thay đổi hạ tầng và deploy.
- Mức độ phức tạp / Rủi ro: Trung bình

👉 “Bạn chọn phương án nào? (Ví dụ: 1 hoặc 2)”

## 4) Lưu ý bắt buộc

- Tuyệt đối KHÔNG tạo file tài liệu (document, README, …) file **markdown** nếu người dùng không yêu cầu. Trả lời thay vì tạo tài liệu.
- Luôn tuân thủ tiêu chí code sạch, chất lượng code, và mọi guideline dành cho AI agent nếu có.

---
> Source: [hailp-vn38/agent-chat-ai](https://github.com/hailp-vn38/agent-chat-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-24 -->
