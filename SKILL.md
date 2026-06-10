---
name: n8n-self-hosting-onboard
description: Use when installing, configuring, upgrading, verifying, or operating a self-hosted n8n Docker Compose setup, including n8n alone, n8n + Postgres, or the n8n AI starter kit with Postgres, Ollama, and Qdrant. Also covers day-2 operations on a running install — backing up and restoring, automating backups, pruning execution data, monthly health checks, migrating SQLite to Postgres, moving n8n to another machine, and troubleshooting slow UI or full disks. Covers remote access over Tailscale (Serve for private tailnet HTTPS, Funnel only when external webhooks are required) and LAN access. Collects environment, access, persistence, security, and AI runtime requirements before creating .env files, compose files, or running Docker commands.
---

# n8n Self Hosting Onboard

## Purpose

Install, verify, and keep alive n8n self-hosting setups with Docker Compose. Favor a small, safe default for local use, and add production hardening only when the user needs remote access, webhooks, teams, or durable operation. For remote access, prefer Tailscale over a public reverse proxy: Tailscale Serve gives valid HTTPS and a stable hostname inside the private tailnet with no domain, certificates, or public exposure. Reach for Tailscale Funnel only when an external SaaS must reach a webhook, and treat it as deliberate public exposure.

This skill has two lanes:

- **Install lane** (new install, re-configure, enable remote access): run the onboarding checkpoint below, then follow Workflow.
- **Operate lane** (backup, restore, upgrade, prune, health check, migration, move to another machine, "n8n is slow", "disk is full"): skip onboarding, read `references/maintenance.md`, and act on the running install. Still confirm before anything destructive and always back up (database + encryption key) before changing `.env` or upgrading.

Bundled files:

- `templates/docker-compose.n8n-postgres.yml` — canonical n8n + Postgres compose (restart policy, healthcheck, log rotation, execution pruning included). Use this for the recommended default stack instead of writing compose from memory.
- `templates/.env.example` — matching env template.
- `references/maintenance.md` — upgrades, backup automation, restore rehearsal, pruning, monthly checklist, migrations.
- `references/troubleshooting.md` — extended troubleshooting.
- `references/glossary.md` — plain-Korean glossary for non-developers (용어 설명).

## Intake

Always run an onboarding checkpoint before creating files, editing `.env`, cloning repositories, or running Docker/Tailscale commands. Do not silently choose a local-only setup just because it is safe. If a reasonable default exists, present it as the proposed answer and ask the user to confirm or change it.

The onboarding checkpoint is mandatory for new installs and for any request that says "set up", "install", "configure", "self-host", or similar. It may be skipped only when the user has already provided all minimum intake values in the current conversation, or when the user explicitly says to use defaults without asking questions. It is also skipped in the operate lane (maintenance on an existing install).

Run the onboarding as a Korean, one-question-at-a-time conversation. Do not show all eight questions at once. For each question:

- Ask in Korean using simple, beginner-friendly wording.
- Show exactly three numbered choices: `1`, `2`, `3`.
- Put the safest recommended default as `1` when possible.
- Ask the user to type only `1`, `2`, or `3`.
- Wait for the user's answer before asking the next question.
- After receiving `1`, `2`, or `3`, briefly confirm what that choice means in Korean, then move to the next question.
- If the user enters anything else, explain that only `1`, `2`, or `3` is needed and ask the same question again.
- If option `3` means custom/advanced/public exposure/existing data, ask the necessary follow-up question immediately after the user selects `3`, then continue.
- Assume the user is a non-developer. When a question contains a technical term (Docker, Postgres, Tailscale, Serve/Funnel, webhook, port, encryption key, SQLite, Ollama, Qdrant, etc.), add a one-line plain-Korean gloss in parentheses, and if the user asks "그게 뭐예요?" or seems confused, explain it using `references/glossary.md` before re-asking the same question. Never make the user feel they must already know the term. 핵심 원칙: 잘 모르겠으면 거의 항상 `1`번(가장 안전한 기본값)을 고르면 됩니다.

