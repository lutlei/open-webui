---
Last updated: 2026-05-02
---

# Agent context — Open WebUI Dev

This file tells agents where to find authoritative environment and deployment information for this repo and the deployed instance.

## Information availability

- **[ENVIRONMENT.md](./ENVIRONMENT.md)** (this repo, root) — **Primary reference.** Contains:
  - What this fork is (lutlei/open-webui, upstream, branches, version strategy, how to deploy from the fork).
  - Where it is deployed (Proxmox CT 111, IP, service, paths, port, Ollama URL, public and local URLs).
  - Important operational notes (LAN-only Ollama, restart, updates, backup, healthcheck, docs location).

- **Cursor skill: openwebui-dev** — Canonical source: **agentic-skillforge** at `openwebui-dev/SKILL.md`. This repo and **stacknotes** use a symlink at `.cursor/skills/openwebui-dev` → that directory (same content everywhere).  
  Use when handling Open WebUI dev: deployment table, SSH/service commands, Ollama connectivity, backup, and pointers to stacknotes docs (VMID 111 folder, Cloudflare, firewall/Tailscale). If you are in the openwebui-dev repo and need deployment or homelab context, load this skill or read ENVIRONMENT.md.

- **Full planning and homelab docs** — In the **stacknotes** repository:
  - `infrastructure/proxmox/Containers/VMID 111 - OpenWebUi (DEV) Container/` (README, Setup-Guide, OpenWebUI-Releases-0.8-and-Plan, Cloudflare-and-Ollama-Integration, **Open-Terminal-Agent-Implementation-Guide.md**, **Open-Terminal-Security-and-Setup.md**, installation scripts).
  - Proxmox host and firewall/Tailscale: `infrastructure/proxmox/proxmox host/` (e.g. Tailscale-and-Network-Vision, LAN-to-VM102-Diagnostic-Plan) and **homelab-firewall-tailscale** skill for connectivity issues.

When working on Open WebUI dev (this repo or the deployed instance), start from **ENVIRONMENT.md** and the **openwebui-dev** skill so agents have consistent access to fork, deployment, and “important to know” information.

**Open Terminal:** To **implement** Open Terminal on CT 111 and surface it in Open WebUI, follow **`infrastructure/proxmox/Containers/VMID 111 - OpenWebUi (DEV) Container/Open-Terminal-Agent-Implementation-Guide.md`** (SSH steps, Docker `slim` on `127.0.0.1:8000`, API key, Admin → Integrations). Background and security rationale: **`Open-Terminal-Security-and-Setup.md`** in the same folder. Official: [Open Terminal – Security](https://docs.openwebui.com/features/open-terminal/advanced/security/).
