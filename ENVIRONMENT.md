---
Last updated: 2026-05-02
---

# Open WebUI Dev — environment

This repo is the **local clone of your fork** used for development and to align with the instance deployed on the homelab. Read this for fork context, deployment details, and important operational notes.

---

## This is a fork of Open WebUI

- **Fork:** [lutlei/open-webui](https://github.com/lutlei/open-webui) — your fork. **Origin** points here.
- **Upstream:** [open-webui/open-webui](https://github.com/open-webui/open-webui). Added as remote **upstream** for pulling new releases.
- **Branches (examples):** `main`, `async-db`, `dev`, `backend-outlet`, `group-members`, `update-user-table`. Use your fork when you want to **customize** (graphics, branding, features) and merge/rebase from upstream to get new features; use upstream tags only for testing releases with no customizations.
- **Version strategy:** Upstream 0.8.x (deployed tag on CT 111: **v0.8.12** as of 2026-05-02) has PostgreSQL support and DB migrations. Back up `/home/openwebui/.open-webui` before major upgrades. No rolling updates for multi-worker — plan for downtime on schema changes.
- **Deploy from fork:** On the container, set `GIT_REPO_URL=https://github.com/lutlei/open-webui.git` and `GIT_BRANCH=<your-branch>` (e.g. `async-db`) when (re)running the install/update; then rebuild and restart the service.

---

## It is deployed on

| Item | Value |
|------|--------|
| **Platform** | Proxmox VE LXC container |
| **VMID** | 111 |
| **Hostname** | openwebui-dev |
| **IP** | 192.168.86.18/24 (LAN only; no Tailscale in container) |
| **SSH** | `ssh pve-openwebui-dev` (root). From stacknotes docs: direct to 192.168.86.18 or via ProxyJump `pve-vm199` if no subnet routes. |
| **Service** | `open-webui-dev` (systemd) |
| **Install dir** | `/opt/open-webui` (app code; owner `openwebui:openwebui`) |
| **Data dir** | `/home/openwebui/.open-webui` (SQLite + embedded ChromaDB) |
| **Env file** | `/home/openwebui/.env` (600; OLLAMA_BASE_URL, WEBUI_PORT, etc.) |
| **Port** | 8081 (production OpenWebUI uses 8080) |
| **Ollama** | VM 102 at **192.168.86.11:11434** (LAN). `OLLAMA_BASE_URL=http://192.168.86.11:11434`, `ENABLE_OLLAMA_API=true`. |
| **Public URL** | **https://webui.kordanda.org** (Cloudflared on CT 105 → `http://192.168.86.18:8081`). Zero Trust protected. |
| **Local URL** | http://192.168.86.18:8081 |
| **Upstream tag (CT 111)** | **v0.8.12** (`open-webui/open-webui` clone on the container; not your fork unless you change `origin` / reinstall with `GIT_REPO_URL`) |

**Proxmox host:** 192.168.86.27. Container created with 200GB on vmdata-lvm, 4 cores, 4GB RAM, 2GB swap, unprivileged. Full planning and setup: see **stacknotes** repo → `infrastructure/proxmox/Containers/VMID 111 - OpenWebUi (DEV) Container/` (README, Setup-Guide, OpenWebUI-Releases-0.8-and-Plan, Cloudflare-and-Ollama-Integration).

---

## This is important to know

- **LAN-only for Ollama:** The container does **not** run Tailscale. It reaches Ollama at 192.168.86.11 over the LAN. VM 102 must use `tailscale set --accept-routes=false` so it answers on the LAN interface; see stacknotes homelab-firewall-tailscale skill and Tailscale-and-Network-Vision if .11 is unreachable.
- **Restart after code/config changes:** `systemctl restart open-webui-dev` (on CT 111). Env changes in `/home/openwebui/.env` require a restart.
- **Updates from your fork:** On the container: `cd /opt/open-webui`, `git fetch origin`, `git checkout <branch>`, `git pull`, then rebuild (e.g. `npm run build`, reinstall Python deps if needed) and `systemctl restart open-webui-dev`. Back up `/home/openwebui/.open-webui` before major version upgrades.
- **Pulling upstream into your branch:** In this repo: `git fetch upstream`, `git merge upstream/main` (or rebase), resolve conflicts, push to origin; then on the container pull and rebuild as above.
- **Health check:** Service uses a pre-start healthcheck script that verifies Ollama is reachable; if Ollama is down, the service may fail to start (by design).
- **Frontend path:** When running from the repo (e.g. `/opt/open-webui`), the app uses `FRONTEND_BUILD_DIR` for the built frontend. The systemd unit sets `FRONTEND_BUILD_DIR=/opt/open-webui/build` so `/` serves the UI; without it the app runs API-only and `/` returns 404. If you recreate the unit or use `.env`, set `FRONTEND_BUILD_DIR=/opt/open-webui/build`.
- **Upgrading to recent 0.8.x (e.g. v0.8.10):** `uv pip install -e ".[all]"` may fail on `ddgs==9.11.2` (missing on PyPI) — use `ddgs==9.11.4` in `pyproject.toml` or rely on **customized_installation_script.sh** (auto-patches after clone). Install **`libmariadb-dev`** (`apt`) before `uv pip` because `[all]` builds the `mariadb` wheel. See stacknotes **OpenWebUI-Releases-0.8-and-Plan.md** §1b.
- **Cloudflare cache after fixes:** The public URL (https://webui.kordanda.org) is behind Cloudflare. If the site was previously broken (e.g. API-only), Cloudflare may have cached 404s for `/` or `/static/*`. After fixing the origin, purge the cache for webui.kordanda.org (Caching → Purge Cache, or Custom Purge for `/` and `/static/loader.js`, `/static/custom.css`, `/static/splash.png`, `/static/splash-dark.png`) and hard-refresh the page.
- **Documentation and skills:** Full deployment, Cloudflare, and firewall/connectivity details live in the **stacknotes** repository. The **openwebui-dev** agent skill is maintained in **agentic-skillforge** (`openwebui-dev/SKILL.md`); stacknotes and this repo symlink `.cursor/skills/openwebui-dev` to that folder so one source stays canonical.
- **Open Terminal (optional):** Canonical security guidance is [Open Terminal – Security](https://docs.openwebui.com/features/open-terminal/advanced/security/) (official checklist). Multi-user vs production: [Multi-user setup](https://docs.openwebui.com/features/open-terminal/advanced/multi-user/) and [open-webui/terminals](https://github.com/open-webui/terminals) (enterprise). See stacknotes **`Open-Terminal-Security-and-Setup.md`** for VM 111 mapping. **To deploy Open Terminal on CT 111 and enable it in Open WebUI, agents should follow `Open-Terminal-Agent-Implementation-Guide.md`** in the same VMID 111 folder (Docker `slim`, localhost, admin integration).
