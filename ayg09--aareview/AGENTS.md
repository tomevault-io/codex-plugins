# 🤖 GitHub Copilot Instructions - AWA (AAR with AI)

> AI 기반 After Action Review 협업 도구를 위한 코딩 에이전트 가이드
> 
> **🚀 배포 상태**: Vercel + Supabase Cloud 운영 중

---

## 🌐 언어 규칙

> ⚠️ **필수**: 모든 추론 과정과 답변은 **한국어**로 작성하세요.

---

## 🔬 MCP 활용 필수 작업 프로세스

### 🚨 트리거 조건
- 사용자가 **"mcp"** 또는 **"mcp로"**를 언급하면 **무조건 아래 5단계를 수행**
- 지침 무시 금지: 트리거 조건 충족 시 바로 작업하지 않고 반드시 프로세스 따름

### ⭐ 필수 작업 프로세스 (5단계)

#### 1단계: 요청 파악 (Sequential Thinking)
- **Sequential Thinking MCP로 사용자의 요청사항이 무엇인지 먼저 파악하고 이해**
- 요청의 목적, 범위, 기술적 요구사항 명확히 정리
- ⚠️ **비개발자 사용자 주의사항**: 사용자는 비개발자이므로, 프로그래밍 언어상 문제가 없는 것에 대해 문제삼거나 엉뚱한 요청을 할 수 있음. 요청 내용을 1차원적으로 해석하지 말고, **사용자가 요청한 것으로 인해 다른 추가적인 작업이 필요한 영역이 있는지 반드시 검토**해야 함

#### 2단계: 심층 분석 (Context7 + Tavily)
- 사용자 요청사항에 대한 솔루션/원인 심층 분석
- **Context7**: 공식 문서, 권장 지침, 코드 예제 확인
- **Tavily**: 개발자 커뮤니티 최신 노하우, 베스트 프랙티스 검색

#### 3단계: 기술 검증 및 최신 정보 확인 (핵심 단계)

> ⚠️ **이 단계의 목적**: 사용자가 특정 모델/버전을 언급하지 않더라도, 요청을 해결하는 데 필요한 **모든 관련 기술의 최신 공식 지침**을 확인하는 것

**Context7/Tavily로 반드시 확인할 사항:**

| 확인 항목 | 질문 예시 |
|-----------|-----------|
| **공식 권장 방식** | 이 문제를 해결하는 공식 문서의 권장 패턴은? |
| **최신 버전 지침** | 현재 프로젝트가 사용하는 기술 스택의 최신 버전에서 권장하는 방법은? |
| **커뮤니티 베스트 프랙티스** | 개발자 커뮤니티에서 이 문제를 어떻게 해결하라고 추천하는가? |
| **호환성/대안 라이브러리** | 현 프로젝트 환경에서 호환되는 다른 라이브러리나 도구는? |
| **알려진 이슈/주의사항** | 이 접근 방식의 알려진 문제점이나 함정(pitfall)은? |

**검증 결과에 따른 분기:**
- ✅ **정보가 현 프로젝트 기술 스택과 일치** → 5단계로 진행
- ⚠️ **버전 불일치 또는 충돌 발견** → 4단계에서 사용자 확인 필수
  - 예: "Next.js 15에서는 이 방식이 deprecated되었는데, 새로운 방식으로 진행할까요?"
  - 예: "공식 문서에서는 A 방식을 권장하는데, 현재 코드는 B 방식입니다. 어떻게 할까요?"
- ❓ **정보 불충분** → Context7/Tavily 재검색 후 3단계 반복

#### 4단계: 사용자 확인 대기 (3단계와 순환)
- 사용자가 **'어', '응', '고'** 등 긍정 발언 → **5단계로 진행**
- 사용자가 다른 지시 → Context7/Tavily로 재정보 수집 후 3단계로 복귀
- ⚠️ **3~4단계는 사용자 요청 모델과 동일 정보 확보 시까지 순환 가능**