Do not ask for all answers in a combined format like `1-1, 2-2, ...`. Do not present a full checklist for the user to edit. Keep recommended intake optional unless it changes the requested install, but include Tailscale/webhook choices because access mode changes security and URL configuration. Do not guess about external exposure, destructive overwrite, or existing data.

Use this one-question style (resolve `~` to the actual home directory of the current host and show the absolute path):

```text
설치 전에 몇 가지를 하나씩 고를게요. 숫자만 입력하면 됩니다.

1/8. n8n 파일을 어디에 둘까요?
1. ~/n8n-self-hosting - 기본 위치라 가장 무난합니다
2. ~/docker/n8n - Docker 관련 폴더를 따로 모으고 싶을 때 좋습니다
3. 직접 입력 - 원하는 경로가 있으면 다음에 물어볼게요

번호만 입력해 주세요: 1, 2, 3
```

Onboarding question sequence and Korean prompts:

1. Install path
   - Prompt: "1/8. n8n 파일을 어디에 둘까요?"
   - `1`: `~/n8n-self-hosting`, shown as the resolved absolute path on this host.
   - `2`: `~/docker/n8n`.
   - `3`: custom path; follow up in Korean: "사용할 전체 경로를 입력해 주세요."

2. Access mode
   - Prompt: "2/8. n8n에 어떻게 접속할까요?"
   - `1`: local only, `http://localhost:5678`; recommended for private local use.
   - `2`: Tailscale Serve, private HTTPS inside the user's tailnet.
   - `3`: LAN, public webhook, or custom exposure; follow up in Korean to choose LAN vs Tailscale Funnel/public webhook vs other. For LAN, warn that login over plain HTTP needs `N8N_SECURE_COOKIE=false` and is only acceptable on a trusted network. For public exposure, warn that it needs extra security.

3. Host environment
   - Prompt: "3/8. 어디에 설치하나요?"
   - `1`: detected local host, for example macOS arm64 / Apple Silicon.
   - `2`: remote Linux server or VPS.
   - `3`: NAS, Windows WSL, or custom; follow up in Korean for the exact host type.

4. Stack shape
   - Prompt: "4/8. 어떤 구성으로 설치할까요?"
   - `1`: n8n + Postgres; recommended durable default. Use the bundled `templates/docker-compose.n8n-postgres.yml`.
   - `2`: n8n only; simpler but less durable if SQLite is used.
   - `3`: AI starter kit with Postgres + Ollama + Qdrant.

5. AI runtime
   - Prompt: "5/8. AI 기능도 같이 준비할까요?"
   - `1`: no AI runtime; recommended unless the user needs local AI workflows.
   - `2`: host Ollama or external LLM API; follow up in Korean to choose host Ollama vs external LLM API.
   - `3`: Docker Ollama.

6. Ports
   - Prompt: "6/8. 포트 번호는 어떻게 할까요?"
   - `1`: defaults: n8n `5678`, Ollama `11434`, Qdrant `6333`.
   - `2`: change only the n8n port; follow up in Korean for the n8n port.
   - `3`: custom all relevant ports; follow up in Korean for all ports used by the selected stack.

7. Secrets
   - Prompt: "7/8. 비밀번호와 암호화 키는 어떻게 만들까요?"
   - `1`: auto-generate strong secrets; recommended for new installs.
   - `2`: user provides secrets; follow up in Korean for how they want to provide them.
   - `3`: preserve existing `.env` secrets; only valid for existing installs, and must read existing `.env` before editing.

8. Backup expectation
   - Prompt: "8/8. 백업은 어느 정도로 준비할까요?"
   - `1`: manual backup instructions after install.
   - `2`: scheduled local backup; follow up in Korean for schedule/location, then set it up after install using `references/maintenance.md`.
   - `3`: off-host backup plan; follow up in Korean for destination or preferred service.

After all answers are collected, summarize the selected plan in Korean and ask for one final confirmation before creating files, editing `.env`, cloning repositories, or running Docker/Tailscale commands. If the user confirms, proceed without asking the same onboarding questions again.

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
- Extra access controls: rely on n8n user management (enable MFA for the owner account) plus tailnet ACLs. n8n v1 removed the `N8N_BASIC_AUTH_*` variables, so do not promise basic auth; for Funnel, n8n's own login is the only application-level gate.

