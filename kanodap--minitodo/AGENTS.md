# Copilot 전역 지침

## Copilot 언어 지침

- 모든 대화, 응답, 문서 작성은 한국어로 진행해주세요.

## Copilot Markdown 작성 지침서

### 목적

Copilot은 모든 `*.md` 문서 작성 시 Markdownlint 규칙을 준수해야 한다.

### 기본 규칙

1. **헤더**

   - 문서 첫 줄은 `#` 헤더로 시작 (MD041)
   - 중복 헤더 금지 (MD024)
   - 헤더 끝 문장부호 금지 (MD026)
   - 헤더 전후 한 줄 공백 (MD022)

2. **코드 블록**

   - 언어 태그 명시 (MD040)
   - 코드펜스 전후 한 줄 공백 (MD031)
   - 탭(`\t`) 대신 4칸 공백 (MD010)

3. **리스트**

   - 인덴트 공백 2칸 (MD007)
   - 리스트 전후 한 줄 공백 (MD032)

4. **텍스트 및 인덴트**

   - 모든 들여쓰기는 공백 사용 (MD010)
   - 줄 끝 공백 제거
   - 중복 내용·잔재 코드 금지
   - UTF-8 인코딩 유지

5. **파일 구조**
   - 문서 중복 삽입 금지
   - 코드 블록 내부에 본문 포함 금지
   - 문서 끝 불필요한 빈 줄 제거

### 금지 사항

- 탭(`\t`) 사용
- 중복 헤더·섹션
- 언어 없는 코드 블록
- 코드펜스 전후 공백 누락
- 문서 전체를 코드 블록 안에 포함
- 불필요한 잔여 코드(`return item`, `TODO` 등)

### 참조 규칙

| 코드  | 의미                    |
| ----- | ----------------------- |
| MD010 | no-hard-tabs            |
| MD022 | blanks-around-headings  |
| MD024 | no-duplicate-heading    |
| MD026 | no-trailing-punctuation |
| MD031 | blanks-around-fences    |
| MD032 | blanks-around-lists     |
| MD040 | fenced-code-language    |
| MD007 | ul-indent               |
| MD041 | first-line-heading      |

---
> Source: [kanodap/minitodo](https://github.com/kanodap/minitodo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-25 -->
