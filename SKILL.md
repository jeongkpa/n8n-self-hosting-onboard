---
name: n8n-self-hosting-onboard
description: Use when installing, configuring, upgrading, or verifying a self-hosted n8n Docker Compose setup, including n8n alone or the n8n AI starter kit with Postgres, Ollama, and Qdrant. Collects environment, access, persistence, security, and AI runtime requirements before creating .env files, compose files, or running Docker commands.
---

# n8n Self Hosting Onboard

## Purpose

Install and verify n8n self-hosting setups with Docker Compose. Favor a small, safe default for local use, and add production hardening only when the user needs remote access, webhooks, teams, or durable operation.

## Intake

Ask only for missing information that changes the install. Use reasonable defaults for low-risk local installs, but do not guess about external exposure, destructive overwrite, or existing data.

Minimum intake:

1. Install path: where repo or compose files should live.
2. Access mode: local only, LAN, private tunnel, or public domain.
3. Host environment: macOS, Linux, Windows WSL, NAS, VPS, or cloud VM; note Apple Silicon.
4. Stack shape: n8n only, n8n + Postgres, or AI starter kit with Postgres + Ollama + Qdrant.
5. AI runtime: no AI, Docker Ollama, host Ollama, or external LLM API.
6. Ports: confirm defaults or choose alternatives for n8n `5678`, Ollama `11434`, and Qdrant `6333`.
7. Secrets: user-provided or auto-generated `POSTGRES_PASSWORD`, `N8N_ENCRYPTION_KEY`, and `N8N_USER_MANAGEMENT_JWT_SECRET`.

Recommended intake:

- Purpose: local experiment, personal automation, team internal use, external webhooks, or production.
- Timezone, default `Asia/Seoul` when the user is in Korea or the system timezone says so.
- Persistence preference: named Docker volumes or host bind mounts for easier backup.
- Version policy: pin n8n image for production; `latest` is acceptable for local experiments.
- Backup expectations: none, manual, scheduled local dump, or off-host backup.
- SMTP need: password reset, invitations, and notifications.

Remote/public intake:

- Canonical URL, for example `https://n8n.example.com`.
- Reverse proxy or tunnel: Caddy, Traefik, Nginx Proxy Manager, Cloudflare Tunnel, Tailscale, or existing ingress.
- TLS responsibility: proxy-managed, platform-managed, or not yet configured.
- Webhook requirement: required for external SaaS callbacks or not.
- Extra access controls: basic auth, IP allowlist, VPN-only, SSO, or n8n user management only.

## Defaults

For a local single-user AI starter kit:

- Use the official repository `https://github.com/n8n-io/self-hosted-ai-starter-kit.git`.
- Use `docker compose --profile cpu up -d`.
- Generate strong random secrets with `openssl rand -hex`.
- Keep Postgres internal to the compose network.
- Leave first owner account creation to the n8n web UI.
- Verify n8n at `http://localhost:5678/setup`.
- Verify Ollama at `http://localhost:11434/api/tags`.
- Treat Docker Ollama on Mac as CPU-oriented. For better Apple Silicon performance, prefer host Ollama and set `OLLAMA_HOST=host.docker.internal:11434`.

For n8n-only local installs:

- Use n8n with Postgres unless the user explicitly wants SQLite.
- Avoid exposing Postgres ports by default.
- Keep binary data on filesystem via `N8N_DEFAULT_BINARY_DATA_MODE=filesystem`.

## Workflow

1. Inspect existing state.
   - Check whether the install path exists and whether it is empty, a git repo, or contains compose volumes/config.
   - Run `docker --version` and `docker compose version`.
   - Check port availability when using defaults.

2. Decide stack.
   - Local AI starter kit: clone or update `n8n-io/self-hosted-ai-starter-kit`.
   - Existing project: read current compose/env before editing.
   - Production/public: plan proxy, URL, webhook URL, backup, and version pinning before running containers.

3. Create configuration.
   - Create `.env` from `.env.example` when available.
   - Replace demo secrets; never keep `password`, `super-secret-key`, or `even-more-secret`.
   - Preserve existing `N8N_ENCRYPTION_KEY` when data already exists. Changing it can make credentials unreadable.
   - Set `GENERIC_TIMEZONE` or `TZ` when the compose file supports it; otherwise note the gap.
   - For public access, set `N8N_HOST`, `N8N_PROTOCOL=https`, and `WEBHOOK_URL` when supported by the compose pattern.

4. Start services.
   - Use the smallest applicable command, usually `docker compose up -d`.
   - For the AI starter kit, include the selected profile: `cpu`, `gpu-nvidia`, or `gpu-amd`.
   - Do not run destructive cleanup commands unless the user explicitly asks to reset data.

5. Verify.
   - `docker compose ps` shows n8n and dependencies running, with Postgres healthy where applicable.
   - `curl -I` or `curl -L` reaches n8n setup or signin.
   - For Ollama, `/api/tags` lists the expected model or model pull is still in progress.
   - Check recent logs for n8n startup, database connection, and obvious errors.

6. Report concise handoff.
   - Access URL.
   - Install path.
   - Start, stop, status, and logs commands.
   - Secrets/persistence location without printing full secrets.
   - Remaining production gaps, especially HTTPS, backups, upgrades, and public exposure.

## Commands

Typical local AI starter kit:

```bash
git clone https://github.com/n8n-io/self-hosted-ai-starter-kit.git <install-path>
cd <install-path>
cp .env.example .env
docker compose --profile cpu up -d
```

Verification:

```bash
docker compose ps
curl -L --max-time 15 http://localhost:5678/setup
curl --max-time 10 http://localhost:11434/api/tags
docker compose logs --tail=80 n8n
```

Operations:

```bash
docker compose up -d
docker compose down
docker compose logs -f n8n
docker compose pull && docker compose up -d
```

## Safety Rules

- Never overwrite an existing `.env` without reading it first.
- Never regenerate `N8N_ENCRYPTION_KEY` for an existing install unless the user explicitly accepts credential loss.
- Never run `docker compose down -v`, remove volumes, or delete install directories as part of ordinary install/upgrade.
- Do not expose Postgres, Qdrant, or Ollama publicly unless the user explicitly asks and security controls are planned.
- For public n8n, require HTTPS and a correct `WEBHOOK_URL` before claiming webhook readiness.
- For production, prefer pinned image tags over `latest` and call out backup gaps.

## Troubleshooting

- `curl /` returns 404 but `/setup` works: n8n is up; use `/setup` or the editor URL.
- n8n starts before import finishes: wait for import container to complete, then re-check n8n.
- Ollama model missing: inspect `ollama-pull-llama` logs or run `docker compose exec ollama ollama pull <model>`.
- Port conflict: adjust host-side port mapping, then update the handoff URL.
- Mac performance is poor with Docker Ollama: install Ollama on the host and set `OLLAMA_HOST=host.docker.internal:11434`.
- Python task runner warning in n8n image: note it if seen; it is usually not blocking for standard JS workflows.
