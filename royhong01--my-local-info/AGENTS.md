# 픽앤조이 (pick-n-joy.com) — Copilot 프로젝트 가이드

> GitHub Copilot이 이 프로젝트 작업 시 항상 먼저 읽어야 할 파일입니다.
> Copilot 작업은 이 파일과 `COPILOT_MEMORY.md`, `WORK_LOG.md`를 기준으로 운영합니다.

## 프로젝트 기본 정보
- 사이트명: 픽앤조이 (pick-n-joy.com)
- 슬로건: "당신의 일상을 Pick, 당신의 주말을 Enjoy!"
- GitHub: https://github.com/RoyHong01/my-local-info
- Cloudflare Pages URL: https://my-local-info-2gs.pages.dev
- 최종 도메인: https://pick-n-joy.com

## 기술 스택
- Next.js 16 (App Router) + TypeScript + Tailwind CSS v4
- 정적 HTML 배포 (`next.config.ts`의 `output: "export"`, `trailingSlash: true`)
- Gemini API (gemini-2.5-pro) — 블로그 글 자동 생성
- Claude API (claude-haiku-4-5) — 데이터 description_markdown 생성
- 공공데이터포털 API + 한국관광공사 TourAPI — 데이터 수집
- GitHub Actions — 매일 04:00 KST 자동 실행
- Cloudflare Pages — 호스팅 및 배포

## 환경변수 (.env.local)
- **환경변수 운영 원칙**: 민감 키는 `.env.local`만 사용하고 `.env`는 사용/유지하지 않는다.
- **우선순위 원칙**: 동일 키가 있으면 `.env.local` 값을 기준으로 적용한다.
- ANTHROPIC_API_KEY: Claude API 데이터 본문(description_markdown) 생성
- GEMINI_API_KEY: Gemini API 블로그 생성
- PUBLIC_DATA_API_KEY: 공공데이터포털 (보조금24, 인천 데이터)
- TOUR_API_KEY: 한국관광공사 TourAPI
- KAKAO_API_KEY: 카카오 로컬 API 호환용
- KAKAO_REST_API_KEY: 일상의 즐거움 맛집 수집용 카카오 로컬 API
- NEXT_PUBLIC_ADSENSE_ID: Google AdSense (미설정)
- NEXT_PUBLIC_GA_ID: Google Analytics (✅ G-6VNKGES4FW)
- NEXT_PUBLIC_COUPANG_PARTNER_ID: 쿠팡 파트너스 (✅ AF5831775)

## API 엔드포인트
- 보조금24: https://apis.data.go.kr/1741000/Subsidy24
- TourAPI (KorService2): https://apis.data.go.kr/B551011/KorService2
  - searchFestival2: 축제 목록 조회
  - detailCommon2: 축제 상세 설명(overview) 조회 (defaultYN, overviewYN 파라미터 사용 금지 — KorService2에서는 기본 포함)
- 카카오 로컬: https://dapi.kakao.com/v2/local/search/keyword.json

## 콘텐츠 카테고리 & 데이터 파일
1. 인천 지역 정보 → `public/data/incheon.json`
2. 전국 보조금·복지 → `public/data/subsidy.json`
3. 전국 축제·여행 → `public/data/festival.json`
4. 전국 맛집 → `src/app/life/restaurant/data/restaurants.json` (카카오 API + Gemini 스냅샷)

## 만료 처리 방식
- 파일 삭제 X (SEO 보존)
- frontmatter의 `expired: true` 마킹
- 목록 페이지에서 expired 항목 제외
- 상세 페이지에서 "종료된 행사" 배지 표시

## 맛집 DB 직결 전환 기준
- 현재 기본 구조는 `Kakao 수집 -> Supabase 캐시(평점) -> restaurants.json 스냅샷 -> 블로그 생성` 순서를 유지한다.
- 아래 조건 충족 전에는 블로그 생성기를 DB 직결로 변경하지 않는다.
  1. `restaurants_cache` 누적 1,000건 이상
  2. `restaurants.json` 용량/편집 성능 이슈가 반복 발생
  3. 즐겨찾기/조회수 등 동적 기능 요구가 확정
- 전환 검토 시에도 우선은 비용/장애 추적이 쉬운 스냅샷 구조를 기본으로 유지하고, 단계적으로 전환한다.

