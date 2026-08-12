# Copilot Instructions for linkedin-auto-poster

> **Public template repo** — this is a reusable template for LinkedIn auto-posting.
> Do not include personal references, employer names, or individual identifiers.

LinkedIn auto-poster that generates posts about industry news.

## Key principles
- All generated text must follow the voice profile in src/drafts/voice_profile.md
- Never include customer names, PII, or monetary amounts in posts
- Posts should sound like a human wrote them on their phone, not an AI
- Verify technical claims against source URLs before including them
- Always update README.md when changing features, config, architecture, or workflows

## Code change workflow
When making code changes (not post drafts):
- **Always rubber duck** the approach before implementing non-trivial changes
- **Always validate** after changes - run the full validation checklist below
- **Always validate in GitHub Actions** - trigger workflows and check logs. Do NOT validate only locally.
- **Never remove features** unless explicitly asked to

### GitHub-first principle
All pipeline execution happens in GitHub Actions, not locally. Only run locally if GitHub Actions is not possible. When validating:
1. Push code changes to the repo
2. Trigger the relevant workflow (fetch-and-draft, tests, etc.)
3. Check the workflow logs for success/failure
4. Iterate if needed

### Validation checklist (run after every code change)
| Check | How |
|---|---|
| Tests | python -m pytest --tb=short -q - all must pass |
| Lint | python -m ruff check src/ tests/ main.py - all clean |
| Security Scan | Review for secrets, PII, hardcoded creds |
| Fetch Smoke Test | python main.py fetch - verify feed fetching works |
| Publish Dry Run | python main.py publish --dry-run - validate without posting |
| Docs | Update README.md / copilot-instructions.md if needed |

## LinkedIn API
- Use UGC Posts API (/v2/ugcPosts), NOT /rest/posts (truncates at 400 chars)
- Always include source_url as article attachment for full text support
- LinkedIn limit is 3000 chars, aim for 800-1400 chars
- Line breaks must be \n\n for paragraph separation

## Authentication
- Copilot SDK: reads COPILOT_GITHUB_TOKEN from env (fine-grained PAT with Copilot Requests permission)
- LinkedIn: LINKEDIN_ACCESS_TOKEN from GitHub Secrets
- GitHub API: use COPILOT_GITHUB_TOKEN for cross-repo access (GITHUB_TOKEN is repo-scoped)

## Security rules
- No secrets in code - everything via environment variables or GitHub Secrets
- No enforce_admins on branch protection (owner must be able to merge)
- No signed commit requirements
- CodeQL only on public repos (requires GHAS on private)
- dependency-review only on public repos
- SHA-pin all GitHub Actions
- Use actions/checkout@v6 and actions/setup-python@v6 (Node.js 24)

## Architecture
- src/feeds/research_agent.py - evidence-gathering agent (Learn search, Terraform verify, article fetch)
- src/feeds/research_tools.py - 3 research tools used by the agent
- src/drafts/drafter.py - dual-model pipeline (Opus drafts, GPT-5.4 critiques)
- src/drafts/copilot_client.py - GitHub Copilot SDK wrapper
- src/drafts/validator.py - content safety + banned phrase validation + line break sanitizer
- src/feeds/ - RSS + GitHub Releases fetching
- src/linkedin/ - OAuth + publishing via UGC API
- src/publish_queue.py - scheduled posting queue
- src/email_digest.py - email news digest
- content-topics.yaml - content calendar

## Posting workflow
- approve-post label + merge = publish immediately
- post-tomorrow label + merge = queue for next day morning
- post-monday label + merge = queue for next Monday morning
- Labels are mutually exclusive
- publish-scheduled.yml runs hourly to check queue

## When reviewing or editing posts
- Check that claims match the source URL
- Verify service names and features are accurate
- Run python main.py publish --dry-run to validate before approving
- Use the voice profile rules to ensure posts don't sound AI-generated

---
> Source: [martinopedal/linkedin-auto-poster](https://github.com/martinopedal/linkedin-auto-poster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-12 -->