#### 5단계: 작업 계획 및 실행 (Shrimp Task Manager)
- **Shrimp Task Manager MCP** 호출
- 확보된 정보 재검토
- `plan_task` 또는 `split_tasks`로 작업 계획 수립
- 계획에 따라 순차적으로 구현 및 검증

---

## 🏗️ 아키텍처 개요

```
Client (Next.js 15 App Router)
    ↓ React Query
API Routes (/api/**)
    ↓
Supabase Cloud (PostgreSQL + Realtime)
    ↓
AI Services (Genkit → Gemini/Claude/OpenAI)
```

### 배포 환경
- **프론트엔드**: Vercel
- **백엔드/DB**: Supabase Cloud (PostgreSQL + Realtime + Storage)
- **AI**: Genkit (사용자 API 키 기반)

### 핵심 데이터 흐름
- **워크샵 생성**: `facilitator_id` → `workshops` 테이블 (6자리 코드 자동생성)
- **실시간 협업**: Supabase Realtime (presence + postgres_changes)
- **AI 제안**: 사용자 API 키 (`localStorage`) → `/api/ai/*` → Genkit flows

## 📁 중요 파일 및 패턴

### 타입 정의 (필수 확인)
```typescript
// 새 타입 작성 전 반드시 확인
src/types/domain.ts     // 핵심 도메인 타입 (Workshop, AARRecord, PostIt)
src/types/aar.ts        // AAR 전용 타입 (로컬 앱 호환용, camelCase)
```

### Step 3 포스트잇 구조 (핵심 패턴)
```typescript
// PostIt 계층 구조 - InteractiveCauseAnalysis 컴포넌트 필수 사용
interface PostIt {
  id: string;
  hierarchyId: string;    // "a", "a-1", "a-1-2" 형식
  parentId: string | null;
  level: number;          // 0=루트, 1=1차, 2=2차...
  sentiment?: 'positive' | 'negative' | null;  // 성공/실패 구분
}
// ❌ 금지: 단순 텍스트 필드로 Step 3 구현
// ✅ 필수: InteractiveCauseAnalysis 컴포넌트로 포스트잇 트리 구현
```

### API 레이어 구조
```
src/lib/api/           # API 클라이언트 함수
src/lib/supabase/      # Supabase 클라이언트 설정
src/app/api/           # Next.js API Routes
  ├── ai/              # AI 제안 엔드포인트
  ├── comments/        # 의견 CRUD
  ├── workshop/        # 워크샵 관리
  └── gateway/         # 게이트웨이 인증
```

## 🔑 핵심 개발 규칙

### 1. 컴포넌트에서 직접 fetch 금지
```typescript
// ❌ 잘못된 방식
const data = await fetch('/api/workshop');

// ✅ 올바른 방식 (React Query + lib/api)
import { useWorkshop } from '@/hooks/api/use-workshop';
const { data } = useWorkshop(workshopId);
```

### 2. 의견(Comment) 섹션 네이밍 규칙
```typescript
// Step별 전체 의견: "step1", "step2", "step3", "step4"
// Step 3 개별 포스트잇: "step3_cause_{hierarchyId}", "step3_success_{hierarchyId}"
// 예시: "step3_cause_a-1-1", "step3_success_b-2"
```

### 3. AI API 키 처리
```typescript
// 클라이언트에서 API 키 전달
const apiKeys = localStorage.getItem('llm-api-keys');
headers: { 'X-API-Keys': apiKeys }

// 서버에서 폴백 처리
const userKeys = parseApiKeys(req.headers['x-api-keys']);
const provider = userKeys?.gemini ? 'gemini' : 'claude'; // 사용 가능한 키로 폴백
```

### 4. Supabase Realtime 구독 패턴
```typescript
// use-realtime.ts 훅 활용
useWorkshopPresence(code, currentUser);     // 온라인 참여자 추적
useAARRecordSubscription(aarId, onUpdate);  // AAR 변경 감지
useCommentsSubscription(workshopId, ...);   // 의견 실시간 동기화
```

