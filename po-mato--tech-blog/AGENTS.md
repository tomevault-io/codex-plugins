# Copilot / AI 에이전트 작업 가이드 (tech-blog)

## 프로젝트 개요
- Vite + React(19) + TypeScript(Strict) SPA 입니다. 진입점은 `index.html` → `src/index.tsx` 입니다.
- 라우팅은 React Router로 `src/routes/index.tsx`에서 정의하고, 좌측 네비게이션은 `src/components/NavigationBar.tsx` 입니다.
- 홈 배경/메인 비주얼은 Three.js로 만든 `src/components/Universe.tsx`(캔버스 DOM 직접 부착)입니다.

## 폴더/구조에서 길 잃지 않기
- 페이지: `src/pages/{Home,Game,Chat}.tsx`
- 라우터: `src/routes/index.tsx` (`/`, `/game`, `/chat`)
- 공통 UI: `src/components/*`
- 정적 리소스: `public/*` (예: `Universe.tsx`는 `/smoke.png`를 절대경로로 로드)
- 경로 별칭: `@` = `src` (`vite.config.ts`, `tsconfig.json`의 paths)

## 개발/빌드 워크플로
- 패키지 매니저는 `pnpm-lock.yaml`이 있으니 기본은 pnpm 사용을 권장합니다.
- 주요 스크립트는 `package.json`의 `dev`, `build`, `preview` 입니다.
- 주의: `build`가 현재 `vue-tsc && vite build`로 되어 있어(React 프로젝트인데) CI/로컬에서 실패할 수 있습니다. 빌드가 깨지면 **먼저 `package.json` 스크립트와 타입체크 도구**를 확인하세요.

## 배포/CI 메모
- `deploy.yaml`은 GitHub Pages 배포 플로우를 포함하며, `npm install`/`npm run build`로 `dist/`를 퍼블리시합니다.
- Node 버전은 워크플로에서 18을 사용합니다(로컬도 Node 18+ 기준으로 맞추는 게 안전).

## 이 코드베이스에서 자주 하는 변경 패턴
- 새 페이지 추가: `src/pages/NewPage.tsx` 생성 → `src/routes/index.tsx`에 `<Route>` 추가 → 필요 시 `NavigationBar.tsx`에 `<Link>` 추가.
- Three.js/캔버스 변경: `Universe.tsx`는 `#universe` 컨테이너에 `renderer.domElement`를 append합니다. **반드시 effect cleanup에서 canvas 제거/renderer dispose** 패턴을 유지하세요.

## 스타일/포맷 관찰
- SCSS 사용: 레이아웃은 `src/global.scss`에서 `#root`를 2열 grid(좌측 200px)로 잡습니다.
- 파일별로 따옴표/들여쓰기 스타일이 섞여 있습니다(예: `Universe.tsx`는 double quotes, `NavigationBar.tsx`는 single quotes). 수정 시 **해당 파일의 기존 스타일을 우선** 따라가세요.

## 커밋 메시지 지침(Conventional Commits)
- 커밋 메시지는 기본적으로 Conventional Commits 포맷을 사용합니다:
	- `<type>(<scope>): <subject>`
- **subject/body는 한국어 우선**(간결한 서술/명령형)으로 작성(72자 이내 권장)
- type/scope는 관례대로 영문 키워드 사용(예: `feat`, `fix`, `refactor`)
- scope는 변경 범위가 명확할 때만 사용(예: `app`, `posts`, `components`, `universe`, `deps`, `config`, `ci`)
- Breaking change가 있으면 `!` 또는 `BREAKING CHANGE:` footer로 명시
- 상세 규칙/예시는 `.github/commit-message-guidelines.md`를 따르세요.

## AI 관련 설정(있다면)
- bmad 에이전트/워크플로 파일은 `.github/agents/`에 있습니다.
- bmad 커스텀 설정은 `bmad-custom-src/custom.yaml`을 참고하세요.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Po-Mato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
