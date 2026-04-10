# AI Coding Assistant Instructions for Playwright Test Suite

> **역할**: Playwright + TypeScript 전문 SDET. 안정성과 가독성을 최우선으로 E2E 테스트 코드 생성.

## Project Architecture

**Multi-site E2E 테스트 스위트**: Makestar(팬 플랫폼), AlbumBuddy(구매 대행), Admin(stage 관리자)

### 핵심 구조
```
tests/
├── pages/           # Page Object Model (BasePage → AdminBasePage → 구체 페이지)
├── helpers/admin/   # 인증 및 공통 유틸리티
├── *_pom.spec.ts    # 실제 테스트 파일 (backup/ 제외)
auth.json            # Makestar/Admin 세션 (JWT 토큰)
ab-auth.json         # AlbumBuddy 세션
global-setup.js      # 테스트 전 토큰 유효성 검증
```

### 설정 파일 분리
- `playwright.config.js`: Admin/Makestar 테스트 (global-setup 포함, 토큰 필요)
- `playwright.albumbuddy.config.js`: AlbumBuddy 전용 (global-setup 없음)

## Critical Workflows

```bash
# 전체 테스트 (토큰 자동 검증)
npx playwright test

# AlbumBuddy만 실행 (로그인 불필요)
npx playwright test --config=playwright.albumbuddy.config.js

# 특정 테스트
npx playwright test tests/admin_test_pom.spec.ts

# 세션 저장 (토큰 만료 시)
npx playwright test tests/save-auth.spec.ts --headed
npx playwright test tests/ab-save-auth.spec.ts --headed  # AlbumBuddy용

# 디버그
npx playwright test --headed --debug
npx playwright test --ui  # Interactive 모드
```

## JWT 토큰 관리 (중요)

토큰은 **30분**마다 만료됨. `global-setup.js`가 자동 갱신 시도:
1. `admin-tokens.json` → `auth.json`의 `refresh_token` 쿠키 순으로 확인
2. 만료 시 브라우저 열어 Google 로그인 유도
3. ISMS 심사로 **하루 1회 이상 재로그인** 필요할 수 있음

```typescript
// 테스트에서 토큰 상태 확인 (admin_test_pom.spec.ts 참조)
import { setupAuthCookies } from './helpers/admin';
```

## Page Object Model 패턴 (엄격 준수)

### 관심사 분리
- `pages/`: 모든 셀렉터와 상호작용 로직 (Click, Fill 등)
- `tests/`: 비즈니스 로직과 검증만. **테스트 파일에서 `page.locator()` 직접 사용 금지**

### 상속 구조
`BasePage` → `AdminBasePage` → 구체 페이지 (SKUListPage, EventListPage 등)

```typescript
// tests/pages/index.ts에서 import
import { SKUListPage, EventListPage, AlbumBuddyPage } from './pages';

// 테스트에서 사용
const skuPage = new SKUListPage(page);
await skuPage.navigate();
await skuPage.search({ field: 'name', value: '테스트' });
```

### JSDoc 필수
Page Class의 모든 Public 메서드에 JSDoc 작성 (기능, 파라미터, 리턴값)

## 로케이터 전략 (우선순위 준수)

UI 변경에 강한 코드를 위해 아래 순서대로 선택:
1. `page.getByRole()` - **최우선 권장**
2. `page.getByLabel()` - 입력 폼
3. `page.getByPlaceholder()`
4. `page.getByText()` - 동적 텍스트 주의
5. `page.getByTestId()` - data-testid 있는 경우만

**금지**: XPath, 일반 CSS Selector (`div > span:nth-child(3)`) - 불가피 시 주석 필수

## 대기 전략 (Wait)

- **Hard Wait 금지**: `page.waitForTimeout()`, `makestar.wait()` 절대 사용 금지
- **Auto-waiting 활용**: Playwright 자동 대기 기능 신뢰
- **Web-first Assertions**: 자동 재시도 지원 단언문 사용
  ```typescript
  await expect(locator).toBeVisible();  // ✅ 권장
  // page.waitForTimeout(3000);         // ❌ 금지
  ```

