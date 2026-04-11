# GitHub Copilot Instructions — PakishNews (Agent Handover, No Confusion)

## 🚨 Agent Lessons Learned (Top Priority — Theme Editing Safety)

Yeh section recent multi-attempt incident se nikla hua hard rule set hai. Har agent theme modify karte waqt **pehle yeh follow kare**:

1. **Repo pass != Live pass**

- Local code sahi hone ka matlab live fix nahi hota.
- Har change ke baad live HTML marker check mandatory:
  - old markers absent
  - new markers present

2. **Ghost theme activation ko assume mat karo**

- Sirf git push par rely mat karo.
- Theme zip build + upload + activate EN/UR/AR explicitly verify karo.

3. **Single source rendering enforce karo**

- Agar same UI 2 paths se ban sakta ho (template + JS/hotfix), ek path disable karo.
- Legacy injectors ko ya to remove karo ya no-op + CSS guard lagao.

4. **Post content legacy HTML ko ignore mat karo**

- Published content ke andar old wrappers reh sakte hain.
- Runtime cleanup + defensive CSS hide dono use karo jab tak content migration complete na ho.

5. **Verification evidence bina "done" mat bolo**

- Minimum evidence:
  - live URL status 200
  - required string/selector check
  - old duplicate marker check
- Agar evidence nahi, task complete claim na karo.

6. **Theme tasks ke liye default autopilot flow**

- edit -> lint -> zip -> upload/activate -> live no-cache verify -> report.

7. **User trust recovery rule**

- Agar issue repeat ho raha ho to narrative nahi, forensic proof do.
- Snippet-level root cause identify karo, phir targeted fix apply karo.

---

**Last Updated:** March 29, 2026  
**Repo:** `adibapk/pakishnews` (`main`)  
**Server:** `164.92.195.115` (SSH via IP only)  
**Coolify App ID:** `gsgcww8g0g8go4ocokswcw4g`

---

## 0) Definitive Architecture & Delivery Charter (Source of Truth)

Yeh file tamam future AI coding agents ke liye **definitive source of truth** hai. Agar kisi aur note/file se conflict ho, is file ko priority milegi.

### 0.1) Core Architecture (Mandatory)

- Production architecture: **Next.js 16 (App Router) + Ghost Headless (Trilingual EN/UR/AR)**.
- Next.js production build **Standalone Mode** mein hona chahiye (deployment-ready standalone output).
- Architecture alignment se deviation bina documented approval ke allowed nahi.

### 0.2) Stateless Build Rule (QC Hard Block)

- Dockerfile mein `.env` files copy karna **strictly prohibited** hai.
- Forbidden patterns: `COPY .env*`, `ADD .env*`, ya koi bhi secret-file bake-in approach.
- Build stateless rehna chahiye; tamam secrets sirf **Coolify Environment Variables** se inject hon.

### 0.3) Deployment Workflow (No SSH Code Drift)

- Mandatory flow: **Local -> Git Push -> GitHub Actions (Build Validation) -> Coolify Deploy**.
- Server par direct code edits via SSH **allowed nahi**.
- Git hi source of truth rahega; live drift create karna policy violation hai.

### 0.4) Quality Control (QC Ops)

- Target: touched components/pages ke liye **95+ PageSpeed** score maintain karo.
- TypeScript policy: **strict types only**, `any` allowed nahi (except explicitly documented temporary waiver).
- UI/UX policy: **Bloomberg-style minimalist design** — text-first, zero fluff, high information density.

### 0.5) Retention Policy (Deploy Records)

- Deployment records ka automatic cleanup enable rehna chahiye.
- Policy: **latest 3 records only** retain karo, baqi prune karo.
- Cleanup logic systemd timer implementation ke mutabiq enforce honi chahiye.

### 0.6) Ghost Fetching Rule (Build-time Connectivity)

- Build-time/content fetching ke liye hamesha internal Docker network aliases use karo.
- Public route/CDN par dependency mat rakho jahan internal alias available ho.
- Path mismatch aur connectivity drift avoid karne ke liye service aliases canonical rahenge.

