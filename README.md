# n8n-self-hosting-onboard

비개발자도 따라올 수 있는 n8n 셀프호스팅 설치·운영 스킬 (Claude Code / Codex 호환).

An agent skill that installs **and operates** a self-hosted n8n with Docker Compose —
guided by a one-question-at-a-time Korean onboarding flow designed for non-developers.

## What it covers

- **Install (Day 0)**: n8n + Postgres (bundled compose template), n8n only, or the
  official AI starter kit (Postgres + Ollama + Qdrant). Local, LAN, or Tailscale
  access (Serve for private HTTPS, Funnel only for external webhooks).
- **Operate (Day 2+)**: safe upgrades (backup first, rollback = restore), backup
  automation with retention, restore rehearsal, execution-data pruning, a monthly
  health check, SQLite→Postgres migration, and moving n8n to another machine.
- **Safety**: never lose `N8N_ENCRYPTION_KEY`, never `down -v`, never auto-update,
  never expose without confirmation.

## Layout

```
SKILL.md                                # entry point: intake, workflow, safety rules
references/glossary.md                  # plain-Korean glossary for non-developers
references/maintenance.md               # day-2 operations procedures
references/troubleshooting.md           # extended troubleshooting
templates/docker-compose.n8n-postgres.yml
templates/.env.example
agents/openai.yaml
```

## Install the skill

```bash
# Claude Code
git clone https://github.com/jeongkpa/n8n-self-hosting-onboard ~/.claude/skills/n8n-self-hosting-onboard

# Codex
git clone https://github.com/jeongkpa/n8n-self-hosting-onboard ~/.codex/skills/n8n-self-hosting-onboard
```

Then ask your agent things like "n8n 설치해줘", "n8n 업그레이드 해줘", or "n8n 백업 확인해줘".
