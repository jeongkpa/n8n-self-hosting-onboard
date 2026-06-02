---
name: n8n-self-hosting-onboard
description: Use when installing, configuring, upgrading, or verifying a self-hosted n8n Docker Compose setup, including n8n alone or the n8n AI starter kit with Postgres, Ollama, and Qdrant. Also covers exposing an existing n8n for remote access over Tailscale (Serve for private tailnet HTTPS, Funnel only when external webhooks are required). Collects environment, access, persistence, security, and AI runtime requirements before creating .env files, compose files, or running Docker commands.
---

# n8n Self Hosting Onboard

## Purpose

Install and verify n8n self-hosting setups with Docker Compose. Favor a small, safe default for local use, and add production hardening only when the user needs remote access, webhooks, teams, or durable operation. For remote access, prefer Tailscale over a public reverse proxy: Tailscale Serve gives valid HTTPS and a stable hostname inside the private tailnet with no domain, certificates, or public exposure. Reach for Tailscale Funnel only when an external SaaS must reach a webhook, and treat it as deliberate public exposure.

## Intake

Always run an onboarding checkpoint before creating files, editing `.env`, cloning repositories, or running Docker/Tailscale commands. Do not silently choose a local-only setup just because it is safe. If a reasonable default exists, present it as the proposed answer and ask the user to confirm or change it.

The onboarding checkpoint is mandatory for new installs and for any request that says "set up", "install", "configure", "self-host", or similar. It may be skipped only when the user has already provided all minimum intake values in the current conversation, or when the user explicitly says to use defaults without asking questions.

Ask the minimum intake as numbered choices, not as a free-form checklist. Each onboarding item must offer exactly three numbered options (`1`, `2`, `3`) with option `1` as the recommended default when a default is safe. Use short labels and one-line tradeoffs. If an item has more than three real variants, put the less common variants behind option `3` as "custom/advanced" and ask one follow-up after the user selects it. Keep recommended intake optional unless it changes the requested install, but include Tailscale/webhook choices in the choices because access mode changes security and URL configuration. Do not guess about external exposure, destructive overwrite, or existing data.

When defaults are appropriate, ask in this shape before proceeding:

```text
Before I create files or run Docker, choose one option for each item.
Reply like: 1-1, 2-2, 3-1, 4-1, 5-1, 6-1, 7-1, 8-2

1. Install path
   1) <default path> (recommended)
   2) ~/docker/n8n
   3) Custom path

2. Access mode
   1) Local only: http://localhost:5678 (recommended)
   2) Tailscale Serve: private HTTPS inside tailnet
   3) Public webhook/LAN/custom: requires a follow-up security check

3. Host environment
   1) <detected OS/hardware> (recommended)
   2) Remote Linux/VPS
   3) NAS/Windows WSL/custom

4. Stack
   1) n8n + Postgres (recommended)
   2) n8n only
   3) AI starter kit: Postgres + Ollama + Qdrant

5. AI runtime
   1) No AI (recommended)
   2) Host Ollama or external LLM API
   3) Docker Ollama

6. Ports
   1) Defaults: n8n 5678, Ollama 11434, Qdrant 6333 (recommended)
   2) Change n8n port only
   3) Custom all relevant ports

7. Secrets
   1) Auto-generate strong secrets (recommended)
   2) I will provide secrets
   3) Preserve existing .env secrets

8. Backup expectation
   1) Manual backup instructions after install (recommended)
   2) Scheduled local backup
   3) Off-host backup plan
```

If the user replies with option numbers, resolve the selections and proceed without asking the same questions again. If the user picks a `custom`, `advanced`, public exposure, or existing-secret option, ask only the necessary follow-up question(s) for those selected items.

Minimum intake:

1. Install path: where repo or compose files should live.
2. Access mode: local only, LAN, Tailscale private (Serve), or Tailscale public webhook (Funnel).
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

Remote access intake (Tailscale):

- Webhook requirement: does an external SaaS need to call back into n8n? If no, Serve is enough; if yes, Funnel is required and means public exposure.
- Tailscale presence: is Tailscale already installed and logged in on the host (`tailscale status`), or does it need setup?
- Tailscale placement: install on the host (default for macOS, Linux, VPS), or run as a Docker sidecar when the host cannot run Tailscale (some NAS, restricted hosts).
- Tailnet hostname: confirm the machine name and tailnet so the canonical URL `https://<machine>.<tailnet>.ts.net` is known; MagicDNS must be enabled.
- Funnel eligibility (only if webhooks needed): tailnet ACL/policy permits Funnel, and the user accepts public reachability of the n8n endpoint.
- Extra access controls: rely on n8n user management plus tailnet ACLs; for Funnel, confirm whether basic auth or IP-independent controls are also wanted.

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
- Record where the encryption key is backed up; treat it as the single most important value to retain.