### 0.7) Delivery Completion Standard

- Sirf guidance likhna kaafi nahi: agent ko code likhna, implement karna, test karna, aur verification evidence ke sath report karna hoga.
- "Done" tabhi claim hoga jab implementation + test + verification complete ho.

---

## 0) Communication & Execution Rules

- Jawab hamesha **roman modern urdu** mein do.
- **Autopilot mode:** chhoti chhoti confirmations mat maango; kaam end-to-end complete karo, verify karo, phir report do.
- Agar production touch ho to rollback path pehle socho, phir change karo.
- Sensitive values ko chat ya commit mein expose na karo.
- Pehlay code ko local workspace mein edit karo, phir Git push karo; Coolify auto-deploy ke baad live verification zaroor karo.
- Local workspace, Git repo, aur live server hamesha sync mein rehne chahiye.

### 0.0) Doctrine Alignment (Mandatory Before UI Suggestions)

- File: `PAKISHNEWS-DOCTRINE.md` ko UI/color/layout/content-density related suggestions se pehle silently read karna mandatory hai.
- Enforce karo:
  - Vercel/Next.js monochromatic dark aesthetic
  - strict 2-column density rule (`grid-cols-1 md:grid-cols-2`)
  - 5-line summary policy (max 5 lines, high-context, zero clickbait)
- Agar kisi request ka direction doctrine se conflict kare, pehle conflict flag karo aur doctrine-aligned alternative propose karo.

### 0.1) Living SOP Index (Must Follow)

- `WORKFLOW-GITOPS.md` → end-to-end GitOps flow + drift reconciliation
- `SECRETS-SETUP.md` → GitHub repo secrets ka click-by-click setup SOP
- `ops/safe-sync-merge.ps1` → safe local-vs-live compare/merge utility
- `ops/pakishnews-safe-cleanup.sh` → pakishnews app-id scoped safe disk cleanup (`--dry-run` first, then `--apply`)
- `ops/start-local-theme-test.ps1` → local Ghost theme test stack (`validate|up|down|restart|logs`)

> Rule: agar in SOP files aur kisi chat instruction mein conflict ho, **repo SOPs ko source of truth** treat karo.

---

always use pnpm not pnp

## 0.5) Quick-Start 60 Seconds (Agent Launch Checklist)

Agar tum naya agent ho aur jaldi context lena hai, yeh 60-second flow follow karo:

1. `docker-compose.yml` + `ai-publisher/src/config.js` read karo.
2. Confirm karo ke app ID `gsgcww8g0g8go4ocokswcw4g` hai aur server `164.92.195.115` hai.
3. YouTube critical envs mentally verify karo:

- `YOUTUBE_OAUTH_CLIENT_ID`
- `YOUTUBE_OAUTH_CLIENT_SECRET`
- `YOUTUBE_OAUTH_REFRESH_TOKEN`
- `YOUTUBE_OAUTH_REDIRECT_URI`
- `YOUTUBE_OAUTH_TOKEN_FILE=/app/data/youtube-oauth.json`

4. Shorts policy verify mindset:

- `SHORTS_YT_PRIVACY_STATUS=public`
- `SHORTS_YT_AUTO_PUBLIC=true`
- `SHORTS_YT_REQUIRED_CHANNEL_ID=UCcoVAIMFgeckjuhbSWJWIWg`

5. Agar YouTube issue ho:

- `youtube-oauth.js --verify`
- queue file check: `/app/data/shorts/queue/queue.json`
- fallback token file check: `/app/data/youtube-oauth.json`

6. End report hamesha 5 bullets mein do:

- kya change hua
- kya verify hua
- kya pending hai
- security state
- rollback note

---

## 1) 2-Minute System Map

PakishNews aik trilingual Ghost platform hai:

```
Cloudflare CDN
   ↓ HTTPS
coolify-proxy (Caddy)
   ├─ /           → ghost-en:2368
   ├─ /ur/*       → ghost-ur:2368
   ├─ /ar/*       → ghost-ar:2368
   ├─ /ai-brain*  → ai-publisher-dashboard:3001
   ├─ /cf-purge*  → cf-purge:3001
   └─ /ads.txt    → direct plain-text response from Caddy

ghost-en + ghost-ur + ghost-ar → shared mysql
ai-publisher → Gemini + Vertex/TTS + Ghost Admin APIs
```