## 핵심 폴더 구조
```
src/app/
  page.tsx            # 메인 (멀티카테고리)
  incheon/            # 인천 지역 정보 목록 + [id] 상세
  subsidy/            # 전국 보조금 목록 + [id] 상세
  festival/           # 전국 축제·여행 목록 + [id] 상세
  blog/               # AI 블로그 목록 + [slug] 상세
  rss.xml/            # RSS 피드 (route.ts)
  about/              # 소개 페이지
src/components/
  BlogFilter.tsx      # 블로그 카테고리 필터 (use client)
src/lib/
  posts.ts            # PostData 인터페이스, 블로그 파싱 (image 필드 포함)
src/content/posts/    # 블로그 마크다운 파일들
scripts/
  collect-incheon.js  # 인천 데이터 수집
  collect-subsidy.js  # 보조금 데이터 수집
  collect-festival.js # 축제 데이터 수집 (detailCommon2로 overview 포함)
  ensure-life-restaurant-candidates.mjs # 맛집 후보 충분 여부 점검 (guard 스크립트, GitHub Actions 진입점)
  collect-life-restaurants.mjs # 일상의 즐거움 맛집 스냅샷 수집 (guard에 의해 조건부 호출)
  generate-life-restaurant-posts.mjs # 맛집 전용 블로그 포스트 생성
  generate-blog-post.js # Claude API 블로그 생성 (카테고리별 2편)
  cleanup-expired.js  # 만료 콘텐츠 처리
  generate-sitemap.js # sitemap.xml 생성 (postbuild)
.github/workflows/
  deploy.yml          # 매일 04:00 KST 자동화
public/images/        # 기본 OG 이미지 4종 (SVG)
```

## 작업 규칙
1. 작업 전 이 파일과 `COPILOT_MEMORY.md`를 먼저 확인
2. 세부 작업 이력은 `WORK_LOG.md`에 기록
3. Copilot 운영 문서는 `.github/copilot-instructions.md`와 `COPILOT_MEMORY.md`를 기준으로 유지
4. `npm run build` 항상 마지막에 실행해서 빌드 오류 확인
5. 빌드 성공 후 `git add . → git commit → git push` 순서로 배포
6. 세션 종료 시 `WORK_LOG.md`, `COPILOT_MEMORY.md`, `.github/copilot-instructions.md` 업데이트
7. **항상 적용할 종료 루틴(필수)**
  - 코드 변경 작업 완료 후, 반드시 `build 성공 → commit/push → 문서/메모리 동기화` 순서를 수행
  - 문서/메모리 동기화 대상: `WORK_LOG.md`, `.github/copilot-instructions.md`, `COPILOT_MEMORY.md`
  - 위 루틴 미완료 상태에서는 작업 완료로 보지 않음
8. **커밋/배포 완료 직후 자동 동기화**: 작업 완료 후 즉시 `WORK_LOG.md`와 메모리 파일(`COPILOT_MEMORY.md`, `PROJECT_MEMORY.md`)을 자동으로 업데이트한다. 사용자에게 별도 확인 질문을 하지 않는다.
9. **사용자가 요청하지 않은 수정 금지**: 사용자가 명시적으로 요청한 파일·코드만 수정한다. 프롬프트가 불분명하거나 작업 대상이 모호할 경우, 임의로 짐작해서 수정하지 말고 반드시 사용자에게 확인 질문을 한 뒤 작업을 진행한다.
10. **초이스 포스트 본문 배너 금지**: 쿠팡 배너/링크는 본문(마크다운 body) 안에 절대 삽입하지 않는다. 쿠팡 배너는 frontmatter(`coupang_banner_image`)를 통해 사이드바에서 자동 렌더링된다. 본문에는 제품 정보·큐레이션 콘텐츠만 작성한다.11. **통합 콘텐츠 & 수익 링크 정책**: 일상의 즐거움 / 픽앤조이 초이스 자동 생성 글은 신규/기존 모두 Web/Mobile에 동일하게 적용되는 쿠팡 파트너스 제휴 링크 삽입이 필수다.
  - **Primary (이미지 기반)**: 제품 이미지가 있는 경우, 이미지 바로 아래에 명확한 CTA 링크 또는 버튼을 삽입한다.
  - **Secondary (텍스트 기반)**: 이미지가 없는 경우에는 본문 2~3번째 문단 이후 자연스러운 전환 지점에 링크를 넣는다.
  - **스타일 & CTA**: 링크는 별도 줄에 **굵게** 표시하고, 모바일에서도 잘 눌리는 간결한 CTA 문장을 사용한다.
  - **예시**: `👉 [제품명] 최저가 확인 및 상세정보 보기`, `🛒 오늘의 추천 상품, 실시간 할인 가격 확인하기`
  - **기존 글 업데이트 규칙**: 기존 포스트를 수정할 때도 원문 맥락을 유지하며 적절한 위치에 링크를 후방 삽입한다.
  - **톤 & 일관성**: 링크는 자연스럽고 도움이 되는 느낌으로 배치하며, 픽앤조이의 “프리미엄 라이프스타일 큐레이션” 톤을 유지한다.