For Tailscale remote access:

- Default to Tailscale Serve (private tailnet HTTPS). Do not use Funnel unless an external webhook requires it.
- Keep n8n bound to localhost or the Docker network; let Tailscale terminate TLS and proxy to `5678`. Do not publish `5678` to `0.0.0.0`.
- Use the MagicDNS hostname `https://<machine>.<tailnet>.ts.net` as the canonical URL; let Tailscale provision the certificate.
- Set `N8N_HOST`, `N8N_PROTOCOL=https`, `WEBHOOK_URL=https://<machine>.<tailnet>.ts.net/`, and `N8N_PROXY_HOPS=1` so n8n trusts the forwarded TLS headers.
- Prefer host-installed Tailscale; use a Docker sidecar only when the host cannot run Tailscale.

## Workflow

1. Run the mandatory onboarding checkpoint.
   - Present the minimum intake as numbered `1/2/3` choices with proposed defaults.
   - Wait for confirmation or corrections before creating files, cloning repositories, writing `.env`, or running Docker/Tailscale commands.
   - If the user already supplied all minimum intake values or explicitly asked to use defaults without questions, state that the checkpoint is satisfied and proceed.

2. Inspect existing state.
   - Check whether the install path exists and whether it is empty, a git repo, or contains compose volumes/config.
   - Run `docker --version` and `docker compose version`.
   - Check port availability when using defaults.

3. Decide stack.
   - Local AI starter kit: clone or update `n8n-io/self-hosted-ai-starter-kit`.
   - Existing project: read current compose/env before editing.
   - Remote access: plan Tailscale Serve (or Funnel if external webhooks are needed), the `ts.net` URL, webhook URL, backup, and version pinning before running containers.

4. Create configuration.
   - Create `.env` from `.env.example` when available.
   - Replace demo secrets; never keep `password`, `super-secret-key`, or `even-more-secret`.
   - Preserve existing `N8N_ENCRYPTION_KEY` when data already exists. Changing it can make credentials unreadable.
   - Set `GENERIC_TIMEZONE` or `TZ` when the compose file supports it; otherwise note the gap.
   - For Tailscale remote access, set `N8N_HOST=<machine>.<tailnet>.ts.net`, `N8N_PROTOCOL=https`, `WEBHOOK_URL=https://<machine>.<tailnet>.ts.net/`, and `N8N_PROXY_HOPS=1`; keep n8n on `5678` unpublished to the public interface and let Tailscale proxy to it.

5. Start services.
   - Use the smallest applicable command, usually `docker compose up -d`.
   - For the AI starter kit, include the selected profile: `cpu`, `gpu-nvidia`, or `gpu-amd`.
   - Do not run destructive cleanup commands unless the user explicitly asks to reset data.

6. Verify.
   - `docker compose ps` shows n8n and dependencies running, with Postgres healthy where applicable.
   - `curl -I` or `curl -L` reaches n8n setup or signin.
   - For Ollama, `/api/tags` lists the expected model or model pull is still in progress.
   - Check recent logs for n8n startup, database connection, and obvious errors.
   - For Tailscale, confirm `tailscale serve status` maps the `ts.net` host to `5678`, then reach `https://<machine>.<tailnet>.ts.net/setup` from another tailnet device. For Funnel, confirm `tailscale funnel status` and that the webhook URL responds from outside the tailnet.

7. Report concise handoff.
   - Access URL: `http://localhost:5678` for local, or `https://<machine>.<tailnet>.ts.net` for Tailscale.
   - Install path.
   - Start, stop, status, and logs commands.
   - Secrets/persistence location without printing full secrets, and confirmation that the encryption key is backed up off-host.
   - Backup status: how to dump the database and where the encryption key is stored; flag if no backup exists yet.
   - Remaining production gaps, especially backups, upgrades, and whether Funnel (public exposure) is active.

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

# Upgrade: back up first (see Backup and restore), then pull and recreate
docker compose pull && docker compose up -d
```

Tailscale remote access (host-installed Tailscale):

```bash
# One-time, if not already on the tailnet
tailscale up

# Identify the canonical URL (MagicDNS must be enabled)
tailscale status --json | jq -r '.Self.DNSName'   # -> machine.tailnet.ts.net.