> **Important:** Yeh setup **subdirectory-based** hai, subdomain-based nahi.

---

## 2) Current Product/Compliance Context (Must Know)

Recent compliance sprint mein yeh critical changes ho chuke hain:

1. Homepage par app-purpose section add hai with exact app name: **PakishNews Shorts** (EN/UR/AR).
2. Footer legal links EN/UR/AR mein explicit hain: About / Privacy / Terms.
3. Legal pages dedupe automation موجود hai: `dedupe-legal-pages.js`.
4. `https://pakishnews.com/ads.txt` ab live hai (Caddy response block).

Google OAuth / verification alignment ke liye hamesha ensure karo:

- App name UI text = **PakishNews Shorts**
- Legal URLs live and reachable:
  - `/about/` (+ localized `/ur/about/`, `/ar/about/`)
  - `/privacy-policy/`
  - `/terms/`
- `ads.txt` endpoint 200 + `text/plain` return kare.

---

## 3) Google Cloud Advanced API Stack (March 2026 Reality)

AI publisher ab sirf basic Gemini tak limited nahi; advanced Google stack use ho raha hai:

### 3.1 Gemini

- Core model default: `gemini-2.5-flash`
- Config source: `ai-publisher/src/config.js` (`gemini.model`)
- Environment override: `GEMINI_MODEL`

### 3.2 Vertex AI (images)

- Config keys:
  - `VERTEX_PROJECT_ID`
  - `VERTEX_LOCATION` (default `us-central1`)
  - `GOOGLE_APPLICATION_CREDENTIALS_JSON` (JSON string creds)
- Related code:
  - `ai-publisher/src/shorts-generator.js`
  - `ai-publisher/src/config.js` (`vertex` + `images.preferredSource='vertex'`)

### 3.3 Google Cloud TTS

- Enabled by default (`TTS_ENABLED=true` unless overridden)
- Voices + speaking rates per language configured in `config.js`
- Uses Cloud Platform auth scope (`google-auth-library`)

### 3.4 YouTube APIs (Data + OAuth)

- OAuth helper script: `ai-publisher/src/youtube-oauth.js`
- NPM script: `npm run youtube:oauth -- --init|--exchange --code=...|--verify`
- Required env vars:
  - `YOUTUBE_OAUTH_CLIENT_ID`
  - `YOUTUBE_OAUTH_CLIENT_SECRET`
  - `YOUTUBE_OAUTH_REFRESH_TOKEN`
  - `YOUTUBE_OAUTH_REDIRECT_URI` (default OAuth Playground)
  - Optional file fallback: `YOUTUBE_OAUTH_CLIENT_SECRET_FILE`
- YouTube Data API key:
  - `YOUTUBE_API_KEY`

---

## 4) Secrets & Security (Hard Rule)

### 4.1 Never hardcode secrets in docs

Is file mein **real keys/passwords mat likho**. Runtime values hamesha Coolify env se lo.

### 4.2 High-risk files currently in repo

Yeh files sensitive nature ki hain:

- `ai-publisher/client_secret_*.json`
- `ai-publisher/vertex-key.json`

Agar security hardening phase ho to:

1. In files ko repo se remove + ignore karo,
2. credentials Coolify/env secret store mein shift karo,
3. impacted OAuth/service keys rotate karo.

---

## 5) Source of Truth Files (Agent ke liye Fast Orientation)

### Infra / routing

- `docker-compose.yml`
- `/data/coolify/proxy/caddy/dynamic/pakishnews.caddy` (server side)

### Themes

- EN: `ghost-theme/`
- UR: `ghost-theme-ur/`
- AR: `ghost-theme-ar/`

### AI Brain

