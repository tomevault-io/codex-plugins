# Copilot Instructions for tire-plan

**Project**: Multi-location tire shop POS and management system with React + TypeScript + Firebase

## Architecture Overview

### Tech Stack
- **Frontend**: React 19 + TypeScript + Vite + TailwindCSS
- **Backend**: Firebase (Firestore, Authentication, Cloud Functions)
- **State Management**: React hooks + Firebase Realtime Listeners
- **Charts**: Recharts
- **Styling**: TailwindCSS + Tailwind Merge

### Project Structure
```
src/
├── components/      # Feature-based components (Dashboard, POS, Inventory, etc)
├── utils/          # Firestore service layer, formatting utilities
├── hooks/          # Custom React hooks (useMenuAccess)
├── types.ts        # Centralized TypeScript interfaces
├── firebase.ts     # Firebase initialization
└── App.tsx         # Main app with role-based routing (~2566 lines)

functions/
├── auth.ts         # Firebase Auth functions
├── index.ts        # Cloud Functions entry
└── subscription.ts # Subscription management
```

## Critical Patterns

### 1. Type Imports vs Value Imports
```typescript
// ✅ Types use 'type' keyword
import type { Sale, Customer, Shift } from './types';

// ✅ Runtime values (enums, constants) don't use 'type'
import { PaymentMethod } from './types';
```
**Why**: PaymentMethod is an object constant used at runtime for checks like `s.paymentMethod === PaymentMethod.CARD`. Other interfaces are pure types for compile-time only.

### 2. Firestore Real-time Data Flow
Most list data (sales, shifts, staff) use `subscribeToQuery()` with `onSnapshot()` listeners:
```typescript
const unsub = subscribeToQuery<Shift>(COLLECTIONS.SHIFTS, constraints, (data) => {
  const sorted = [...data].sort((a, b) => ...);
  setShifts(sorted);
});
// Auto-unsubscribe on cleanup
return () => unsub?.();
```
**Key**: Date range queries in `App.tsx` control which data loads. Calendar and schedule views request different date ranges dynamically.

### 3. Date Handling: Timezone Pitfalls
ISO string dates can cause midnight shifts when converting to Date objects:
```typescript
// ✅ Safe: Extract date part directly from ISO string
const isoToLocalDate = (iso: string) => iso.split('T')[0];  // Returns "2026-01-03"

// ❌ Risky: new Date("2026-01-03") creates midnight UTC, not local
```
**Apply to**: Any date comparison in filters (src/components/ScheduleAndLeave.tsx, Dashboard.tsx)

### 4. Firestore Datetime Fields Store ISO Strings
Sales, shifts, and leave requests store `date: string` in ISO format ("2026-01-03T12:00:00Z").
- Always normalize with `isoToLocalDate()` before comparing
- Use `dateToLocalString()` for creating new dates: `new Date(...).toISOString()`

### 5. Sale Item Calculations
Each `Sale` has `items: SalesItem[]` where each item has `quantity: number`:
```typescript
// Sum tire quantities across all items in a sale
const tireQuantity = sale.items?.reduce((sum, item) => sum + item.quantity, 0) || 0;
```
**Used in**: Dashboard tire count displays, inventory tracking

### 6. Role-Based Access Control
Three roles with distinct permissions:
- `SUPER_ADMIN` (ID: '999999'): Master user, all features
- `STORE_ADMIN` (Owner): Own store's data, staff management
- `STAFF`: View-only, single store assigned

Implemented in `App.tsx` via conditional routing and in components via `useMenuAccess()` hook.

### 7. Multi-Store Context
Each sale, shift, and inventory record has `storeId`. Dashboard and reports filter by `selectedStoreId`. When `selectedStoreId === 'ALL'`, data aggregates across all visible stores.

## Development Workflows

### Start Development
```bash
npm run dev        # Vite server on localhost:5173
```
Hot Module Replacement auto-refreshes on file changes.

### Build & Deploy
```bash
npm run build      # TypeScript + Vite build
npm run lint       # ESLint check
```
Deployment uses GitHub Actions to Lightsail (see `LIGHTSAIL_DEPLOYMENT_GUIDE.md`).

### Testing
End-to-end tests use Playwright:
```bash
scripts/e2e-login-test.js  # Login flow verification
```

## Deployment Workflow (CRITICAL - READ FIRST)

### ⚠️ NEVER Skip These Steps

