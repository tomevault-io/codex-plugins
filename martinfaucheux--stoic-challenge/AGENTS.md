## Introduction

We want to build an **Email Security** tool to flag emails with potential financial fraud attempts (fake president, fake supplier, etc.) that are received by our clients. We focus on clients that are using Google Workspace and Microsoft O365 so users and emails can be retrieved through relevant APIs.

The goal is to code the service retrieving the emails.

## Requirements

- Build the service in Python with FastAPI
- Use PostgreSQL
  - use SQLAlchemy as the ORM
  - use Alembic for migrations
- Dockerize the service
- Tests using pytest

# Notes

each command should be run with uv. For example

- uv run pytest
- uv run alembic revision --autogenerate -m "create email table"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martinfaucheux) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