- `ai-publisher/src/config.js` (**primary behavior control**)
- `ai-publisher/src/index.js` (scheduler + pipeline)
- `ai-publisher/src/ai-writer.js` (prompt/editorial logic)
- `ai-publisher/src/shorts-generator.js` (shorts/video pipeline)
- `ai-publisher/src/tts-generator.js` (voice generation)
- `ai-publisher/src/social-embed.js` (YT/Twitter embeds)
- `ai-publisher/src/youtube-oauth.js` (OAuth lifecycle)

### Ops scripts

- `dedupe-legal-pages.js`
- `deploy-ads-txt.sh`
- `create-legal-pages.js`

---

## 6) Deployment & Change Rules (Practical SOP)

1. Local edit → `main` branch commit/push.
2. Server sync to bind mounts (`/data/coolify/applications/gsgcww8g0g8go4ocokswcw4g/...`).
3. Theme update ke baad relevant Ghost containers restart zaroor karo (template cache flush).
4. Caddy changes sirf **existing** `pakishnews.caddy` site block mein karo.
   - Duplicate `https://pakishnews.com` site block create karoge to `ambiguous site definition` error ayega.

---

## 6.1) Deployment & Sync SOPs (Strict Rules)

### Local First

- Koi bhi code ya theme change hamesha pehle local workspace mein develop aur test hoga.
- Local testing ke baad hi code ko Git par push karne ki permission hogi.

### Git as Single Source of Truth

- Git hamesha single source of truth hoga. Local test pass hone ke baad hi code Git par push hoga.
- Git repo aur local workspace ka state hamesha sync mein hona chahiye.

### Deployment Wait & Live Verification

- Git push ke baad Coolify ke auto-deploy hone ka wait karo.
- Deployment ke baad live server par ja kar verify karo ke jo task manga gaya tha, wo 100% sahi tarah live implement hua hai ya nahi.

### Strict Synchronization

- Local Workspace, Git Repo, aur Live Server hamesha 100% sync mein rehne chahiye.
- Live server par direct SSH edits strictly forbidden hain.

### Drift Safety (Enforced)

- Drift detect/reconcile ke liye `ops/safe-sync-merge.ps1 -MergeStrategy Manual` preferred hai.
- `.sync/runs/*/sync-report.json` evidence ko review kiye baghair production sync finalize na karo.
- Server `.bak`/temporary artifacts ko source-of-truth code mat samjho.

### GitHub Actions Dependency

- `main.yml` (pnpm CI Workflow): Har push par ESLint, Prettier, aur `pnpm audit` run hota hai. Agar Code Quality (SOLID, DRY) meet na ho to pipeline mein warnings aati hain. Code likhte waqt is standard ka khayal rakho.
- `gitops-deploy.yml` aur `live-drift-audit.yml` workflows ke liye required secrets `SECRETS-SETUP.md` ke mutabiq configure karo.
- Agar secrets missing hon to pehle unko setup karo, phir workflow rerun karo.

### pnpm Monorepo Rules

- Ye project **pnpm workspaces** use kar raha hai.
- Global dependencies ya formatting scripts chalane ke liye root par `pnpm run lint` ya `pnpm run format` (Prettier) use karo.
- AI Agent ko chahiye ke generated code hamesha ESLint aur Prettier rules ke mutabiq error-free produce kare.

---

## 7) Validation Checklist (Every Production Change)

### Functional

- `/`, `/ur/`, `/ar/` all return 200.
- `/ads.txt` returns 200 + plain text.
- Footer legal links teenon languages mein visible.

### AI / GCP

- AI publisher logs mein auth/model errors nahi.
- YouTube OAuth verify (`--verify`) channel return kare (jab OAuth touch ho).
- Vertex/TTS warnings absent ya expected documented ho.

### Compliance-facing

- Homepage pe **PakishNews Shorts** visible ho.
- About/Privacy/Terms URLs reachable ho.

### Local-first preflight (Theme changes)

- `ops/start-local-theme-test.ps1 -Action validate` run karo.
- Ports `2368/2369/2370`, compose config, aur Docker readiness pass honi chahiye.

---

## 8) Known Pitfalls (Repeat Mistakes Avoid)