**배포 전 필수 체크리스트:**
1. ✅ **로컬 빌드 먼저 실행**: `npm run build` (서버에서 빌드하지 말 것!)
2. ✅ **dist 폴더 확인**: 빌드 후 `dist/assets/` 파일명 해시 변경 확인
3. ✅ **GitHub Actions 우선**: 수동 SSH 배포는 최후의 수단
4. ✅ **변경사항만 배포**: `deploy-to-lightsail.sh`는 이미 최적화됨 (dist만 전송)

### Deployment Methods

#### 1️⃣ **GitHub Actions (권장 방법)**
```bash
# 코드 변경 후 커밋만 하면 자동 배포
git add .
git commit -m "feat: 기능명"
git push origin main
# → GitHub Actions가 자동으로 빌드 + 배포
```

**워크플로우 위치**: `.github/workflows/deploy-lightsail.yml`
- 자동 빌드 (npm run build)
- SSH 키로 Lightsail 접속 (GitHub Secrets 저장됨)
- dist 폴더만 전송 (60초 이내 완료)
- nginx 자동 재시작

**GitHub Secrets 등록 상태** (이미 설정 완료):
- `LIGHTSAIL_SSH_KEY`: SSH private key (~/Downloads/LightsailDefaultKey-ap-northeast-2.pem)
- `LIGHTSAIL_HOST`: 52.78.72.19
- `LIGHTSAIL_USER`: ubuntu

#### 2️⃣ **수동 배포 (긴급 시에만)**
```bash
# Step 1: 로컬 빌드 (필수!)
npm run build

# Step 2: 배포 스크립트 실행
bash deploy-to-lightsail.sh 52.78.72.19 ~/Downloads/LightsailDefaultKey-ap-northeast-2.pem
```

**⚠️ 배포 스크립트는 이미 최적화됨:**
- ✅ 로컬에서 사전 빌드된 dist만 전송
- ✅ SSH 호스트 키 검증 비활성화 (`StrictHostKeyChecking=no`)
- ✅ 서버에서 npm install/build 실행 안 함 (시간 절약)
- ✅ PM2 자동 재시작 (또는 nginx 서빙)

### SSH Key 정보 (인식 필수)

**파일 위치**: `~/Downloads/LightsailDefaultKey-ap-northeast-2.pem`
- **권한**: `chmod 400` 필요
- **GitHub Secrets**: 이미 등록됨 (`LIGHTSAIL_SSH_KEY`)
- **용도**: Lightsail 서버 접속용 private key

**다시 물어보지 말 것:**
- SSH 키 경로는 항상 `~/Downloads/LightsailDefaultKey-ap-northeast-2.pem`
- GitHub Actions는 Secrets에서 자동으로 키 사용
- 수동 배포 시에도 위 경로 사용

### 서버 환경

**Lightsail 서버 정보:**
- IP: 52.78.72.19
- OS: Ubuntu 22.04.5 LTS
- User: ubuntu
- Web Server: **nginx** (PM2는 사용 안 함)
  - Port 80: HTTP → HTTPS 리다이렉트
  - Port 443: HTTPS (SSL/TLS)
  - Document Root: `/home/ubuntu/tire-plan/dist`
- Domain: https://tireplan.kr

**nginx 설정 위치**: `/etc/nginx/sites-available/tireplan.kr`
```nginx
server {
    server_name tireplan.kr;
    root /home/ubuntu/tire-plan/dist;
    index index.html;
    location / {
        try_files $uri $uri/ /index.html;
    }
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/tireplan.kr/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tireplan.kr/privkey.pem;
}
```

### 배포 후 검증

```bash
# 1. 서버 파일 타임스탬프 확인
ssh -i ~/Downloads/LightsailDefaultKey-ap-northeast-2.pem ubuntu@52.78.72.19 \
  "stat /home/ubuntu/tire-plan/dist/index.html"

# 2. nginx 상태 확인
ssh -i ~/Downloads/LightsailDefaultKey-ap-northeast-2.pem ubuntu@52.78.72.19 \
  "sudo systemctl status nginx"

# 3. 배포된 파일 해시 확인
ssh -i ~/Downloads/LightsailDefaultKey-ap-northeast-2.pem ubuntu@52.78.72.19 \
  "ls -lh /home/ubuntu/tire-plan/dist/assets/ | grep index-"

# 4. 브라우저에서 강력 새로고침 (캐시 무시)
# macOS: Cmd + Shift + R
# Windows: Ctrl + Shift + F5
```

### 문제 해결

**증상**: 변경사항이 라이브에 반영 안 됨
**원인**: dist 폴더가 최신 코드로 빌드되지 않음

