# GitHub Copilot Instructions for Criterion

## Project Overview

Criterion is an AI-powered Islamic Da'i (guide) chatbot built with Next.js 16, featuring dual-source RAG (Retrieval Augmented Generation) over the Quran (6,236 verses) and Hadith (21,641 narrations from 6 major collections). The system uses tool-based AI with XAI Grok 4 LLM for autonomous retrieval and real-time streaming responses.

## Architecture & Data Flow

**Core Pattern**: User query → LLM tool selection → Vector search → Context enhancement → Response with citations

- **Frontend**: Next.js 16 App Router, React 19, Tailwind CSS, real-time SSE streaming
- **AI Stack**: XAI Grok 4 (chat + reasoning), Google Gemini text-embedding-004 (768 dims), Vercel AI SDK
- **Database**: PostgreSQL + pgvector (HNSW indexes), Drizzle ORM
- **Key Components**: Tool-based RAG, semantic search, multilingual reading (English + Slovak), shareable URLs

## Important: How to code effectively

- KISS: Keep It Simple, Stupid - avoid over-engineering
- DRY: Don't Repeat Yourself - reuse existing functions and components
- Follow established patterns for RAG, tool usage, and context
- Principle of Least Astonishment: Code should behave as expected
- Avoid verbosity: concise and clear code is better
- Use context7 when looking for documentation on libraries and frameworks
- Do not reinvent the wheel - leverage existing solutions and libraries

DO NOT: create summaries after every task unless specifically asked for.

---
> Source: [BalajSaleem/criterion](https://github.com/BalajSaleem/criterion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