## Defaults

For the recommended n8n + Postgres stack:

- Start from the bundled `templates/docker-compose.n8n-postgres.yml` and `templates/.env.example`; do not write compose files from memory.
- The template already includes: `restart: unless-stopped`, a Postgres healthcheck with `depends_on: condition: service_healthy`, Docker log rotation, execution-data pruning (`EXECUTIONS_DATA_PRUNE=true`, 7-day max age), `N8N_DEFAULT_BINARY_DATA_MODE=filesystem`, port binding to `127.0.0.1` by default, and Postgres kept internal to the compose network.
- Generate strong random secrets with `openssl rand -hex 32` (keys) and `openssl rand -hex 16` (passwords).
- Leave first owner account creation to the n8n web UI; recommend enabling MFA for the owner.
- Verify n8n at `http://localhost:5678/setup` and later via `http://localhost:5678/healthz`.
- Record where the encryption key is backed up; treat it as the single most important value to retain.

For the local single-user AI starter kit:

- Use the official repository `https://github.com/n8n-io/self-hosted-ai-starter-kit.git`.
- Use `docker compose --profile cpu up -d`.
- Keep Postgres internal to the compose network; replace all demo secrets.
- Verify Ollama at `http://localhost:11434/api/tags`.
- Treat Docker Ollama on Mac as CPU-oriented. For better Apple Silicon performance, prefer host Ollama and set `OLLAMA_HOST=host.docker.internal:11434`.

For LAN access (option `3` in access mode):

- Bind the n8n port to `0.0.0.0` (template: set `N8N_BIND=0.0.0.0` in `.env`) so other devices on the same network can reach `http://<host-LAN-IP>:5678`.
- Plain-HTTP login requires `N8N_SECURE_COOKIE=false`. Set it only for LAN-over-HTTP, tell the user this weakens cookie security, and never combine it with public exposure. If the user wants HTTPS instead, steer them to Tailscale Serve.
- Set `WEBHOOK_URL=http://<host-LAN-IP>:5678/` so generated webhook URLs work from other LAN devices.

For Tailscale remote access:

- Default to Tailscale Serve (private tailnet HTTPS). Do not use Funnel unless an external webhook requires it.
- Keep n8n bound to localhost or the Docker network; let Tailscale terminate TLS and proxy to `5678`. Do not publish `5678` to `0.0.0.0`.
- Use the MagicDNS hostname `https://<machine>.<tailnet>.ts.net` as the canonical URL; let Tailscale provision the certificate.
- Set `N8N_HOST`, `N8N_PROTOCOL=https`, `WEBHOOK_URL=https://<machine>.<tailnet>.ts.net/`, and `N8N_PROXY_HOPS=1` so n8n trusts the forwarded TLS headers.
- Prefer host-installed Tailscale; use a Docker sidecar only when the host cannot run Tailscale.

## Workflow

0. Route the request.
   - New install / re-configure / enable remote access → continue with step 1.
   - Maintenance on a running install (backup, restore, upgrade, prune, health check, migrate, move, slow/full-disk complaints) → read `references/maintenance.md` and follow the matching procedure; skip onboarding but keep all Safety Rules.

1. Run the mandatory onboarding checkpoint.
   - Ask the minimum intake in Korean, one question at a time, with `1/2/3` choices.
   - Wait for the user's numeric answer before asking the next onboarding question.
   - After all answers are collected, summarize the plan in Korean and ask for final confirmation before creating files, cloning repositories, writing `.env`, or running Docker/Tailscale commands.
   - If the user already supplied all minimum intake values or explicitly asked to use defaults without questions, state that the checkpoint is satisfied and proceed.

2. Inspect existing state.
   - Check whether the install path exists and whether it is empty, a git repo, or contains compose volumes/config.
   - Run `docker --version` and `docker compose version`.
   - Check port availability when using defaults.