**해결 방법:**
```bash
# 1. dist 삭제 후 재빌드
rm -rf dist
npm run build

# 2. 빌드 파일 해시 변경 확인
ls -lh dist/assets/ | grep index-
# → 파일명의 해시값(예: index-BExVPYBQ.js)이 변경되어야 함

# 3. 재배포
git add dist  # dist를 .gitignore에서 제외한 경우
git commit -m "build: rebuild with latest changes"
git push origin main
```

**증상**: PM2 프로세스가 계속 에러 상태
**해결**: PM2는 사용하지 않음. nginx가 정적 파일 서빙 중.
```bash
# nginx 상태 확인
ssh -i ~/Downloads/LightsailDefaultKey-ap-northeast-2.pem ubuntu@52.78.72.19 \
  "sudo systemctl status nginx"

# nginx 재시작
ssh -i ~/Downloads/LightsailDefaultKey-ap-northeast-2.pem ubuntu@52.78.72.19 \
  "sudo systemctl restart nginx"
```

**증상**: GitHub Actions 실패 (SSH 관련)
**확인 사항**:
- GitHub Secrets에 `LIGHTSAIL_SSH_KEY` 등록 여부
- SSH 키 포맷 (BEGIN/END 포함)
- Lightsail 서버 방화벽 설정 (포트 22 오픈)

### 예방 원칙

1. **빌드는 항상 로컬에서**: 서버 빌드는 느리고 불안정
2. **GitHub Actions 우선**: 일관성 있는 배포 프로세스
3. **dist 폴더 확인 습관**: 빌드 후 파일 해시 변경 체크
4. **배포 전 로컬 테스트**: `npm run dev`로 로컬 작동 확인
5. **커밋 메시지 명확히**: `feat:`, `fix:`, `build:` prefix 사용

## Common Tasks & Patterns

### Adding a Calendar Feature
1. Use `getDaysInMonth()` to get date array (Dashboard.tsx line ~267)
2. Calculate daily stats with helper functions that normalize dates
3. **Must**: Request data for full month with `onShiftRangeChange()` if spanning month boundaries
4. **Template**: Grid cells show date (top-left), amounts (bottom-right), details in tooltip

### Modifying Firestore Queries
- Constraints use Firebase query operators: `where()`, `orderBy()`, `limit()`
- Date ranges must account for timezone: use ISO string comparisons
- Always clean up listeners with `unsubscribe()` on component unmount

### Formatting Numbers
Use utilities from `src/utils/format.ts`:
- `formatCurrency(1000000)` → "₩1,000,000"
- `formatNumber(100)` → "100"

### TailwindCSS Styling
- Color palette: blue-600 (primary), emerald-600 (success), red-500 (danger), violet-600 (alternate)
- Use `clsx()` or ternary for conditional classes
- Responsive: `md:` prefix for tablet+ breakpoints

## Key Files to Know

| File | Purpose |
|------|---------|
| `src/types.ts` | All interfaces: Sale, Staff, Shift, LeaveRequest, etc. |
| `src/firebase.ts` | Firebase init, db instance |
| `src/utils/firestore.ts` | CRUD operations, `subscribeToQuery()`, `COLLECTIONS` constant |
| `src/components/Dashboard.tsx` | Revenue charts, calendar, admin analytics (~1055 lines) |
| `src/components/ScheduleAndLeave.tsx` | Weekly/monthly shift grid (~731 lines) |
| `src/App.tsx` | Main router, auth, Firestore listeners (~2566 lines) |
| `functions/index.ts` | Cloud Functions: auth, subscriptions |

## Common Pitfalls

1. **Forgetting date normalization**: Always use `isoToLocalDate()` for ISO strings before comparing
2. **Unfiltered Firestore queries**: Month-based queries may miss cross-month data (e.g., Jan 1 week includes Dec 29-31). Explicitly request wider date range if needed.
3. **Type vs value imports**: Use `import type {...}` for interfaces, plain import for runtime values
4. **Stale listeners**: Component unmount must call `unsubscribe()` to prevent memory leaks
5. **Missing store context**: Always check `storeId` when filtering sales/shifts; aggregate across stores when `selectedStoreId === 'ALL'`

## Recent Changes to Know

- **Schedule month-boundary fix**: Week view now loads data spanning both months (ScheduleAndLeave.tsx ~line 145)
- **Dashboard tire metrics**: Calendar shows daily tire count and monthly total (Dashboard.tsx ~line 290)
- **ISO date extraction**: Direct string split instead of Date object parsing (Dashboard.tsx line ~125)

---
> Source: [mytireplan/tire-plan](https://github.com/mytireplan/tire-plan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