1. Static container suffix hardcode mat karo — redeploy par change hota hai.
2. Old password notes trust mat karo; current runtime env/container inspect se verify karo.
3. `pakishnews.com` ke liye alag extra Caddy site file banane se conflict ho sakta hai.
4. Theme files copy kar ke Ghost restart na karo to old templates serve hotay rehte hain.
5. OAuth consent alignment mein app-name mismatch (PakishNews vs PakishNews Shorts) reject risk badhata hai.

---

## 9) Agent Startup Protocol (What to do first)

Har naya agent session start par:

1. `docker-compose.yml` + `ai-publisher/src/config.js` read karo.
2. `pakishnews.caddy` live route structure verify karo (server side).
3. Current container names dynamic list se nikaalo (suffix assume mat karo).
4. Task scope note karo: theme / ai / infra / compliance.
5. End mein evidence-based verification do (logs + endpoint checks).

---

## 10) Short Reminder

Goal sirf code chalana nahi — **production-safe + compliance-ready + handover-friendly** system maintain karna hai.

---

## 11) Security Hardening Runbook (V2)

Yeh section un sessions ke liye hai jahan objective secret hygiene + OAuth/GCP hardening ho.

### 11.1 Scope

- Repo se sensitive credential files hatao.
- `.gitignore` ko enforce karo taa-ke future mein dobara secret commit na ho.
- Coolify env/secret store ko single source of truth banao.
- Affected Google/Ghost/API credentials rotate karo.

### 11.2 Files to treat as sensitive

- `ai-publisher/client_secret_*.json`
- `ai-publisher/vertex-key.json`
- Koi bhi raw `.env`, service-account JSON, OAuth token dump, private key file.

### 11.3 Execution order (safe)

1. **Backup current runtime env references** (values reveal kiye baghair names list bana lo).
2. Secrets Coolify environment mein confirm/update karo.
3. Repo se sensitive files remove karo.
4. `.gitignore` entries add/verify karo.
5. Deploy + runtime verify karo (AI publisher auth, OAuth verify, Vertex/TTS logs).
6. Phir key rotation execute karo (staged manner, ek service aik dafa).

### 11.4 `.gitignore` minimum rules

Ensure yeh patterns present hon:

- `ai-publisher/client_secret_*.json`
- `ai-publisher/vertex-key.json`
- `.env`
- `.env.*`
- `*.pem`
- `*.p12`
- `*service-account*.json`

### 11.5 Rotation priority (blast radius control)

1. YouTube OAuth client secret + refresh token
2. Vertex/Google service credentials
3. Ghost Admin API keys (EN/UR/AR)
4. Cloudflare purge token
5. Gemini / YouTube Data API keys

> Rotation ke baad old credentials disable/revoke karna mandatory hai.

### 11.6 Validation after hardening

- `npm run youtube:oauth -- --verify` expected channel return kare.
- AI publisher logs mein `invalid_grant`, `unauthorized`, `403`, `quota/auth` errors absent hon.
- Vertex image generation and TTS pipeline expected behavior show kare.
- Publishing EN/UR/AR teeno instances par successful ho.

### 11.7 Rollback strategy

- Rollback sirf code ka nahi — **secret references** ka bhi hota hai.
- Agar naya credential fail kare to:
  1. Coolify env mein last-known-good secret set karo,
  2. impacted service/container restart karo,
  3. verification checks rerun karo,
  4. incident note maintain karo (what failed + why).

### 11.8 Future-proof rule

- Agent kabhi bhi chat/docs/repo mein real secret value paste na kare.
- Evidence reporting mein sirf `present/missing/valid/invalid` states use karo.
- Agar secret leak suspect ho to usay **already compromised** treat karo aur immediate rotation trigger karo.

---

## 12) YouTube Shorts Integration Runbook (Production Truth)

Yeh section **must-follow** hai agar koi bhi agent YouTube upload/OAuth/shorts distribution touch kare.

### 12.1 Current behavior (as of March 2026)

- Shorts upload target: **official channel only**
  - Channel ID: `UCcoVAIMFgeckjuhbSWJWIWg` (Pakish News TV)
