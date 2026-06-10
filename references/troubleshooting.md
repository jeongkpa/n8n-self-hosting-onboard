# Troubleshooting (extended)

Install-time quick hits live in SKILL.md; this file covers everything else. Explain the
fix to the user in plain Korean before running commands.

## Access and login

- `curl /` returns 404 but `/setup` works: n8n is up; use `/setup` or the editor URL.
- Login fails with a "secure cookie" error over plain HTTP (`http://<LAN-IP>:5678`):
  set `N8N_SECURE_COOKIE=false` (trusted LAN only — see Safety Rules) or move the user
  to Tailscale Serve for real HTTPS. Remove the flag if HTTPS is adopted later.
- Login loop or "secure cookie" errors behind Tailscale Serve: ensure
  `N8N_PROTOCOL=https` and `N8N_PROXY_HOPS=1` so n8n trusts the forwarded TLS headers.
- Port conflict on 5678: change `N8N_PORT` (host side) in `.env`, recreate, update the
  handoff URL and `WEBHOOK_URL`.

## Webhooks and remote access

- Webhooks show `localhost` or a wrong host: `WEBHOOK_URL`/`N8N_HOST` were not updated
  to the current access URL; fix `.env` and restart n8n.
- External SaaS webhook never fires while only Serve is enabled: Serve is
  tailnet-private; the callback needs Funnel (public) — confirm with the user before
  enabling, then update `WEBHOOK_URL`.
- Tailscale `ts.net` URL not resolving: MagicDNS must be enabled in the tailnet admin
  console; without it, fall back to the tailnet IP (`100.x.y.z:5678`).
- `tailscale serve` fails with an HTTPS/cert error: enable HTTPS certificates for the
  tailnet in the admin console and confirm the node is logged in (`tailscale status`).

## Performance and disk

- UI slow, executions list crawling, or disk filling up: almost always unpruned
  execution history. Apply the pruning env vars and, for SQLite, VACUUM — see
  "Execution data pruning" in references/maintenance.md.
- Disk full from Docker logs: the bundled template rotates logs; older installs without
  a `logging:` section accumulate unbounded json logs. Add log rotation and recreate.
- Mac performance is poor with Docker Ollama: install Ollama on the host and set
  `OLLAMA_HOST=host.docker.internal:11434`.

## Containers and startup

- Containers not running after a host reboot: compose file lacks
  `restart: unless-stopped`; add it and `docker compose up -d`.
- n8n starts before the database/import is ready: with the bundled template,
  `depends_on: condition: service_healthy` prevents this; in the AI starter kit, wait
  for the import container to complete, then re-check n8n.
- Ollama model missing: inspect `ollama-pull-llama` logs or run
  `docker compose exec ollama ollama pull <model>`.
- Python task runner warning in the n8n image: usually not blocking for standard JS
  workflows; note it and move on.

## Upgrades, restore, and credentials

- Upgrade failed / n8n won't start after upgrade: check
  `docker compose logs --tail=200 n8n` for migration errors. Do NOT just lower the image
  tag — migrations are one-way. Restore the pre-upgrade dump, pin the old version, then
  retry the upgrade later (see references/maintenance.md "Upgrade procedure").
- After a restore, credentials fail to decrypt or workflows error on saved auth: the
  restored database was paired with a different `N8N_ENCRYPTION_KEY`. Set `.env` back to
  the original key and restart; the key must match the dump.
- Imported credentials unreadable after SQLite→Postgres migration: same cause — the new
  `.env` must reuse the original encryption key.
