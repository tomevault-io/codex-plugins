
# Generating Test Data with AI

Khi cần sinh test data bằng AI, ĐỌC và LÀM THEO nguyên văn skill đầy đủ tại
`.claude/skills/generating-test-data/SKILL.md`.

Nguyên tắc cốt lõi: AI sinh dữ liệu trông thật rất nhanh, nhưng mặc định chỉ ra giá trị đẹp/happy-path,
dễ sai lệch schema mà không báo, và dễ bị cám dỗ dùng dữ liệu prod thật cho "giống thật". Dữ liệu test
tốt phải phủ đủ nhóm hợp lệ/biên/bất hợp lệ/unicode/bulk, được kiểm chứng đúng schema trước khi tin, và
tuyệt đối 100% synthetic, không chứa PII hay dữ liệu khách hàng thật.

---
> Source: [huyphan13189/QE-AI](https://github.com/huyphan13189/QE-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-03 -->