### 조건부 대기 메서드 (BasePage 제공)
Hard wait 대신 아래 메서드들을 사용:
```typescript
// 요소 대기
await makestar.waitForElement(locator, { state: 'visible' });

// 콘텐츠 안정화 대기 (DOM 변경 완료)
await makestar.waitForContentStable();

// 네트워크 안정화 대기
await makestar.waitForNetworkStable();

// URL 변경 대기
await makestar.waitForUrlContains('search');

// 페이지 콘텐츠 로드 대기 (MakestarPage)
await makestar.waitForPageContent();

// 검색 결과 대기
await makestar.waitForSearchResults();

// 모달 처리 + 콘텐츠 안정화
await makestar.handleModalAndWaitForContent();
```

### 성능 측정 (Web Vitals)
```typescript
// Core Web Vitals 측정
const vitals = await makestar.measureWebVitals();
console.log(`LCP: ${vitals.lcp}ms, FCP: ${vitals.fcp}ms, CLS: ${vitals.cls}`);

// 페이지 로딩 시간 측정 (LCP 기반)
const { totalTime, vitals } = await makestar.measurePageLoadTime(url);
```

## 테스트 작성 규칙

1. **한글 테스트명**: `test('1-1 사이트 접근 가능 여부', ...)`
2. **Serial 그룹**: 인증 필요한 테스트는 `test.describe.serial()` 사용
3. **뷰포트 전략**:
   - **Admin 페이지**: Desktop 전용 (1920x1080), 모바일이면 즉시 Fail
   - **B2C 페이지**: Mobile + Desktop 모두 테스트
   ```typescript
   expect(viewport === null || viewport.width >= 1024, '데스크톱 전용').toBeTruthy();
   ```
4. **타임아웃 설정**: `tests/pages/base.page.ts`의 `DEFAULT_TIMEOUTS` 사용

## 인증 실패 처리

Worker 간 인증 실패 공유를 위해 `.auth-failed` 파일 사용:
```typescript
// 첫 번째 테스트 실패 시 즉시 Fail 처리
const { failed, reason } = isAuthFailed();
expect(failed, reason ?? '인증 실패').toBe(false);
```

## Self-Healing 워크플로우

테스트 실패 시 다음 순서로 대응:
1. **Context Check**: Admin vs User 판단 → 뷰포트 설정
2. **Interactive Debugging**: `--ui` 모드 또는 스크린샷 분석
3. **Iteration**: 에러 분석 → 코드 수정 → 통과까지 반복
4. **뷰포트 문제**: Admin에서 실패 시 뷰포트 넓혀서 재시도

## User Journey Navigation (사용자 시나리오 기반 네비게이션)

테스트에서 페이지 이동 시 **URL 직접 접근(`goto()`)보다 사용자 시나리오처럼 버튼/링크 클릭**을 우선한다.

### 1. 원칙
- **버튼 클릭 우선**: `navigateToShop()`, `navigateToEvent()` 등 POM 메서드로 GNB 버튼 클릭
- **URL 직접 접근 지양**: `goto('https://...')` 대신 사용자가 실제로 수행하는 클릭 액션 사용
- **폴백 허용**: 버튼을 찾지 못할 경우에만 URL 직접 이동 (로그에 경고 출력)

### 2. 적용 예시
```typescript
// ❌ 지양: URL 직접 접근
await makestar.goto('https://www.makestar.com/my-page/password');

// ✅ 권장: 사용자 시나리오 (버튼 클릭)
await makestar.navigateToPasswordPage();  // 마이페이지 → 비밀번호 변경 메뉴 클릭
```

### 3. 예외 상황
- **비회원 테스트**: 별도 컨텍스트에서 `goto()` 사용 (로그인 상태 불가)
- **성능 측정**: `measurePageLoadTime(url)` 등 직접 URL 필요한 경우

## Test Execution & Iteration Workflow (테스트 실행 및 반복 수정)