## 헤더/네비게이션 패턴
- 헤더는 10개 페이지에 동일 패턴 반복 (컴포넌트 미분리 상태)
- 로고: `text-3xl font-bold text-orange-500` "픽앤조이 🎯"
- 네비: `text-base font-medium text-stone-600`, 간격 `space-x-4 md:space-x-6`
- 메뉴명: 인천시 정보 | 전국 보조금·복지 정책 | 전국 축제·여행 정보 | 블로그

## SEO 현황
- ✅ `<html lang="ko">`, metadataBase, 전 페이지 OG/canonical
- ✅ sitemap.xml (postbuild 자동 생성, pick-n-joy.com 도메인)
- ✅ robots.txt, RSS 피드 (/rss.xml)
- ✅ JSON-LD (블로그 상세)
- ✅ 네이버 서치어드바이저 인증
- ✅ favicon (ico + svg)
- ✅ og:image 자동화 (TourAPI firstimage + 카테고리별 기본 SVG)

## 최근 동기화 메모 (압축판)

- 상세 이력은 `WORK_LOG.md`에 누적하고, 본 문서는 운영 규칙/현행 상태 위주로 유지한다.
- 2026-04-10 핵심 반영:
  - **만료 보조금 자동 감지 보강**: `scripts/collect-subsidy.js` + `scripts/cleanup-expired.js`에서 `(YYYY.MM.DD.한)` 패턴을 파싱해 기한 경과 항목을 `expired: true`로 자동 보정
  - **만료 항목 정리**: `서비스ID 131200000013`(사회적기업 지방세 감면) `expired: true` 처리 및 연관 포스트 삭제
  - **맛집 생성 모델 안정화**: `scripts/generate-life-restaurant-posts.mjs`의 Gemini 모델을 `GEMINI_MODEL` env 기반으로 전환(기본 `gemini-2.5-flash-lite`)
  - **배포 워크플로우 정합화**: `.github/workflows/deploy.yml` 3단계 맛집 생성 step에 `GEMINI_MODEL: 'gemini-2.5-flash-lite'` env 추가
- 2026-04-08 핵심 반영:
  - **맛집 재수집 정책 전환**: GitHub Actions에서 `collect-life-restaurants.mjs` 직접 호출 → `ensure-life-restaurant-candidates.mjs`(guard)로 교체
  - guard 로직: unused 후보 10건 이상이면 재수집 생략, 10건 미만일 때만 실제 수집 실행 (`MIN_UNUSED_RESTAURANT_CANDIDATES` env로 override 가능)
  - **비용 절감 KPI 가시화**: `cache_hit`, `cache_miss`, `google_called` 지표를 일일 리포트/텔레그램에 노출
  - Supabase 평점 캐시(`restaurants_cache`) 적용: kakao_id 기준 hit 시 Google Places 호출 생략
- 2026-04-03 핵심 반영:
  - GitHub Actions 스케줄 04:00 KST로 변경 (`0 19 * * *`)
  - admin/admin/runs 배경 bg-cherry-blossom 통일
  - 맛집 파이프라인 2지역 → 3지역(`incheon`/`seoul`/`gyeonggi`) 분리
  - 지역당 30개 후보 수집 (월 1회 수집 목표)
  - 빈 버킷 자동 재수집 → unused 10건 미만일 때만 재수집으로 정책 변경
  - 프론트엔드 RestaurantExplorer 3탭 (인천/서울/경기)
  - 텔레그램 리포트 이슈 4건 수정 (경기 미생성, 테이블 CSS, 리포트 재배포)
- 고정 참고값:
  - 쿠팡 파트너ID `AF5831775`
  - 사이드바 배너 `976244`, 하단 배너 `976089`
  - 쿠팡 배너는 공식 iframe URL 직접 사용 + `referrerPolicy="unsafe-url"`

## 쿠팡 파트너스 배너 현황 (2026-03-29 최종)
- 파트너ID: AF5831775
- 사이드바: id **976244** (고객 관심 기반 추천), 240×600, `CoupangBanner` (`'use client'` 포함)
  - src: `https://ads-partners.coupang.com/widgets.html?id=976244&template=carousel&trackingCode=AF5831775&subId=&width=240&height=600&tsource=`
- 하단: id **976089** (카테고리 베스트 - 패션의류/잡화), 680×300, `CoupangBottomBanner`
  - src: `https://ads-partners.coupang.com/widgets.html?id=976089&template=carousel&trackingCode=AF5831775&subId=&width=680&height=300&tsource=`
- 구현 방식: 파트너스 공식 iframe URL 직접 사용, `referrerPolicy="unsafe-url"` 필수
- 적용 페이지: blog목록/상세, incheon목록/상세, subsidy목록/상세, festival목록/상세, about (총 9곳)

## 백로그

- [x] Google Analytics 설정 ✅
- [ ] Google AdSense 설정
- [ ] 자동화 오류 모니터링 강화
- [ ] 맛집 포스트 이미지 전략 고도화

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RoyHong01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
