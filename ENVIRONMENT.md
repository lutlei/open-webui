---
Last updated: 2026-09-07
---

# Open WebUI — lutlei fork environment

Homelab clone of [`lutlei/open-webui`](https://github.com/lutlei/open-webui). Durable inventory and guest prose live in **stacknotes** (`$HOME/repositories/stacknotes`). This file is the short map for agents working in *this* repo.

## Remotes

| Remote | URL | Role |
|--------|-----|------|
| **origin** | `https://github.com/lutlei/open-webui.git` | Your fork — push custom branches here |
| **upstream** | `https://github.com/open-webui/open-webui.git` | Official project — fetch tags/releases only |

Do **not** push to `upstream`. Do **not** develop on a detached upstream tag on CT 111.

The fork’s GitHub tag `v0.11.3` was rewritten when `main` was synced (points at a later merge). The real upstream tag is commit `2a960a59f`. Prefer `git fetch upstream --tags` and `upstream` refs, not the fork tag, until that is fixed.

## Custom branch

| Item | Value |
|------|--------|
| **Branch** | `custom/hverdagsassistent` |
| **Base** | upstream `v0.11.3` (`2a960a59f`) |
| **Local path (Mac)** | `$HOME/repositories/openwebui-dev` |

Work on this branch (or further `custom/*` branches). Merge/rebase **upstream tags** into it when you want a new Open WebUI release.

## Deployment (CT 111)

| Item | Value |
|------|--------|
| **VMID** | 111, hostname `openwebui-dev` |
| **IP** | `192.168.86.18/24` |
| **SSH** | `ssh pve-openwebui-dev` (root; ProxyJump `pve-vm199` if LAN routes are down) |
| **Install dir** | `/opt/open-webui` |
| **Data dir** | `/home/openwebui/.open-webui` (SQLite + ChromaDB) — never commit this |
| **Env** | `/home/openwebui/.env` |
| **Service** | `open-webui-dev` |
| **Port** | 8081 |
| **Public URL** | https://webui.kordanda.org |
| **Guest docs** | `stacknotes/infrastructure/proxmox/Containers/VMID 111 - OpenWebUi (DEV) Container/` |

After pulling this branch on CT 111: rebuild frontend only if `src/` changed; backend-only patches need `systemctl restart open-webui-dev`.

## Sync upstream into the custom branch

```bash
git fetch upstream --tags
git merge v0.11.4   # use the tag you intend, not a rewritten fork tag
# resolve conflicts in *your* files, then:
git push origin custom/hverdagsassistent
```

On CT 111: `git fetch origin && git checkout custom/hverdagsassistent && git pull --ff-only origin custom/hverdagsassistent`, then rebuild/restart per stacknotes `openwebui-dev` skill.