# Private tailnet HTTPS (default) — proxy ts.net :443 to local n8n :5678
tailscale serve --bg 5678
tailscale serve status

# Public webhook exposure (ONLY when an external SaaS must call in)
tailscale funnel --bg 5678
tailscale funnel status

# Stop exposure
tailscale serve --https=443 off
tailscale funnel --https=443 off
```

After enabling Serve/Funnel, set `WEBHOOK_URL` and `N8N_HOST` to the `ts.net` host and restart n8n so generated webhook URLs match.

Backup and restore:

```bash
# Postgres logical dump (n8n + Postgres / AI starter kit). User and db come from .env.
docker compose exec -T postgres pg_dump -U "$POSTGRES_USER" "$POSTGRES_DB" \
  > n8n-db-$(date +%F).sql

# Restore a dump into a running, empty Postgres
cat n8n-db-<date>.sql | docker compose exec -T postgres psql -U "$POSTGRES_USER" "$POSTGRES_DB"

# Back up the encryption key OFF-host (password manager / secret store) — required to read credentials
grep '^N8N_ENCRYPTION_KEY=' .env

# SQLite-only installs: snapshot the n8n data volume instead of pg_dump
docker run --rm -v <n8n_data_volume>:/data -v "$PWD":/backup alpine \
  tar czf /backup/n8n-data-$(date +%F).tgz -C /data .
```

A complete backup is the database dump (or data-volume snapshot) **plus** the encryption key stored separately. Restoring the database without the original key leaves saved credentials undecryptable.

## Safety Rules

- Never overwrite an existing `.env` without reading it first.
- Never regenerate `N8N_ENCRYPTION_KEY` for an existing install unless the user explicitly accepts credential loss.
- Back up `N8N_ENCRYPTION_KEY` to a location separate from the install directory (password manager or secret store). Losing it makes every stored credential unreadable even with a full database backup, so a database dump alone is not a sufficient backup.
- Take a backup (database dump plus encryption key) before any upgrade or any change that touches `.env` secrets.
- Never run `docker compose down -v`, remove volumes, or delete install directories as part of ordinary install/upgrade.
- Do not expose Postgres, Qdrant, or Ollama publicly unless the user explicitly asks and security controls are planned.
- For remote n8n, require HTTPS and a correct `WEBHOOK_URL` before claiming webhook readiness; with Tailscale Serve the `ts.net` HTTPS endpoint satisfies this inside the tailnet.
- Default remote access to Tailscale Serve (private tailnet). Never enable Tailscale Funnel without explicit user confirmation — Funnel publishes n8n to the public internet, so treat it like opening a public port.
- Do not publish n8n's `5678` to `0.0.0.0` when fronting it with Tailscale; let Tailscale proxy to localhost or the Docker network.
- For production, prefer pinned image tags over `latest` and call out backup gaps.

## Troubleshooting

- `curl /` returns 404 but `/setup` works: n8n is up; use `/setup` or the editor URL.
- n8n starts before import finishes: wait for import container to complete, then re-check n8n.
- Ollama model missing: inspect `ollama-pull-llama` logs or run `docker compose exec ollama ollama pull <model>`.
- Port conflict: adjust host-side port mapping, then update the handoff URL.
- Mac performance is poor with Docker Ollama: install Ollama on the host and set `OLLAMA_HOST=host.docker.internal:11434`.
- Python task runner warning in n8n image: note it if seen; it is usually not blocking for standard JS workflows.
- Tailscale `ts.net` URL not resolving: MagicDNS must be enabled in the tailnet admin console; without it, fall back to the tailnet IP (`100.x.y.z:5678`).
- Webhooks show `localhost` or a wrong host: `WEBHOOK_URL`/`N8N_HOST` were not updated to the `ts.net` host; fix the env and restart n8n.
- n8n login loop or "secure cookie" errors behind Tailscale Serve: ensure `N8N_PROTOCOL=https` and `N8N_PROXY_HOPS=1` so n8n trusts the forwarded TLS.
- External SaaS webhook never fires while only Serve is enabled: Serve is tailnet-private; the callback needs Funnel (public) — confirm with the user before enabling.
- `tailscale serve` fails with HTTPS/cert error: enable HTTPS certificates for the tailnet and confirm the node is logged in (`tailscale status`).
- After a restore, credentials fail to decrypt or workflows error on saved auth: the restored database was paired with a different `N8N_ENCRYPTION_KEY`. Set `.env` back to the original key and restart; the key must match the dump.