- Upload pipeline default privacy:
  - `SHORTS_YT_PRIVACY_STATUS=public`
  - `SHORTS_YT_AUTO_PUBLIC=true`
- Wrong-channel protection enabled:
  - `SHORTS_YT_REQUIRED_CHANNEL_ID=UCcoVAIMFgeckjuhbSWJWIWg`
- OAuth fallback persistence enabled:
  - `YOUTUBE_OAUTH_TOKEN_FILE=/app/data/youtube-oauth.json`
  - Is file ka purpose: agar container env inject na ho to bhi uploader credentials resolve kar sakay.

### 12.2 Source files that control YouTube flow

- `ai-publisher/src/youtube-oauth.js`
  - `--init`, `--exchange --code=...`, `--verify`
  - OAuth token file persistence logic
- `ai-publisher/src/shorts-distributor.js`
  - YouTube upload + thumbnail + metadata + queue processing
  - Auto-public enforcement + required-channel guard + fallback credential load
- `ai-publisher/src/config.js`
  - `shorts.youtubePrivacyStatus`
  - `shorts.youtubeAutoPublic`
  - `shorts.youtubeRequiredChannelId`
- `docker-compose.yml`
  - All YT OAuth + shorts privacy/channel env keys

### 12.3 OAuth scopes (critical)

`--init` URL mein yeh scopes honay chahiye:

- `https://www.googleapis.com/auth/youtube`
- `https://www.googleapis.com/auth/youtube.force-ssl`
- `https://www.googleapis.com/auth/youtube.upload`
- `https://www.googleapis.com/auth/youtube.readonly`

Reason:

- sirf `upload + readonly` se old/private video update (`videos.update`) par `insufficient authentication scopes` aa sakta hai.

### 12.4 Standard OAuth lifecycle (golden path)

1. `--init` run karo aur fresh URL generate karo.
2. User se authorization code lo (official channel select karwao).
3. `--exchange` run karo.
4. Generated refresh token ko Coolify app `.env` mein persist karo:
   - `YOUTUBE_OAUTH_REFRESH_TOKEN`
   - `YOUTUBE_OAUTH_REDIRECT_URI=http://localhost` (agar localhost flow use hua ho)
5. `--verify` run karo aur channel ID confirm karo.
6. Agar channel mismatch ho to uploads block rakho aur re-auth repeat karo.

### 12.5 Channel mismatch guard policy

- Upload se pehle authenticated channel ID check karo.
- Agar expected ID se mismatch ho:
  - upload fail karo (silent fallback mat karo)
  - clear error do: expected vs actual channel
  - new auth URL generate karke user ko correct account select karne ko bolo.

### 12.6 Env propagation edge-case (known)

Kuch redeploy cases mein app `.env` set hota hai lekin runtime container env blank aa sakta hai.

Mitigation:

- OAuth fallback JSON (`/app/data/youtube-oauth.json`) maintained rakho.
- Distributor pehle env, phir fallback file se credentials resolve kare.
- Diagnostics mein dono states check karo:
  - app env key presence
  - container runtime env presence

### 12.7 Private upload issue prevention

Expected defaults:

- `SHORTS_YT_PRIVACY_STATUS=public`
- `SHORTS_YT_AUTO_PUBLIC=true`

Uploader behavior:

- Privacy status sanitize/normalize karo (`public|private|unlisted`).
- Invalid/empty status ko `public` treat karo.

### 12.8 Legacy private short remediation

If an older short remains private:

1. Verify current OAuth scope includes `youtube` / `youtube.force-ssl`.
2. `videos.list` by id + `videos.update` with `privacyStatus=public` run karo.
3. Re-verify via `videos.list`.

If video list mein return na ho:

- likely different channel ownership context hai.
- us case mein same ID ko current channel token se update nahi kar sakte.

---

## 13) SEO + GEO Execution Playbook (Website + YouTube Growth)

Yeh section growth-focused agents ke liye hai.

### 13.1 Already implemented optimizations

