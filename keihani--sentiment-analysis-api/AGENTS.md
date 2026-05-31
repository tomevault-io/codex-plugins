Purpose: Short, concrete guidance to help AI coding agents be productive in this repository.

High-level architecture

- Single-process Flask app in `app.py`. The API and the model live in the same process.
- On startup `app.py` loads `data/sentiment_dataset.json` and trains an in-memory scikit-learn pipeline (TfidfVectorizer + LogisticRegression).
- `demo.html` is a static frontend that calls the API at `http://localhost:5000` (expects JSON responses).

Key files

- `app.py` — API routes, `SentimentModel` class, preprocessing rules, and startup training. Primary place to change behavior.
- `data/sentiment_dataset.json` — training data: an array of `[text, label]` pairs (not objects). Labels observed: `positive`, `negative`, `neutral`.
- `demo.html` — example client; expects fields: `sentiment`, `confidence`, `scores` in responses.
- `requirements.txt` — pinned Python deps (Flask, flask-cors, numpy, scikit-learn, gunicorn).
- `SETUP_GUIDE.md`, `test_api.py`, `Dockerfile` — present but empty; likely intended targets for tests, docs, and containerization.

Observable runtime behavior & constraints (do not assume changes)

- Training happens synchronously during import/startup in `app.py`. Large `data/` will slow startup and the model is not persisted to disk.
- Preprocessing: lowercasing, remove non-alphabetic chars, collapse whitespace (see `SentimentModel.preprocess_text`). This means punctuation and emojis are removed before vectorization.
- Vectorizer: `TfidfVectorizer(max_features=1000)` — vocabulary size limited to 1000 on fit.
- Classifier: `LogisticRegression(max_iter=1000)`; code uses `predict_proba` and `model.classes_` to build `scores`.
- Predictions will raise if `sentiment_model.trained` is False — but in this repo training runs at startup, so predict is normally safe.
- CORS is enabled globally via `flask_cors.CORS(app)`. The demo uses `http://localhost:5000` as `API_URL`.

API surface and example payloads

- GET `/` — returns metadata and available endpoints.
- GET `/health` — returns `{ status: 'healthy', model_trained: true/false }`.
- POST `/analyze` — request JSON: `{ "text": "..." }`.
  Response JSON (example):
  `{ "text":"...", "sentiment":"positive", "confidence":0.87, "scores":{"positive":0.87,"neutral":0.10,"negative":0.03} }`
- POST `/batch` — request JSON: `{ "texts": ["t1","t2"] }`. Responds with `count` and `results` array (same result objects as `/analyze`).

Quick local dev commands (Windows PowerShell)

```powershell
# create venv (optional)
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# install deps
pip install -r requirements.txt

# run dev server (debug mode) — uses Flask built-in server from app.py
python .\app.py

# production-style single-process using gunicorn (works if you prefer it)
# from repo root: expose app object from app.py as `app` (already present)
gunicorn --bind 0.0.0.0:5000 app:app -w 4
```

Notes for agents making edits

- When changing model training or vectorization, update expectations in `demo.html` and any tests — the frontend depends on `sentiment`, `confidence`, and `scores` keys.
- Do not assume model persistence — if you add a pickle save/load, document the path and update startup behavior and `SETUP_GUIDE.md`.
- `data/sentiment_dataset.json` is an array of pairs ([text,label]) not objects — maintain that shape when reading/writing.

Low-risk helpful improvements to consider (explicit suggestions agents can implement)

- Add lightweight unit tests to `test_api.py` (use `pytest` + `requests` or Flask test client) that exercise `/health`, `/analyze` and `/batch`.
- Populate `SETUP_GUIDE.md` with the venv and run steps above.
- Provide a minimal `Dockerfile` and mention the port and `gunicorn` entrypoint.

If anything above is unclear or missing examples you'd like (tests, Dockerfile, or a sample `model.pkl` flow), tell me which section to expand and I will update this file.

---
> Source: [keihani/sentiment-analysis-api](https://github.com/keihani/sentiment-analysis-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-31 -->
