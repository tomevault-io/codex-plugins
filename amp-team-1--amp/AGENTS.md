# Copilot Instructions for AMP Todolist Expo Project

## 프로젝트 개요
- **Expo + React Native** 기반의 크로스 플랫폼 앱 (Android/iOS/Web)
- **FastAPI + SQLAlchemy** 기반의 Python 백엔드가 `/app` 폴더에 포함됨
- 프론트엔드와 백엔드가 같은 리포지토리에 공존하며, 인증/회원가입/라우팅 등 통합 관리

## 주요 폴더 구조
- `app/` : Expo 라우터 기반 프론트엔드 + FastAPI 백엔드 (Python)
  - `main.py` : FastAPI 진입점, CORS/인증/DB 연결
  - `models.py`, `schemas.py`, `crud.py`, `security.py`, `database.py` : 백엔드 핵심 모듈
  - `home.tsx`, `signup.tsx`, `(tabs)/index.tsx` 등 : 주요 화면 컴포넌트
- `components/` : 재사용 가능한 UI 컴포넌트 (ThemedView, ThemedText 등)
- `constants/theme.ts` : 테마/폰트 정의, 플랫폼별 폰트 처리
- `assets/images/` : 앱 아이콘 및 이미지
- `scripts/reset-project.js` : 프로젝트 초기화 스크립트

## 라우팅 및 화면 구조
- **파일 기반 라우팅**: `app/(tabs)/index.tsx`, `app/(tabs)/explore.tsx` 등에서 자동 라우팅
- `_layout.tsx`에서 Tab/Stack 네비게이션 설정, 헤더 숨김 등 커스텀
- `router.replace()`로 화면 전환 (로그인/회원가입/홈 등)

## 인증 및 API 연동
- **로그인/회원가입**: Expo SecureStore로 토큰 저장, axios로 FastAPI 백엔드와 통신
- API 주소는 환경 변수(`process.env.EXPO_PUBLIC_API_URL`)로 관리, 기본값은 `http://127.0.0.1:8000`
- 백엔드 FastAPI는 OAuth2/JWT 기반 인증, SQLAlchemy로 DB 관리

## 개발/빌드/테스트 워크플로우
- **프론트엔드 실행**: `npm install` 후 `npx expo start` (Android/iOS/Web 지원)
- **백엔드 실행**: Python 환경에서 `uvicorn app.main:app --reload` (DB URL/SECRET_KEY는 .env로 관리)
- **프로젝트 초기화**: `npm run reset-project` (starter 코드 분리)
- **Lint**: `npm run lint` (Expo ESLint 설정)

## 주요 패턴 및 컨벤션
- **TypeScript strict 모드** + 경로 별칭(`@/`) 사용
- **ThemedView/ThemedText** 등 테마 컴포넌트 일관 사용
- **API 에러 핸들링**: axios 에러 응답별 Alert 처리
- **DB/모델/스키마**: SQLAlchemy + Pydantic 구조, 필드/유효성 검사 명확히 분리
- **환경 변수**: `.env` (백엔드), `process.env.EXPO_PUBLIC_API_URL` (프론트)

## 통합/외부 연동
- **React Navigation** + Expo Router로 화면 관리
- **FastAPI**: OAuth2/JWT, SQLAlchemy, Pydantic, passlib, jose 등 사용
- **Expo SecureStore**: 토큰 안전 저장

## 예시 코드
- 로그인: `app/(tabs)/index.tsx` → axios로 `/login/` POST, 토큰 SecureStore 저장
- 회원가입: `app/signup.tsx` → axios로 `/signup/` POST, 에러별 Alert
- DB 모델: `app/models.py` → User 테이블 정의

---

**AI Agent 지침**
- 파일 기반 라우팅, 테마 컴포넌트, 환경 변수, 인증 흐름 등 위 패턴을 반드시 준수
- 프론트/백엔드 모두에서 코드 변경 시, 관련 의존 파일(예: 모델/스키마/라우터) 함께 수정
- Expo/React Native와 FastAPI가 통합된 구조임을 항상 고려
- 불명확한 부분은 사용자에게 질문하여 명확히 할 것

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AMP-TEAM-1)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/AMP-TEAM-1)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
