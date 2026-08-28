# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 빌드 및 개발 명령어

```bash
npm run dev    # 개발 서버 (localhost:3000)
npm run build  # 프로덕션 빌드
npm run lint   # ESLint
```

## 기술 스택

- Next.js 16 (App Router), React 19, TypeScript 5
- Tailwind CSS v4 (CSS 기반 설정, tailwind.config 없음 — `app/globals.css`의 `@theme` 사용)
- shadcn/ui (new-york 스타일), lucide-react, Recharts 3
- NextAuth.js v5 beta (`next-auth@beta`), next-themes
- Google Sheets API v4 (googleapis, 서비스 계정 방식)

## 아키텍처

### 라우팅 (Route Groups)
- `app/(auth)/` — 로그인 페이지 (센터 정렬 레이아웃)
- `app/(dashboard)/` — 보호된 대시보드 (사이드바 + 헤더 레이아웃)
- `proxy.ts` (루트) — Next.js 16의 라우트 보호 프록시 (middleware.ts → proxy.ts 변경됨)

### 인증 흐름 (auth.ts)
- Google OAuth 환경변수(`AUTH_GOOGLE_ID`, `AUTH_GOOGLE_SECRET`) 유무에 따라 자동 전환
  - 설정됨 → Google Provider
  - 미설정 → 개발용 Credentials Provider (이메일만 입력)
- `ALLOWED_EMAILS` 환경변수로 이메일 화이트리스트 (비어있으면 모두 허용)
- 에러 페이지는 `/login?error=코드` 로 리다이렉트 → 한국어 메시지 표시

### 데이터 3-레이어 패턴
```
lib/sheets.ts   → Google Sheets API 래퍼 (fetchSheetData)
lib/data.ts     → 통합 레이어 (getDashboardData: Sheets 또는 mock 폴백)
lib/mock-data.ts → 기본 목 데이터
```
- 각 시트를 `Promise.all`로 병렬 페칭, 개별 시트 실패 시 해당 영역만 mock 폴백
- `GOOGLE_PRIVATE_KEY`의 `\\n`을 실제 줄바꿈으로 변환 필수 (sheets.ts에서 처리)

### Server Component vs Client Component
- **Server**: 대시보드 페이지, KPI 카드, 주문 테이블 (데이터 페칭/렌더링 전용)
- **Client** (`"use client"`): Recharts 차트, 사이드바(useState), 헤더(useSession/DropdownMenu), 테마 토글(useTheme), 프로바이더

### 스타일링 규칙
- 색상: `app/globals.css`의 oklch CSS 변수 사용 (다크모드 자동 전환)
- 차트: `var(--chart-1)` ~ `var(--chart-5)` CSS 변수 (단, Recharts SVG에서는 직접 HSL 값 매핑)
- 사이드바: `var(--sidebar-*)` CSS 변수 활용

## 코드 규칙

- **언어**: 코드 주석/커밋 메시지/문서 → 한국어, 변수명/함수명 → 영어
- **금액 포맷**: `₩${(amount / 10000).toLocaleString()}만` (만원 단위)
- **타입 정의**: `types/dashboard.ts`에 집중 — KpiData, MonthlyRevenue, CategoryDistribution, RecentOrder
- **shadcn/ui**: 설치된 컴포넌트만 사용 (Card, Button, Avatar, Table, DropdownMenu)
- **환경변수**: `.env.local`에 설정, `.env.example`이 템플릿

## 주의사항

- Next.js 16에서 `middleware.ts` → `proxy.ts`로 변경됨 (export도 `proxy`)
- `<html>` 태그에 `suppressHydrationWarning` 필수 (next-themes)
- `next-auth@beta` (5.0.0-beta.30) 사용 중 — `AUTH_` 접두사 환경변수 자동 인식

---
> Source: [citizendev9c/googlesheet-dashboard-starterkit](https://github.com/citizendev9c/googlesheet-dashboard-starterkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-28 -->