- YouTube shorts descriptions mein source article URL with UTM:
  - `utm_source=youtube`
  - `utm_medium=shorts`
  - `utm_campaign=pakishnews_<lang>_viral_short`
- Language-specific CTA lines description mein add hoti hain.
- Title-based keyword extraction se enriched tags generate hotay hain.
- Channel guard + auto-public se distribution reliability improve hui.

### 13.2 GEO (Generative Engine Optimization) rules for content agents

- Har article mein factual clarity high rakho (who/what/when/where/why).
- Intro ke early paragraphs mein direct answer-style summary do.
- Ambiguous claims avoid karo; source context provide karo.
- Entity names consistent rakho (person/org/location spellings).
- Multi-language parity maintain karo (EN/UR/AR meaning alignment).

### 13.3 SEO rules for article pipeline

- Headlines concise + intent-rich hon.
- Meta title/meta description duplication avoid karo.
- Internal links relevant category ke top posts se inject karo.
- Slug readable aur keyword-aligned ho.
- Thumbnail/image alt-text equivalent context (jahan applicable) ensure karo.

### 13.4 YouTube SEO rules for shorts

- Title ≤ 95 chars + `#shorts` maintain karo.
- 3–6 topical tags + brand tags + language-relevant tags use karo.
- Description mein first 2 lines hook + value statement ho.
- Source article link (UTM) zaroor include karo.
- Thumbnail permission fail ho to upload fail na karo (graceful warning only).

### 13.5 KPIs agents should monitor

- Website:
  - organic sessions
  - CTR (GSC)
  - indexed pages
  - internal link depth
- YouTube:
  - impressions CTR
  - average view duration
  - shorts feed reach
  - link click-through to site (UTM analytics)

### 13.6 High-impact next improvements (recommended backlog)

1. ✅ Daily automated analytics summary (site + shorts) in one JSON report.
2. Hook/title A/B queue for shorts.
3. Topic-cluster memory by language (winning entities repeat strategy).
4. Article-level FAQ answer block for GEO snippets.
5. Publish-time optimization by language based on best-performing windows.

### 13.7 Growth digest system (implemented)

- Exporter file: `ai-publisher/src/growth-digest-exporter.js`
- Daily scheduler: `index.js` (cron from `GROWTH_DIGEST_CRON`)
- Output JSON: `/app/data/reports/YYYY-MM-DD-growth-digest.json`
- Optional webhook push: `GROWTH_DIGEST_WEBHOOK_URL`
- Dashboard trigger available: `triggerGrowthDigest`

---

## 14) Agent Handover Checklist (No-Confusion Exit)

Session close karte waqt agent yeh report zaroor de:

1. Kya change hua (files + behavior impact).
2. Kya verify hua (logs/API/endpoints).
3. Kya pending hai (exact blocker + next action).
4. Security state (no secrets exposed, token state masked).
5. Production risk note (rollback/how to revert quickly).

Is checklist ke baghair handover incomplete treat hoga.

---

## 15) Repo Secrets Setup SOP (Quick Link)

Team ko GitHub secrets configure karne ke liye hamesha yeh file follow karni hai:

- `SECRETS-SETUP.md`

Minimum required secrets:

- `COOLIFY_DEPLOY_WEBHOOK`
- `SERVER_SSH_KEY`
- `SERVER_HOST`
- `SERVER_USER`
- `SERVER_APP_PATH` (optional; default app path fallback supported)

Policy:

- Secret names exact hone chahiye (case-sensitive)
- Secret values kabhi chat/commit mein reveal na karo
- Secret update ke baad workflow manual run karke verification karo

---

## 16) Local Ghost Theme Test Runbook (First-Run Ready)

Local testing ke liye canonical stack:

- `docker-compose.theme-local.yml`
- `ops/start-local-theme-test.ps1`

Recommended flow:

1. `validate`
2. `up`
3. `logs` (agar issue ho)
4. `down`

Expected behavior:

- EN/UR/AR Ghost containers respective ports `2368/2369/2370` par run karen.
- Agar port busy ho ya Docker unavailable ho to script preflight fail kare (yeh intentional safety guard hai).

