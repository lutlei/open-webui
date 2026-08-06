# AGENTS.md

This is a fork of [open-webui/open-webui](https://github.com/open-webui/open-webui) — a SvelteKit frontend + FastAPI (Python) backend. Upstream docs live at [docs.openwebui.com](https://docs.openwebui.com); standard scripts are in `package.json` (frontend) and `backend/dev.sh` / `backend/start.sh` (backend). The homelab deployment of this fork is covered by the `openwebui-dev` skill in the `stacknotes` repo.

## Cursor Cloud specific instructions

The app runs **fully self-contained** in the cloud VM with SQLite + embedded Chroma — no external services, no secrets required. Docker is not available; run the two processes directly.

- **Node:** the frontend has `engine-strict=true` and `engines.node <=22.x`, so it **must** use Node ≤22. The VM default `node` (`/exec-daemon/node`, v22.x) satisfies this — do **not** switch to Node 24 for this repo.
- **Backend (port 8080):** a venv already exists at `backend/.venv` (created with the system `python3.12-venv`; deps from `backend/requirements.txt`, which pulls torch/sentence-transformers/chromadb — a large install). Run from the `backend/` dir:
  ```bash
  cd backend && source .venv/bin/activate
  export WEBUI_SECRET_KEY="dev-secret"          # dev.sh does NOT set this; startup exits without it
  export CORS_ALLOW_ORIGIN="http://localhost:5173;http://localhost:8080"
  uvicorn open_webui.main:app --host 0.0.0.0 --port 8080
  ```
  First start creates `backend/data/webui.db` and downloads the default embedding model (`all-MiniLM-L6-v2`) from HF. `GET /health` → `{"status":true}`.
  - If `backend/.venv` is ever missing (e.g. snapshot not retained), recreate it: `sudo apt-get install -y python3.12-venv`, then `python3 -m venv backend/.venv && backend/.venv/bin/pip install -r backend/requirements.txt`. (Not in the startup update script — too heavy/system-level.)
- **Frontend (port 5173):** `npm run dev`. This first runs `pyodide:fetch` (downloads Pyodide). In dev the Svelte app talks directly to the backend at `hostname:8080` (no Vite proxy), so the backend `CORS_ALLOW_ORIGIN` must include the frontend origin.
- **First-run / auth:** the **first account created becomes admin**. The animated onboarding splash ("Get started") can get stuck looping in a headless/automated browser. Reliable workarounds: create the first user via the API —
  `curl -X POST localhost:8080/api/v1/auths/signup -H 'Content-Type: application/json' -d '{"name":"Admin","email":"admin@example.com","password":"..."}'` — which flips the instance out of onboarding so `/` then shows a normal sign-in form.
- **LLM models:** none are configured by default (no Ollama/OpenAI key), so chat completions won't return responses, but account creation, the workspace, and the chat UI all work for smoke-testing.
