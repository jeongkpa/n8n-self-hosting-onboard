# Maintenance and Operations (Day 2+)

Procedures for keeping a self-hosted n8n alive: upgrades, backup automation, restore
rehearsal, execution-data pruning, the monthly health check, and migrations. All Safety
Rules from SKILL.md apply here. Talk to the user in plain Korean; commands stay as-is.

## Upgrade procedure

n8n releases often, and every upgrade may run **irreversible database migrations**.
Lowering the image tag after a migration does NOT roll back — the only rollback is
restoring the pre-upgrade backup. Therefore:

1. Check the release notes for breaking changes: https://github.com/n8n-io/n8n/releases
   (also https://docs.n8n.io/release-notes/). For large jumps (many minor versions),
   upgrade in steps rather than one leap.
2. Back up first, always: database dump + confirm the encryption key is stored off-host
   (commands in SKILL.md "Backup and restore").
3. Update the pinned tag: edit `N8N_VERSION` in `.env` (or the image tag in compose).
   If the install uses `latest`, this is the moment to recommend pinning.
4. Apply: `docker compose pull && docker compose up -d`.
5. Verify: `curl http://localhost:5678/healthz` returns ok, the user can log in, and one
   known workflow runs. Check `docker compose logs --tail=100 n8n` for migration errors.
6. If the upgrade fails: do NOT just lower the tag. Restore the pre-upgrade dump into a
   clean database, set the old `N8N_VERSION` back, and start again.

Never configure Watchtower or any unattended auto-update for n8n. Migrations make
unattended upgrades a data-loss mechanism, not a convenience.

## Backup automation

A complete backup = database dump **plus** the encryption key stored separately
(password manager / secret store). Automate the dump; the key only needs to be saved
once (and again if it ever changes).

Daily dump with 7-copy retention (`<install-path>/n8n-backup.sh`):

```bash
#!/bin/sh
# Daily n8n Postgres dump, keep the 7 most recent.
cd "$(dirname "$0")" || exit 1
mkdir -p backups
docker compose exec -T postgres sh -c 'pg_dump -U "$POSTGRES_USER" "$POSTGRES_DB"' \
  | gzip > "backups/n8n-db-$(date +%F).sql.gz" || exit 1
ls -1t backups/n8n-db-*.sql.gz | tail -n +8 | xargs rm -f
```

Make it executable and schedule it:

```bash
chmod +x n8n-backup.sh
# cron (Linux, and works on macOS): daily at 03:00
crontab -e   # add:
# 0 3 * * * /full/install/path/n8n-backup.sh >> /full/install/path/backups/backup.log 2>&1
```

On macOS, note that the machine must be awake at the scheduled time; suggest a time the
Mac is usually on, or `launchd` with `StartCalendarInterval` for a more robust setup.

Off-host backup (intake option 3): after the local dump, sync `backups/` to another
location the user already has — iCloud/Dropbox folder, `rclone` to object storage, or
`scp` to another machine. Never sync the `.env` file itself to shared storage; the
encryption key belongs in a password manager, not in a synced folder.

SQLite-only installs: replace the dump line with the stop → tar → start snapshot from
SKILL.md. Stopping n8n briefly is required for a consistent SQLite copy.

## Restore rehearsal — a backup is only real once it has been restored

After setting up backups (and roughly twice a year), verify one:

1. Spin up a throwaway Postgres: `docker run -d --name n8n-restore-test -e POSTGRES_PASSWORD=test -e POSTGRES_USER=n8n -e POSTGRES_DB=n8n postgres:16-alpine`
2. Load the latest dump: `gunzip -c backups/n8n-db-<date>.sql.gz | docker exec -i n8n-restore-test psql -U n8n n8n`
3. Sanity-check: `docker exec n8n-restore-test psql -U n8n n8n -c 'SELECT count(*) FROM workflow_entity;'` — expect the user's workflow count.
4. Clean up: `docker rm -f n8n-restore-test`

For a full rehearsal (or before relying on a backup for a real migration), restore into
a complete n8n+Postgres stack in a temp directory with the SAME `N8N_ENCRYPTION_KEY`,
log in, and open a credential. If credentials fail to decrypt, the stored key does not
match the dump — fix that today, not on disaster day.

## Execution data pruning and database size

The bundled template enables pruning by default (`EXECUTIONS_DATA_PRUNE=true`,
`EXECUTIONS_DATA_MAX_AGE=168` hours, `EXECUTIONS_DATA_PRUNE_MAX_COUNT=10000`). For
installs created without it (including the AI starter kit), add those env vars to the
n8n service and restart — runaway execution history is the most common reason a
self-hosted n8n gets slow or fills the disk.

Tuning: heavy workflows with large payloads may want `EXECUTIONS_DATA_MAX_AGE=72`;
users who debug rarely can also set `EXECUTIONS_DATA_SAVE_ON_SUCCESS=none` to keep only
failed executions.

SQLite note: pruning deletes rows but the `database.sqlite` file does not shrink by
itself. To reclaim disk: stop n8n, then
`sqlite3 database.sqlite 'VACUUM;'` (or run it via a temporary alpine container against
the data volume), then start n8n. Postgres reclaims space automatically via autovacuum.

## Monthly health check (월간 점검)

Run these five checks and report results in plain Korean; fix what fails:

1. **Alive**: `docker compose ps` (everything Up, postgres healthy) and
   `curl -s http://localhost:5678/healthz` returns `{"status":"ok"}`.
2. **Disk**: `df -h` for the host, `docker system df` for Docker. If the n8n volume or
   Postgres keeps growing, check pruning settings above.
3. **Backups**: newest file in `backups/` is recent (`ls -lt backups/ | head`); confirm
   the encryption key is still in the password manager.
4. **Version**: compare the running version (`docker compose exec n8n n8n --version`)
   with the latest release; offer the upgrade procedure if behind. Being a few releases
   behind is fine; months behind makes the eventual jump riskier.
5. **Security audit**: `docker compose exec n8n n8n audit` reports risky credentials and
   workflow settings; summarize findings for the user.

Also worth checking after host reboots: `docker compose ps` — with the bundled template
(`restart: unless-stopped`) everything should come back on its own; if not, the compose
file is missing the restart policy.

## Migration: SQLite → Postgres

The natural growth path for "n8n only" installs. n8n has no in-place converter; the
reliable route is export/import with the same encryption key:

1. Back up the existing install (volume snapshot + key).
2. Export from the old instance:
   `docker compose exec n8n n8n export:workflow --all --output=/home/node/.n8n/export/workflows.json`
   `docker compose exec n8n n8n export:credentials --all --output=/home/node/.n8n/export/credentials.json`
   (credentials export stays encrypted — that is fine, the key travels in `.env`).
3. Stand up the new n8n + Postgres stack from the bundled template in a new directory,
   reusing the ORIGINAL `N8N_ENCRYPTION_KEY` in the new `.env`. This is mandatory;
   credentials are unreadable under a new key.
4. Copy the export files into the new n8n volume, then import:
   `docker compose exec n8n n8n import:workflow --input=... ` and
   `docker compose exec n8n n8n import:credentials --input=...`
5. Recreate the owner account in the web UI (users/settings do not migrate), re-activate
   workflows, run one end-to-end test, then retire the old stack only after the user
   confirms.

## Moving to another machine

Same recipe as restore, applied across hosts:

1. On the old host: fresh database dump + copy of `docker-compose.yml` and `.env`
   (the `.env` carries the encryption key — transfer it securely, not via public
   channels).
2. On the new host: install Docker, recreate the install directory with the same
   compose + `.env`, start only Postgres, restore the dump, then start n8n.
3. Re-point access: Tailscale machine name changes → update `N8N_HOST`/`WEBHOOK_URL`
   and re-run `tailscale serve --bg 5678`; LAN IP changes → update `WEBHOOK_URL`.
4. Verify login, credentials decrypt, and one workflow run before deleting anything on
   the old host.

## When this skill is not enough (growth pointers)

Mention these exist; do not implement them here:

- **Queue mode**: many concurrent or long-running executions → n8n queue mode with
  Redis and worker containers (https://docs.n8n.io/hosting/scaling/queue-mode/).
- **Multi-user/team production**: consider source control features, SSO, and a real
  monitoring stack — beyond this skill's single-host scope.