## 🛠️ 개발 명령어

```bash
npm run dev              # 개발 서버 (next dev)
npm run build            # Next.js 프로덕션 빌드
npm run genkit:dev       # Genkit AI 개발 서버
npm run lint             # ESLint
npm run typecheck        # TypeScript 검사

# Supabase (마이그레이션)
supabase db push         # 스키마 적용
supabase migration new   # 새 마이그레이션 생성
```

## 🤖 AI 프롬프트 작성 규칙

> Gemini, Claude, OpenAI 공식 가이드라인 기반 최적화 패턴

### 공통 원칙
```typescript
// 1. 명확하고 구체적인 지시 (모든 모델 공통)
const prompt = `
당신은 AAR(After Action Review) 분석 전문가입니다.

<context>
- 프로젝트 목표: ${expected}
- 실제 결과: ${actual}
</context>

<task>
위 정보를 바탕으로 근본 원인 5가지를 5-WHY 기법으로 분석하세요.
</task>

<output_format>
JSON 배열 형식으로 반환:
[{ "cause": "원인", "why_chain": ["1차 왜", "2차 왜", ...] }]
</output_format>
`;
```

### 모델별 최적화

**Gemini (기본 모델)**
- XML 스타일 태그 또는 마크다운 구분자 사용
- 퓨샷 예시 2-3개 제공 권장
- 온도 기본값 1.0 유지 (복잡한 추론 작업)

**Claude**
- `<xml_tags>` 적극 활용 (구조화에 강점)
- Chain of Thought: "단계별로 생각한 후 답변하세요"
- 프리필 응답으로 형식 유도: `응답: {`

**OpenAI (GPT-4o)**
- System prompt에 역할과 제약조건 집중
- Function calling으로 구조화된 출력 유도
- 간결한 지시 + 명확한 예시

### 프롬프트 구조 템플릿 (src/ai/flows/)
```typescript
// 권장 구조
const systemPrompt = `
<role>AAR 분석 전문가</role>
<constraints>
- 한국어로 응답
- 실행 가능한 제안만 포함
- 3-5개 항목으로 제한
</constraints>
`;

const userPrompt = `
<context>${contextData}</context>
<task>${specificTask}</task>
<output_format>${formatSpec}</output_format>
`;
```

## ⚠️ 절대 하지 말아야 할 것

1. **`any` 타입 사용** → `unknown` 또는 제네릭 사용
2. **`domain.ts` 확인 없이 새 타입 정의** → 중복/동의어 타입 생성 금지
3. **Step 3을 단순 텍스트로 구현** → `InteractiveCauseAnalysis` 필수
4. **컴포넌트에서 직접 Supabase 호출** → `lib/api/` 레이어 사용
5. **환경변수 하드코딩** → `.env.local` 및 Vercel 환경변수 사용

## 📊 데이터베이스 스키마 핵심

```sql
-- 주요 테이블 관계
workshops (id, code, facilitator_id)
  ↓ 1:N
workshop_participants (workshop_id, user_id, role)
  ↓ 1:1
aar_records (workshop_id, cause_analysis_notes JSONB)
  ↓ 1:N
aar_comments (workshop_id, aar_record_id, section, user_id)

-- RLS: 세션 기반 인증 (auth.users 대신 x-user-id 헤더)
```

## 🎨 UI 컴포넌트

- **shadcn/ui** (`src/components/ui/`) - 수정 금지, 그대로 사용
- **커스텀 컴포넌트** - `src/components/` 직하에 배치
- **애니메이션**: Framer Motion (`framer-motion`)

## 🔗 참고 문서

- `docs/blueprint.md` - 프로젝트 설계 철학
- `docs/AI_MODELS.md` - 지원 AI 모델 상세
- `docs/REALTIME_COLLABORATION.md` - 실시간 협업 구현
- `.cursorrules` - 프로젝트별 개발 규칙 (상세)

---
> Source: [AYG09/AAReview](https://github.com/AYG09/AAReview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
