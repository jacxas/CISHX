# AGENTS.md — Base44 dev environment for Odysseus

## What this is

Odysseus is a self-hosted AI workspace (FastAPI + vanilla JS frontend) running on Python 3.14. The Base44 dev environment runs it from the cloned source with `uvicorn --reload` so edits appear live in the preview.

## Running the app

```bash
docker compose -f docker-compose.base44.yml up -d --build
```

- Web UI is served on **port 3000** (mapped from internal port 7000).
- Health check: `curl http://localhost:3000/api/health`
- Auth is **disabled** (`AUTH_ENABLED=false`) for dev convenience.
- SQLite database at `/app/data/app.db` (named volume `odysseus-data`).

## Services

| Service  | Image                              | Purpose                          |
|----------|------------------------------------|----------------------------------|
| odysseus | Built from `Dockerfile.base44-dev` | FastAPI app (live-reload)        |
| searxng  | `searxng/searxng:2026.5.31-...`    | Web search (metasearch engine)   |
| chromadb | `chromadb/chroma:latest`           | Vector store (RAG/embeddings)    |
| ntfy     | `binwiederhier/ntfy`               | Push notifications               |

## Dev image

`Dockerfile.base44-dev` installs system deps (build-essential, cmake, libgl1, libmagic1, etc.) and Python requirements for layer caching. The source is **not** copied — it's bind-mounted at `/app`. On startup, compose re-runs `pip install -r requirements.txt` (fast no-op if unchanged), then `python setup.py` (creates dirs, DB, admin), then `uvicorn app:app --reload`.

## External credentials (all optional)

The app boots without any external API keys. These are optional and only needed for specific features:
- `OPENAI_API_KEY` — OpenAI models
- `OLLAMA_BASE_URL` — local Ollama
- `DATA_BRAVE_API_KEY`, `TAVILY_API_KEY`, `SERPER_API_KEY`, `GOOGLE_API_KEY` — web search providers
- `HF_TOKEN` — HuggingFace downloads (Cookbook)

Set these via the Base44 secrets dashboard; they're delivered to `/run/base44/app.env`.

## Key files

- `app.py` — FastAPI app, middleware, lifespan, route mounting
- `core/` — auth, database, middleware, constants
- `src/` — service layer (chat, agents, email, embeddings, etc.)
- `routes/` — API route modules
- `static/` — frontend (vanilla JS, no build step)
- `setup.py` — first-time setup (dirs, DB, admin user)
- `config/searxng/settings.yml` — SearXNG config template

## Testing

```bash
docker compose -f docker-compose.base44.yml exec -T odysseus python -m pytest tests/ -x
```
