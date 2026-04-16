When performing a code review, respond in Korean.

# AIoIA Core 개발 가이드

## 코드 품질

- DRY: 공통 패턴 중복 금지
- 제네릭: 타입 안정성 유지
- 최소 의존성: 필수 패키지만 포함
- 범용성: 특정 프로젝트 규칙 하드코딩 지양
- Guard Clause: 함수 초반에 전제조건 검사, 실패 시 즉시 반환

## 보안

- SQL Injection: ORM 사용
- API 키: 환경 변수 관리
- 타입 검증: Pydantic, Zod

## 테스트

- TDD: Red → Green → Refactor
- 모든 public API 테스트 커버리지

## 코드 수정 후 필수 실행

### Python
Python 코드를 수정한 후에는 **반드시** `python/` 디렉토리에서 다음 명령을 실행하세요:
1. `make format` - 코드 포맷팅 (isort, black)
2. `make lint` - 린트 검사 (pylint)
3. `make code-style` - 코드 스타일 검증
4. `make type-check` - 타입 검사 (mypy, pyright)
5. `make test` - 테스트 실행

### TypeScript
TypeScript 코드를 수정한 후에는 **반드시** `typescript/` 디렉토리에서 다음 명령을 실행하세요:
1. `npm run lint` - ESLint 검사
2. `npm run check-format` - Prettier 코드 포맷 검사
3. `npm run type-check` - TypeScript 타입 검사
4. `npm test` - 테스트 실행

## Python

- SQLAlchemy Enum: `name="snake_case"` 명시
- Tests: `assert <var> is not None` 로 타입 안정성 유지

## 네이밍

- Python: `snake_case.py`, `PascalCase`, `snake_case()`
- TypeScript: `PascalCase.ts`, `camelCase()`, `index.ts`

## 릴리즈

- 자동화: main push 시 Conventional Commits 규약에 따라 semantic-release로 자동 버전 관리 및 배포

## 외부 기여

- Atomic PR: 하나의 PR은 하나의 논리적 변경만 포함
- 최소 변경: 안정성 최우선
- 하위 호환성: Breaking change 최소화
- 테스트 필수: 모든 PR에 테스트 포함
- PR 템플릿: `.github/PULL_REQUEST_TEMPLATE.md` 참고

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/algorima) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-14 -->