테스트 작성/수정 후 반드시 **실행 → 검증 → 수정** 사이클을 완료한다.

### 1. 실행 및 검증
```bash
# 전체 테스트 실행
npx playwright test tests/<파일명>.spec.ts --retries=0 --reporter=list

# 특정 테스트만 실행
npx playwright test --grep='테스트명' --retries=0
```

### 2. 실패 시 대응 (Self-Healing Loop)
1. **에러 로그 분석**: 타임아웃, 셀렉터 미검출, Assertion 실패 등 원인 파악
2. **코드 수정**: POM 메서드 또는 테스트 로직 개선
3. **재실행**: 수정 후 즉시 재실행하여 검증
4. **반복**: 모든 테스트 통과까지 위 과정 반복

### 3. 타임아웃 처리 가이드
- `waitForContentStable()`: 타임아웃 시 `.catch(() => {})` 추가하여 진행
- 네트워크 호출: `.catch()` 처리로 불필요한 대기 방지
```typescript
// 타임아웃 시 무시하고 계속 진행
await makestar.waitForContentStable('body', { stableTime: 500, timeout: 3000 }).catch(() => {});
```

## Test Integrity & Anti-Skip Policy (중요)

에이전트는 테스트 실패 시 임의로 코드를 수정하여 상황을 회피하거나 스킵해서는 안 된다.

### 1. 임의 Skip 금지 규칙
- **No Silent Skips**: 테스트 실패, 타임아웃, 셀렉터 미검출 시 `test.skip()`이나 `test.fixme()`를 추가하여 통과시키는 행위를 엄격히 금지한다.
- **No Legacy Skips**: 기존 코드에 `test.skip()`/`test.fixme()`가 있어도 그대로 두지 말고, 실패 원인이 드러나도록 `expect`/`throw` 기반 Fail 처리로 전환한다.
- **Fail-Fast & Report**: 실패 시 코드를 수정하기 전, 먼저 실패 원인(Assertion Error, Timeout, Locator Mismatch 등)을 분석하여 사용자에게 보고한다.
- **Exception handling**: 네트워크 순변동 등으로 인한 일시적 오류는 Playwright의 `retry` 설정을 제안하되, 코드 자체에 무분별한 `try-catch`로 에러를 은닉하지 않는다.

### 2. 에러 분석 및 수정 워크플로우
1. **분석**: Trace Viewer나 Log를 통해 에러가 발생한 정확한 라인과 원인을 파악한다.
2. **보고**: 수정 전 "OOO 이유로 실패했습니다. 로케이터를 XXX로 수정해도 될까요?"라고 먼저 확인한다.
3. **처리 원칙**: 환경 문제/셀렉터 문제/데이터 문제 모두 `skip`이 아니라 `fail`로 노출하고 원인을 추적한다.

## Troubleshooting

| 증상 | 원인 | 해결 |
|------|------|------|
| Admin 테스트 모바일에서 실패 | 사이드바가 콘텐츠 가림 | 뷰포트 조건 `expect`로 즉시 Fail 후 데스크톱 환경에서 재실행 |
| 401 Unauthorized | JWT 만료 (30분) | `npx playwright test tests/save-auth.spec.ts --headed` |
| `.auth-failed` 파일 존재 | 이전 인증 실패 | 파일 삭제 후 재실행 |
| AlbumBuddy 세션 만료 | ab-auth.json 쿠키 만료 | `npx playwright test tests/ab-save-auth.spec.ts --headed` |
| 테이블 로딩 타임아웃 | 네트워크 지연 | `ADMIN_TIMEOUTS.long` (10초) 사용 |
| Flaky 테스트 | 불안정한 셀렉터 | `getByRole()` 우선, XPath 제거 |

## Key Files Reference
- `tests/pages/index.ts`: POM 모듈 export (여기서 import)
- `tests/helpers/admin/index.ts`: 인증 헬퍼
- `global-setup.js`: 토큰 자동 갱신 로직
- `auto-refresh-token.js`: 세션 관리 유틸리티

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dykim0518)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dykim0518)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
