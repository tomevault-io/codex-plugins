# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A live developer dashboard that streams trending GitHub repos and top HackerNews/Reddit posts in real-time. Built with Python, Socket.IO, MongoDB, and deployed on Render (not AWS — kept free and simple).

## Tech Stack

- **Backend**: Python (Flask + Flask-SocketIO)
- **Scheduler**: APScheduler — polls APIs every 5 minutes
- **Database**: MongoDB Atlas (M0 free tier) via PyMongo
- **Real-time**: Socket.IO — broadcasts updates to all connected clients
- **Frontend**: Single HTML file served by Python (vanilla JS, no framework)
- **Hosting**: Render.com free tier (not AWS — intentionally avoided to keep MVP simple)

## Data Sources (all free, no paid keys)

- **HackerNews** — official API, no key required
- **Reddit** — free API, requires simple app registration (`r/programming`, `r/python`)
- **GitHub Trending** — GitHub Search API (free) + `gtrending` scraper

## Architecture

```
Python backend
├── Scheduler (APScheduler) — runs fetch jobs every 5 min
│   ├── fetch_hackernews.py → HackerNews top stories
│   ├── fetch_reddit.py     → Reddit hot posts
│   └── fetch_github.py     → GitHub trending repos
├── MongoDB Atlas
│   └── Caches latest fetch per source + stores user bookmarks
├── Flask-SocketIO server
│   └── Broadcasts new data to all connected clients on update
└── Static frontend (single HTML file)
    ├── HackerNews panel
    ├── Reddit panel
    └── GitHub trending panel
```

## MVP Scope

HackerNews only — one data source, one panel, full pipeline working end to end:
- Fetch HackerNews top stories every 5 min via APScheduler
- Cache results in MongoDB
- Broadcast updates via Socket.IO
- Display in a simple frontend table (title, score, link)

Reddit and GitHub panels are added after MVP is confirmed working.

## Folder Structure

```
/
├── server/
│   ├── app.py              # Flask + SocketIO entry point
│   ├── scheduler.py        # APScheduler setup
│   ├── fetchers/
│   │   ├── hackernews.py
│   │   ├── reddit.py
│   │   └── github.py
│   └── db.py               # MongoDB connection + queries
├── frontend/
│   └── index.html          # Single-file frontend
├── tests/
│   └── smoke/              # Smoke checks per feature
├── requirements.txt
├── .env.example
├── spec.md
└── CLAUDE.md
```

## Coding Conventions

- Use `python-dotenv` for all secrets — never hardcode credentials
- Keep fetchers stateless — each fetcher returns a list of dicts, nothing else
- MongoDB documents follow a consistent schema: `{ source, title, url, score, fetched_at }`
- Emit Socket.IO events as `{ event: "update", source: "hackernews", data: [...] }`
- One file per fetcher — do not combine fetchers

## Rules for Claude

- **Verify every task before moving on** — each feature must pass a smoke check or test before the next one starts
- Do not write application code for phase 2+ until phase 1 (MVP) is confirmed working
- Do not add AWS or Terraform — hosting is Render.com
- Do not introduce frontend frameworks — keep the frontend as a single HTML file
- Always check `.env.example` is updated when adding new environment variables

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/da-b0ss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
