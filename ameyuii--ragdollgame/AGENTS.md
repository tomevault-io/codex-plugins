# GitHub Copilot Custom Instructions

## Response Guidelines
- CHỈ trả lời đúng vào câu hỏi được đặt ra
- KHÔNG thực hiện các tác vụ ngoài yêu cầu
- KHÔNG thêm thông tin không cần thiết
- Hỏi xác nhận trước khi thực hiện bất kỳ thay đổi nào
- Không tạo các script new khi đã có 1 script cũ tương tự có thể sửa, sẽ làm lỗi duplicate
- Không tạo các chức năng mới tương tự chức năng đã có với cách code khác và ở nơi khác, phải hỏi lại trước khi làm điều này

## Git Safety Rules
- **TUYỆT ĐỐI KHÔNG** thực hiện bất kỳ lệnh git nào mà không được xác nhận trước
- **PHẢI HỎI** user trước khi chạy: git restore, git reset, git stash, git checkout, git merge, git rebase
- **PHẢI HỎI** user trước khi xóa/thay đổi file quan trọng
- **PHẢI GIẢI THÍCH** rõ ràng hậu quả của lệnh git trước khi thực hiện
- Ưu tiên sử dụng git status, git log để kiểm tra trước khi thao tác

## Project Context
- Unity 3D project với AI NPC system
- Sử dụng C# và Unity Engine
- Git repository: https://github.com/Ameyuii/ragdollgame

## Coding Standards
- Sử dụng Unity coding conventions
- Comment bằng tiếng Việt cho dễ hiểu
- Luôn test code trước khi commit

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ameyuii)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Ameyuii)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
