# Copilot Instructions — Pusdatik SPBE RAG

Instruksi ini mengaktifkan **Superpowers skills** dan konfigurasi proyek untuk GitHub Copilot.

---

## 🚀 Superpowers Skills Library

<EXTREMELY_IMPORTANT>
Superpowers adalah library skills yang powerful untuk Claude Code. Gunakan skills ini untuk:
- **TDD Workflows** — Test-driven development patterns
- **Debugging** — Systematic debugging dengan scientific method
- **Collaboration** — Proven collaboration patterns
- **Code Quality** — Best practices dan refactoring techniques

Superpowers skills tersedia di: `~/.copilot/superpowers/skills/`
</EXTREMELY_IMPORTANT>

### Cara Menggunakan Superpowers

Untuk menggunakan skill dari superpowers, minta secara eksplisit:

```
Use the superpowers:tdd skill to write tests for this feature

Use the superpowers:debugging skill to debug this error

Use the superpowers:refactoring skill to improve this code
```

Daftar skill utama yang tersedia:
- `superpowers:tdd` — Test-driven development workflow
- `superpowers:debugging` — Debugging techniques
- `superpowers:refactoring` — Code refactoring patterns
- `superpowers:collaboration` — Collaboration workflows
- `superpowers:code-review` — Code review patterns

---

## 🎯 Proyek Konfigurasi — SPBE RAG System

### Project Vision
Sistem **Retrieval-Augmented Generation** untuk menjawab pertanyaan tentang peraturan SPBE dan hasil audit keamanan BSSN. Dibangun untuk **Badan Siber dan Sandi Negara (BSSN)**.

### Tech Stack
- **Backend**: FastAPI (Python) + LlamaIndex + Qdrant + SQLite
- **Frontend**: Vue.js 3 + Vite + Vanilla CSS
- **AI/ML**: Qwen 2.5 7B (GGUF Q4_K_M), indo-sentence-bert-base, BAAI/bge-reranker-base
- **OCR**: PaddleOCR (GPU)
- **Infrastructure**: Docker Compose + NVIDIA Docker
- **Evaluation**: RAGAS + BUS-11

### Konvensi Kode
- **Python**: PEP 8, type hints, loguru untuk logging
- **Vue.js**: Composition API (`<script setup>`), Vanilla CSS
- **Naming**: snake_case (Python), camelCase (JS), kebab-case (files/CSS)
- **Error Handling**: Always handle exceptions, log dengan loguru
- **Bahasa**: Code comments dalam English, user-facing strings dalam Bahasa Indonesia

### File Kritis (Extra Hati-Hati saat Modifikasi)
- `backend/app/core/rag/langchain_engine.py` — RAG engine utama (~61KB)
- `backend/app/core/ingestion/document_manager.py` — Document lifecycle (~67KB)
- `backend/app/core/rag/prompts.py` — System prompts
- `backend/app/api/routes/chat.py` — Chat endpoint (streaming SSE)
- `backend/app/core/ingestion/structured_chunker.py` — Chunking strategy

### Testing Requirements
- Jalankan existing tests sebelum modifikasi file kritis: `python -m pytest tests/ -v`
- Gunakan TDD approach (test dulu, implementasi)
- Test RAG changes dengan `backend/scripts/rag_trace.py`

### Environment Constraints
- **GPU**: NVIDIA GTX 1650 (4GB VRAM) — model harus quantized
- **RAM**: 16GB total
- **Python**: 3.10+ dengan venv di `backend/venv/`
- **Node**: Untuk frontend di `frontend/`
- **Network**: Internal BSSN — no external API calls

---

## 📋 Model Delegation Strategy

### 🟡 Sonnet — Default Coder
Sonnet adalah conductor utama untuk:
- Coding & implementation (Python, Vue.js, CSS, JavaScript)
- Bug fixing & feature development
- API routes & database operations
- Test writing & quick refactoring
- Docker/Config & Git operations

**Prinsip**: Act first, iterate fast.

### 🔴 Opus — Heavy Analysis
Delegasikan ke Opus untuk:
- Architecture design & redesign RAG pipeline
- Complex multi-component debugging
- Performance analysis & algorithm design
- Security review (BSSN project!)
- Data model design & evaluation framework

### 🟢 Haiku — Lightweight Tasks
Gunakan Haiku untuk:
- Dokumentasi & docstrings
- Boilerplate code & test data
- Changelog & config generation
- Simple search & replace tasks

---

## 🔗 Referensi Lanjut

Untuk informasi lebih detail, lihat:
- `.claude/CLAUDE.md` — Konfigurasi agent delegation
- `SETUP_GUIDE.md` — Panduan setup backend/frontend
- `QUICKSTART.md` — Quick reference commands
- `README.md` — Dokumentasi proyek

---

## 💡 Tips & Tricks

1. **Gunakan Superpowers untuk TDD**: Minta Copilot menggunakan `superpowers:tdd` skill untuk menulis test terlebih dahulu
2. **Debugging Sistematis**: Gunakan `superpowers:debugging` untuk investigasi bug yang kompleks
3. **Code Review Pattern**: Gunakan `superpowers:code-review` sebelum commit
4. **Referensi File**: Gunakan `@workspace` untuk memberikan konteks proyek penuh
5. **RAG Testing**: Selalu test RAG changes dengan rag_trace.py setelah perubahan

---

**Last Updated**: May 2026 | Framework: GitHub Copilot Chat | Superpowers v5.1.0

---
> Source: [aqilalfa/TA_PUSDATIK](https://github.com/aqilalfa/TA_PUSDATIK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-12 -->
