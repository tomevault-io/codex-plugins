# Task
Create an autonomous "earning robot" that operates only through a laptop, phone, and internet.
The robot must:
- Accept user requests (via Telegram bot or web API).
- Execute tasks using external AI APIs (e.g., Mistral, OpenAI).
- Automatically charge payments and track expenses/income.
- Generate reports and notifications for the owner.

# Architecture
1. **Frontend (control)**
   - Telegram bot or simple web panel.
   - Owner authentication.
   - Interface to view income/expenses.

2. **Backend (orchestrator)**
   - Python/Node.js server.
   - REST API for receiving requests.
   - Task scheduler (cron or Celery).
   - Logging and monitoring.

3. **Integrations**
   - AI APIs (Mistral, OpenAI).
   - Payment gateway (Stripe, PayPal, YooKassa, local bank APIs).
   - Google Sheets or SQLite for accounting.

4. **Financial logic**
   - Subscriptions (monthly).
   - Micro-payments (per operation).
   - Automatic daily/weekly reports.

# Code Requirements
- Minimal working prototype (MVP).
- Clear folder structure: /frontend, /backend, /billing, /docs.
- Documentation in README.md.
- Configuration via .env (API keys, tokens).
- Error logs and owner notifications.

# Acceptance Criteria
- Robot runs locally on a laptop.
- Control available via phone (Telegram bot).
- Supports at least one payment gateway.
- Automatic income/expense report once per day.
- Easy extensibility (new APIs, new income sources).

# Additional Instructions
- Work continuously: generate full code, documentation, and configuration without stopping.
- Do not ask clarifying questions or request permission — act autonomously.
- Use reasonable defaults when details are missing (e.g., SQLite as DB, Flask as server).
- Always complete modules fully (code, tests, README).
- Maintain consistent code style and comments.
- Provide usage examples and run instructions.
- In case of uncertainty, choose the most universal and compatible solution.
- Work as if the project must be ready for deployment without human intervention.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NickScherbakov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