3. Decide stack.
   - n8n + Postgres: copy the bundled templates into the install path.
   - Local AI starter kit: clone or update `n8n-io/self-hosted-ai-starter-kit`.
   - Existing project: read current compose/env before editing.
   - Remote access: plan Tailscale Serve (or Funnel if external webhooks are needed), the `ts.net` URL, webhook URL, backup, and version pinning before running containers.

4. Create configuration.
   - Create `.env` from the bundled `templates/.env.example` (or the project's `.env.example` when one exists).
   - Replace demo secrets; never keep `password`, `super-secret-key`, or `even-more-secret`.
   - Preserve existing `N8N_ENCRYPTION_KEY` when data already exists. Changing it can make credentials unreadable.
   - Set `GENERIC_TIMEZONE` or `TZ` when the compose file supports it; otherwise note the gap.
   - For production use, pin `N8N_VERSION` to the current stable release (check https://github.com/n8n-io/n8n/releases) instead of `latest`.
   - For Tailscale remote access, set `N8N_HOST=<machine>.<tailnet>.ts.net`, `N8N_PROTOCOL=https`, `WEBHOOK_URL=https://<machine>.<tailnet>.ts.net/`, and `N8N_PROXY_HOPS=1`; keep n8n on `5678` unpublished to the public interface and let Tailscale proxy to it.

5. Start services.
   - Use the smallest applicable command, usually `docker compose up -d`.
   - For the AI starter kit, include the selected profile: `cpu`, `gpu-nvidia`, or `gpu-amd`.
   - Do not run destructive cleanup commands unless the user explicitly asks to reset data.

6. Verify.
   - `docker compose ps` shows n8n and dependencies running, with Postgres healthy where applicable.
   - `curl -I` or `curl -L` reaches n8n setup or signin; `curl http://localhost:5678/healthz` returns ok once up.
   - For Ollama, `/api/tags` lists the expected model or model pull is still in progress.
   - Check recent logs for n8n startup, database connection, and obvious errors.
   - For Tailscale, confirm `tailscale serve status` maps the `ts.net` host to `5678`, then reach `https://<machine>.<tailnet>.ts.net/setup` from another tailnet device. For Funnel, confirm `tailscale funnel status` and that the webhook URL responds from outside the tailnet.

7. Set up the backup the user chose in question 8.
   - Manual: show the backup commands below and confirm the user stored the encryption key off-host.
   - Scheduled/off-host: follow "Backup automation" in `references/maintenance.md` before handing off.

8. Report concise handoff.
   - Access URL: `http://localhost:5678` for local, `http://<LAN-IP>:5678` for LAN, or `https://<machine>.<tailnet>.ts.net` for Tailscale.
   - Install path.
   - Start, stop, status, and logs commands.
   - Secrets/persistence location without printing secret values, and confirmation that the encryption key is backed up off-host.
   - Backup status: how (or when, if scheduled) the database is dumped and where the encryption key is stored; flag if no backup exists yet.
   - Maintenance pointer: mention that upgrades, backup automation, and the monthly health check live in this skill (`references/maintenance.md`) and can be requested later in plain language ("n8n 업그레이드 해줘", "백업 확인해줘").
   - Remaining production gaps, especially backups, upgrades, and whether Funnel (public exposure) is active.

## Commands

Typical n8n + Postgres install (bundled template):

```bash
mkdir -p <install-path> && cd <install-path>
# copy templates/docker-compose.n8n-postgres.yml -> docker-compose.yml
# copy templates/.env.example -> .env, then fill in generated secrets
docker compose up -d
```

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
curl --max-time 10 http://localhost:5678/healthz
curl --max-time 10 http://localhost:11434/api/tags   # AI starter kit only
docker compose logs --tail=80 n8n
```

Operations:

```bash
docker compose up -d
docker compose down          # never use 'down -v' in normal operation
docker compose logs -f n8n

# Upgrade: follow the full procedure in references/maintenance.md
# (back up first, pin the new tag, never auto-update)
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

Backup and restore (one-off; automation lives in `references/maintenance.md`):

```bash
# Postgres logical dump. Variables expand INSIDE the container, where .env values exist.
docker compose exec -T postgres sh -c 'pg_dump -U "$POSTGRES_USER" "$POSTGRES_DB"' \
  > n8n-db-$(date +%F).sql

# Restore a dump into a running, empty Postgres
docker compose exec -T postgres sh -c 'psql -U "$POSTGRES_USER" "$POSTGRES_DB"' \
  < n8n-db-<date>.sql

# Back up the encryption key OFF-host (password manager / secret store).
# Copy to clipboard without printing it to the terminal:
grep '^N8N_ENCRYPTION_KEY=' .env | cut -d= -f2- | pbcopy        # macOS
grep '^N8N_ENCRYPTION_KEY=' .env | cut -d= -f2- | xclip -sel c  # Linux

# SQLite-only installs: STOP n8n first — tarring a live SQLite file can produce a corrupt backup
docker compose stop n8n
docker run --rm -v <n8n_data_volume>:/data -v "$PWD":/backup alpine \
  tar czf /backup/n8n-data-$(date +%F).tgz -C /data .
docker compose start n8n
```

A complete backup is the database dump (or data-volume snapshot) **plus** the encryption key stored separately. Restoring the database without the original key leaves saved credentials undecryptable.

## Safety Rules

- Never overwrite an existing `.env` without reading it first.
- Never regenerate `N8N_ENCRYPTION_KEY` for an existing install unless the user explicitly accepts credential loss.
- Back up `N8N_ENCRYPTION_KEY` to a location separate from the install directory (password manager or secret store). Losing it makes every stored credential unreadable even with a full database backup, so a database dump alone is not a sufficient backup. Do not print the key value into the conversation or terminal output; use the clipboard commands above.
- Take a backup (database dump plus encryption key) before any upgrade or any change that touches `.env` secrets.
- Never run `docker compose down -v`, remove volumes, or delete install directories as part of ordinary install/upgrade.
- Never set up unattended auto-updates (Watchtower or similar) for n8n. Upgrades run irreversible database migrations; they must be deliberate, with a fresh backup. Rollback after a migration means restoring the backup, not lowering the image tag.
- Do not expose Postgres, Qdrant, or Ollama publicly unless the user explicitly asks and security controls are planned.
- `N8N_SECURE_COOKIE=false` is acceptable only for plain-HTTP access on a trusted LAN. Never combine it with public exposure, and remove it when switching to HTTPS.
- For remote n8n, require HTTPS and a correct `WEBHOOK_URL` before claiming webhook readiness; with Tailscale Serve the `ts.net` HTTPS endpoint satisfies this inside the tailnet.
- Default remote access to Tailscale Serve (private tailnet). Never enable Tailscale Funnel without explicit user confirmation — Funnel publishes n8n to the public internet, so treat it like opening a public port.
- Do not publish n8n's `5678` to `0.0.0.0` when fronting it with Tailscale; let Tailscale proxy to localhost or the Docker network.
- For production, prefer pinned image tags over `latest` and call out backup gaps.

## Troubleshooting

Most common install-time issues (full list in `references/troubleshooting.md`):

- `curl /` returns 404 but `/setup` works: n8n is up; use `/setup` or the editor URL.
- Port conflict: adjust host-side port mapping, then update the handoff URL.
- Login fails with a "secure cookie" error over plain HTTP (LAN IP): set `N8N_SECURE_COOKIE=false` (trusted LAN only) or switch to Tailscale Serve for HTTPS.
- n8n login loop or "secure cookie" errors behind Tailscale Serve: ensure `N8N_PROTOCOL=https` and `N8N_PROXY_HOPS=1` so n8n trusts the forwarded TLS.
- Ollama model missing: inspect `ollama-pull-llama` logs or run `docker compose exec ollama ollama pull <model>`.
- Mac performance is poor with Docker Ollama: install Ollama on the host and set `OLLAMA_HOST=host.docker.internal:11434`.
- Webhooks show `localhost` or a wrong host: `WEBHOOK_URL`/`N8N_HOST` were not updated; fix the env and restart n8n.

For operations issues (disk full, slow UI, failed upgrade, restore problems, containers missing after reboot), read `references/troubleshooting.md` and `references/maintenance.md`.
