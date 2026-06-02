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

Run the onboarding as a Korean, one-question-at-a-time conversation. Do not show all eight questions at once. For each question:

- Ask in Korean using simple, beginner-friendly wording.
- Show exactly three numbered choices: `1`, `2`, `3`.
- Put the safest recommended default as `1` when possible.
- Ask the user to type only `1`, `2`, or `3`.
- Wait for the user's answer before asking the next question.
- After receiving `1`, `2`, or `3`, briefly confirm what that choice means in Korean, then move to the next question.
- If the user enters anything else, explain that only `1`, `2`, or `3` is needed and ask the same question again.
- If option `3` means custom/advanced/public exposure/existing data, ask the necessary follow-up question immediately after the user selects `3`, then continue.
- Assume the user is a non-developer. When a question contains a technical term (Docker, Postgres, Tailscale, Serve/Funnel, webhook, port, encryption key, SQLite, Ollama, Qdrant, etc.), add a one-line plain-Korean gloss in parentheses, and if the user asks "그게 뭐예요?" or seems confused, explain it using the "용어 설명 (비개발자용)" section below before re-asking the same question. Never make the user feel they must already know the term.

Do not ask for all answers in a combined format like `1-1, 2-2, ...`. Do not present a full checklist for the user to edit. Keep recommended intake optional unless it changes the requested install, but include Tailscale/webhook choices because access mode changes security and URL configuration. Do not guess about external exposure, destructive overwrite, or existing data.

Use this one-question style:

```text
설치 전에 몇 가지를 하나씩 고를게요. 숫자만 입력하면 됩니다.

1/8. n8n 파일을 어디에 둘까요?
1. /Users/fran/n8n-self-hosting - 기본 위치라 가장 무난합니다
2. ~/docker/n8n - Docker 관련 폴더를 따로 모으고 싶을 때 좋습니다
3. 직접 입력 - 원하는 경로가 있으면 다음에 물어볼게요

번호만 입력해 주세요: 1, 2, 3
```

Onboarding question sequence and Korean prompts:

1. Install path
   - Prompt: "1/8. n8n 파일을 어디에 둘까요?"
   - `1`: default install path, usually `/Users/fran/n8n-self-hosting` on this host.
   - `2`: `~/docker/n8n`.
   - `3`: custom path; follow up in Korean: "사용할 전체 경로를 입력해 주세요."

2. Access mode
   - Prompt: "2/8. n8n에 어떻게 접속할까요?"
   - `1`: local only, `http://localhost:5678`; recommended for private local use.
   - `2`: Tailscale Serve, private HTTPS inside the user's tailnet.
   - `3`: public webhook, LAN, or custom exposure; follow up in Korean to choose LAN vs Tailscale Funnel/public webhook vs other, and warn that public exposure needs extra security.

3. Host environment
   - Prompt: "3/8. 어디에 설치하나요?"
   - `1`: detected local host, for example macOS arm64 / Apple Silicon.
   - `2`: remote Linux server or VPS.
   - `3`: NAS, Windows WSL, or custom; follow up in Korean for the exact host type.

4. Stack shape
   - Prompt: "4/8. 어떤 구성으로 설치할까요?"
   - `1`: n8n + Postgres; recommended durable default.
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
   - `2`: scheduled local backup; follow up in Korean for schedule/location.
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
- Extra access controls: rely on n8n user management plus tailnet ACLs; for Funnel, confirm whether basic auth or IP-independent controls are also wanted.

## 용어 설명 (비개발자용)

This glossary exists so a first-time, non-developer user is never blocked by jargon. Use it to answer "그게 뭐예요?" during onboarding. Keep explanations to one or two plain-Korean sentences, lead with an everyday analogy, then say what to pick if unsure. Do not lecture; answer, then return to the question.

핵심 원칙: 잘 모르겠으면 거의 항상 **`1`번(가장 안전한 기본값)** 을 고르면 됩니다.