---

## 17) AI Publisher Quality-First Sync Rules (March 2026)

Yeh rules `/ai-brain` aur backend sync ke liye **mandatory** hain:

1. **Single source-of-truth config:**

- Runtime news/image/writing controls `ai-publisher/data/prompts-config.json` se read hon.
- `src/config.js` sirf fallback ke liye treat hoga.

2. **RSS source policy:**

- Generic aggregator feed policy restricted hai; official high-authority feeds prefer karo.
- Hardcoded source lists new code mein add na karo; dashboard-editable config use karo.

3. **Scoring policy (config-driven):**

- Official domain bonus (`+70`) aur media metadata bonus (`+50`) configurable rehna chahiye.
- Domain/source scoring constants runtime config se resolve hon, code constants fallback only.

4. **Image chain policy (strict):**

- Priority: verified source image → Wikipedia entity image → social context → official fallback.
- Unsplash path globally disabled by default (quality-first mode).
- Ambiguous image query par guessing disable rahe (`allowGuessingForAmbiguousQuery=false`).

5. **Writing policy hooks:**

- Quick Answer block, contextual depth, analytical rewrite-only flags config mein maintain karo.
- Attribution template configurable ho: `Source: {agency} via PakishNews Research.`

6. **Archive-first cleanup:**

- Legacy logic delete na karo; `ops/archive/` mein move/archive reference maintain karo.
- Any removed behavior ke liye archive note + replacement path mention mandatory hai.

7. **Dashboard safety UX (Phase-2):**

- `newsSources` editing raw JSON textarea se na ho; structured repeater UI use karo.
- Image chain priorities comma-text field se na ho; reorder/toggle controls use karo.
- Non-technical admin flows mein malformed JSON risk zero rakhna mandatory hai.

---

## 18) Old Server Migration + Cleanup Guardrails (March 2026)

Yeh section un tasks ke liye mandatory hai jahan objective old server cleanup / disk reclaim / migration readiness ho.

### 18.1 Canonical cleanup script (version-controlled)

- Script path: `ops/pakishnews-safe-cleanup.sh`
- Server copy path (runtime): `/root/pakishnews-safe-cleanup.sh`
- Policy: server par hotfix script banay to **same session mein repo mein sync + commit** mandatory hai.

### 18.2 Scope lock (no cross-project damage)

- Default `APP_ID=gsgcww8g0g8go4ocokswcw4g` only.
- Cleanup sirf is app-id se prefixed containers/images/volumes aur app-path artifacts ko touch kare.
- Other projects (e.g. `globnic`, `futbify`) ke containers/volumes/images ko kabhi cleanup target na banao.

### 18.3 Mandatory execution order

1. Baseline capture: `df -h`, `docker system df`, `docker ps -a`, `docker volume ls`
2. `--dry-run` run karo aur candidates verify karo.
3. Agar candidates safe hon to `--apply` run karo.
4. Post-checks mandatory:
   - pakishnews containers healthy
   - pakishnews volumes intact
   - live endpoints 200 (`/`, `/ur/`, `/ar/`, `/ads.txt`)

### 18.4 Safe-delete class (allowed)

- Non-runtime artifacts under app path:
  - `ops/qa-screenshots`
  - `ops/reports`
  - `ai-publisher/data/smoke`
  - `pakishnews-theme.zip`
- Stopped containers with pakishnews app-id name filter
- Unused pakishnews-prefixed images/volumes (only if not referenced by running containers)

### 18.5 Never-delete class (blocked)

- Active MySQL and Ghost runtime volumes (`gsgcww8..._mysql-data`, `gsgcww8..._ghost-content-*`)
- Any running container
- Any non-pakishnews project resources

### 18.6 Verification evidence format

Cleanup task close karte waqt minimum evidence do:

1. reclaim summary (before/after)
2. deleted targets list
3. protected critical volumes list
4. container health snapshot
5. endpoint status codes

Todos ko up to date rakhna zarori hai

IP address is 139.59.219.216
ssh root@139.59.219.216

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adibapk)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/adibapk)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