- **셀프호스팅(self-hosting)**: 남의 서비스(클라우드)를 빌리지 않고, 내 컴퓨터/서버에 프로그램을 직접 띄워서 쓰는 것. 데이터가 내 손 안에 있는 대신 관리도 내가 합니다.
- **n8n**: 여러 앱·서비스를 "이거 오면 저거 해줘" 식으로 자동으로 연결해 주는 자동화 도구. 블록을 선으로 잇듯 워크플로우를 만듭니다.
- **Docker / 컨테이너**: 프로그램과 그 실행에 필요한 모든 것을 한 상자에 담아 "어디서나 똑같이 켜지게" 만든 기술. 내 컴퓨터를 더럽히지 않고 깔끔하게 설치·삭제할 수 있습니다.
- **Docker Compose**: 여러 컨테이너(예: n8n + 데이터베이스)를 설정 파일 하나로 한 번에 켜고 끄는 도구.
- **이미지(image) / 풀(pull)**: 이미지는 프로그램이 담긴 "설치 원본". 풀은 그 원본을 인터넷에서 내려받는 것. 처음 설치 때 시간이 걸리는 이유입니다.
- **포트(port)**: 컴퓨터의 "몇 번 출입문"인지 가리키는 번호. n8n은 보통 `5678`번 문으로 접속합니다. 다른 프로그램이 같은 번호를 쓰면 "충돌"이 납니다.
- **localhost / `127.0.0.1`**: "바로 이 컴퓨터"를 뜻하는 주소. `http://localhost:5678`은 내 컴퓨터에서만 열리는 주소입니다.
- **LAN(랜)**: 같은 공유기/사무실 망에 연결된 기기들. LAN 접속을 켜면 옆자리 컴퓨터·폰에서도 접속할 수 있습니다(인터넷 전체 공개는 아님).
- **데이터베이스 / Postgres / SQLite**: 워크플로우·설정·실행 기록을 저장하는 "창고". **Postgres**는 튼튼한 별도 창고(권장), **SQLite**는 파일 하나로 쓰는 가벼운 창고(간단하지만 덜 견고).
- **시크릿(secret) / 비밀번호 / 자동 생성**: 외부에 노출되면 안 되는 열쇠값들. "자동 생성"은 추측 불가능한 무작위 열쇠를 컴퓨터가 만들어 주는 것이라 직접 정하는 것보다 안전합니다.
- **암호화 키(`N8N_ENCRYPTION_KEY`)**: n8n이 저장한 비밀번호·API 키를 잠그고 푸는 "마스터 열쇠". **이걸 잃으면 백업이 있어도 저장된 자격증명을 다시 열 수 없습니다.** 그래서 설치 폴더 밖(비밀번호 관리자 등)에 따로 보관해야 합니다.
- **백업(backup)**: 사고에 대비해 데이터 사본을 따로 떠 두는 것. 완전한 백업 = 데이터베이스 사본 **+** 위의 암호화 키 (둘 다 있어야 복구됩니다).
- **AI 런타임 / LLM**: n8n 안에서 AI(챗봇 같은 언어 모델)를 쓰게 해 주는 부분. 필요 없으면 "AI 없음"을 골라도 됩니다.
- **Ollama**: 내 컴퓨터에서 AI 모델(예: llama3.2)을 직접 돌리는 프로그램. 인터넷 API 없이 로컬에서 AI를 씁니다. **Docker Ollama**는 컨테이너 안에서 돌리는 방식(맥에서는 CPU 위주라 다소 느림), **호스트 Ollama**는 맥에 직접 설치해 더 빠른 방식.
- **외부 LLM API**: OpenAI·Claude 등 클라우드 AI를 키(API Key)로 불러 쓰는 것. 내 컴퓨터 성능 부담이 없습니다.
- **Qdrant / 벡터 DB**: AI가 문서를 "의미로 검색"하게 해 주는 특수 창고. RAG(내 문서 기반 답변) 같은 AI 워크플로우에 쓰입니다.
- **Tailscale**: 내 기기들끼리만 연결되는 "사설 안전망(VPN)". 복잡한 설정 없이 내 노트북·서버를 한 울타리로 묶어 줍니다.
- **tailnet**: 내 Tailscale 기기들이 모인 그 울타리(사설 네트워크) 전체.
- **MagicDNS / `ts.net` 주소**: Tailscale이 기기마다 자동으로 붙여 주는 고정 주소(예: `내기기.tailxxxx.ts.net`). 외우기 쉬운 이름으로 접속하게 해 줍니다.
- **HTTPS / 인증서**: 주소창에 자물쇠가 뜨는 암호화된 안전한 연결. Tailscale이 인증서를 자동으로 만들어 줍니다.
- **Tailscale Serve(서브)**: 내 tailnet **안에서만** HTTPS로 n8n에 접속하게 해 주는 기능. 인터넷 전체에는 공개되지 않아 안전합니다.
- **Tailscale Funnel(퍼널)**: n8n을 **인터넷 전체에 공개**하는 기능. 외부 서비스가 내 n8n으로 신호(웹훅)를 보내야 할 때만 씁니다. 공개 포트를 여는 것과 같으니 신중히, 반드시 로그인 보안을 켜고 사용하세요.
- **웹훅(webhook)**: "어떤 일이 생기면 이 주소로 알려 줘"라고 다른 서비스에 등록해 두는 콜백 주소. 외부 서비스가 내 n8n을 호출하려면 그 주소가 외부에서 닿아야 합니다(그래서 Funnel이 필요할 수 있음).
- **`WEBHOOK_URL` / `N8N_HOST` / `N8N_PROXY_HOPS`**: n8n에게 "네 진짜 바깥 주소는 이거야"라고 알려 주는 설정값들. 원격 접속을 켜면 생성되는 웹훅 주소가 올바르게 나오도록 맞춰 줍니다.
- **리버스 프록시(reverse proxy)**: 바깥 요청을 먼저 받아서 안쪽 프로그램으로 넘겨 주는 "안내 데스크". 이 스킬은 보통 더 간단한 Tailscale을 먼저 권합니다.
- **타임존(timezone)**: 스케줄·기록 시각의 기준 시간대. 한국이면 `Asia/Seoul`.

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
   - Ask the minimum intake in Korean, one question at a time, with `1/2/3` choices.
   - Wait for the user's numeric answer before asking the next onboarding question.
   - After all answers are collected, summarize the plan in Korean and ask for final confirmation before creating files, cloning repositories, writing `.env`, or running Docker/Tailscale commands.
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
