# 03 — TOOLS & REPOS

Alle Tools, die in den letzten Projekten auf CKs Maschine + Cowork etabliert sind. Install-Befehle, Workflow, was rein- und was rausgehört.

---

## §1 Ralph (`snarktank/ralph`)

**Was er macht:** Autonomer Loop, der AI-Coding-Tools (Claude Code oder Amp) wiederholt aufruft, bis alle PRD-Stories `passes: true` sind. Jede Iteration = fresh Context. Memory zwischen Iterations: Git-History, `progress.txt`, `prd.json`.

**Voraussetzungen:**
- Eines von: Amp CLI oder Claude Code (`npm install -g @anthropic-ai/claude-code`)
- `jq` installiert (Windows: `choco install jq` oder `winget install jqlang.jq`)
- Git-Repo

### Install — Option 1: Plugin-Marketplace (empfohlen für Claude Code)

```bash
/plugin marketplace add snarktank/ralph
/plugin install ralph-skills@ralph-marketplace
```

Skills danach:
- `/prd` — generiert Product Requirements Documents mit Klärfragen
- `/ralph` — konvertiert PRD-Markdown nach `prd.json`

Auto-Trigger: "create a prd", "write prd for", "plan this feature", "convert this prd", "turn into ralph format".

### Install — Option 2: Manuell ins Repo

```bash
mkdir -p scripts/ralph
# Aus github.com/snarktank/ralph holen:
cp /path/to/ralph/ralph.sh scripts/ralph/
cp /path/to/ralph/CLAUDE.md scripts/ralph/CLAUDE.md  # Claude-Code-Variante
chmod +x scripts/ralph/ralph.sh
```

### Workflow

```bash
# Mit Claude Code
./scripts/ralph/ralph.sh --tool claude 10   # 10 Iterations

# Mit Amp (default)
./scripts/ralph/ralph.sh 10
```

Pro Iteration:
1. Feature-Branch erstellt (aus PRD `branchName`)
2. Höchste Priorität Story mit `passes:false` ziehen
3. Eine Story implementieren
4. Quality-Checks (typecheck, test)
5. Wenn grün: commit, `prd.json` updaten (`passes:true`), `progress.txt` append
6. Wiederholen bis alle pass oder max-iterations

### Pflicht-Anpassungen für jedes neue Projekt

In `CLAUDE.md` (Repo-Root, nicht Ralph-Skill-CLAUDE.md):
- Projekt-spezifische Quality-Check-Befehle (`npm test`, `npm run typecheck`, etc.)
- Codebase-Konventionen (Naming, File-Layout)
- Common Gotchas für den Stack (siehe `04_MEMORY_LEARNINGS.md`)

In `AGENTS.md` (Repo-Root): leer starten, Ralph trägt Discovered Patterns / Gotchas / Useful Context selbst nach jeder Iteration ein.

### Stop-Condition

Wenn alle Stories `passes:true`, gibt Ralph `<promise>COMPLETE</promise>` aus und der Loop exitet.

### Debugging

```bash
cat prd.json | jq '.userStories[] | {id, title, passes}'
cat progress.txt
git log --oneline -10
```

### Amp-Auto-Handoff (falls Amp statt Claude Code)

In `~/.config/amp/settings.json`:
```json
{ "amp.experimental.autoHandoff": { "context": 90 } }
```

---

## §2 Caveman (`JuliusBrussee/caveman`)

**Was er macht:** Komprimiert Agent-Output ~65% bei voller technischer Genauigkeit. Spart Tokens, beschleunigt Antworten, macht Output lesbarer.

### Install

```bash
# macOS / Linux / WSL / Git Bash
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash

# Windows PowerShell 5.1+
irm https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.ps1 | iex
```

Findet alle installierten Agents (Claude Code, Codex, Gemini, Cursor, Windsurf, Cline, Copilot, 30+) und installiert für jeden. ~30 Sekunden. Node ≥18 nötig. Re-run sicher.

### Trigger

- `/caveman` (im Agent tippen)
- "talk like caveman" (Konversation)
- `normal mode` (zurück)

### Modi

| Modus | Token-Save | Wann |
|---|---:|---|
| `/caveman lite` | ~30% | Default für Coding-Sessions |
| `/caveman full` | ~65% | Default-Caveman, Status-Reviews |
| `/caveman ultra` | ~75% | Telegraphic, sehr fokussierte Sessions |
| `/caveman wenyan` | ~80% | Klassisches Chinesisch — Nische |

### Sub-Skills

- `/caveman-commit` — Conventional-Commit-Messages, ≤50 char subject, Why-over-What
- `/caveman-review` — One-Line-PR-Comments: `L42: 🔴 bug: user null. Add guard.`
- `/caveman-stats` — Real session token usage + lifetime savings + USD
- `/caveman-compress <file>` — rewrite Memory-File in Caveman-Speak, spart ~46% Input-Tokens jede Session, Code/URLs/Paths byte-preserved

### Statusline-Badge

Claude Code zeigt `[CAVEMAN] ⛏ 12.4k` (lifetime Tokens gespart). Setzen mit `/caveman-stats`, abschalten mit `CAVEMAN_STATUSLINE_SAVINGS=0`.

### Ecosystem (optional, später)

- `caveman` — Output-Kompression (Standard)
- `cavemem` — Cross-Agent-Memory (https://github.com/JuliusBrussee/cavemem)
- `cavekit` — Spec-driven Build-Loop (https://github.com/JuliusBrussee/cavekit)

---

## §3 Claude Code CLI

**Install:**
```bash
npm install -g @anthropic-ai/claude-code
claude login   # einmalig
```

**Pflicht-Files im Repo:**
- `CLAUDE.md` (Root) — wird automatisch geladen
- `AGENTS.md` (Root) — wird automatisch geladen
- Skills unter `~/.claude/skills/` (für globale Skills wie `/prd`, `/ralph`, `/caveman`)

**Run:**
```bash
claude                            # Interactive
claude -p "Add E2E test for login"   # One-shot
```

---

## §4 gh CLI

**Install:**
```bash
# Windows
winget install --id GitHub.cli

# macOS
brew install gh
```

**Login:**
```bash
gh auth login   # GitHub.com, HTTPS, browser-based
```

**Häufige Calls:**
```bash
gh repo create {{REMOTE_NAME}}/{{PROJEKT_NAME}} --private --source=. --remote=origin --push
gh pr create --base main --title "feat: ..." --body "..."
gh pr list --state open
gh run list --workflow=deploy --limit 5
gh run view {{run_id}} --log
gh workflow run deploy-hetzner.yml
```

---

## §5 MCP-Connectors

In Cowork via Settings → Connectors aktivieren.

### Pflicht (alle Projekte)

| Connector | Zweck |
|---|---|
| Gmail | Drafts anlegen für Anwalts-/Behörden-Mails (`create_draft`, `search_threads`) — Auto-Send NIE |
| Calendar | Termin-Tracking für Investor-/Behörden-Calls |
| claude-in-chrome | Web-Aktionen (Hetzner, Stripe, GitHub-Web) — falls Chrome-Extension installiert |
| computer-use | Native-App-Aktionen (Excel, Mail-Client, System-Settings) |
| mcp-registry | `search_mcp_registry`, `suggest_connectors` — Self-Service neue Connectors finden |
| session_info | `list_sessions`, `read_transcript` — Memory aus älteren Sessions |
| skills | `list_skills`, `suggest_skills` — Skills entdecken / installieren |
| workspace | `bash`, `web_fetch` — Linux-Sandbox + raw HTTP |
| cowork | `create_artifact`, `update_artifact`, `present_files` — persistente Status-Pages |
| scheduled-tasks | `create_scheduled_task` — Daily-Briefings, Reminders |

### Optional (nach Domain)

| Connector | Wann |
|---|---|
| Slack / Discord | Team-Projekte |
| Linear / Asana / Jira | Echtes Issue-Tracking |
| Notion | Doc-Workspace |
| Stripe MCP | Pricing/Invoice-Automation |
| Figma | Design-Handoffs |
| PDF-Tools | Forms ausfüllen, Verträge mergen |
| PopHIVE (Public Health) | Medical-Projekte mit US-Daten |

### Custom MCP für Projekt-Connectoren

Wenn der Projekt-Stack eine eigene API hat (z. B. eigenes Praxis-System), eigenen MCP-Server bauen mit `@modelcontextprotocol/sdk` und in Cowork als lokalen Server eintragen.

---

## §6 Hosting-CLIs

### flyctl (Fly.io)

```bash
# Install Windows PowerShell
iwr https://fly.io/install.ps1 -useb | iex

# macOS / Linux
curl -L https://fly.io/install.sh | sh
```

Häufige Calls:
```bash
flyctl auth login
flyctl launch                            # init mit fly.toml
flyctl deploy --remote-only              # Build-on-Fly, kein lokaler Docker nötig
flyctl secrets set DATABASE_URL="..."
flyctl logs -a {{APP_NAME}}
flyctl status -a {{APP_NAME}}
flyctl scale count 1 -a {{APP_NAME}}
flyctl machine restart {{machine_id}}
```

Cold-Start-Fix in `fly.toml`:
```toml
[[services]]
  internal_port = 3001
  protocol = "tcp"
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
```

### netlify CLI

```bash
npm install -g netlify-cli
netlify login
netlify init                             # link Repo zu Site
netlify deploy --prod                    # production Deploy
netlify env:set DATABASE_URL "..." --context production
```

### neonctl (Neon Postgres)

```bash
npm install -g neonctl
neonctl auth
neonctl projects list
neonctl branches create --name preview-pr-42 --project-id {{id}}
neonctl connection-string preview-pr-42
```

Branches per PR sind Pattern für sichere Migrations-Tests.

### Stripe CLI

```bash
# nur wenn Pricing aktiv
stripe login
stripe listen --forward-to localhost:3001/webhooks/stripe
stripe trigger payment_intent.succeeded
```

---

## §7 Repo-Struktur (ASCII-Tree)

Default-Layout für ein Monolith-Projekt (Frontend + Backend in einem Repo):

```
{{PROJEKT_NAME}}/
├── .devcontainer/
│   └── devcontainer.json              # Codespace-Fallback (4-core/16GB)
├── .github/
│   └── workflows/
│       ├── ci.yml                     # lint + typecheck + test + build
│       ├── deploy-backend.yml         # Fly.io
│       └── deploy-frontend.yml        # Netlify
├── apps/                              # Optional bei Workspaces
│   ├── frontend/
│   └── backend/
├── packages/                          # Optional, geteilter Code
│   └── shared/
├── docs/                              # Architektur, Decision-Records, externe Briefe
│   ├── ARCHITECTURE.md
│   ├── DEPLOY.md
│   └── decisions/
├── memory/
│   └── runs/                          # YYYY-MM-DD_<agent>-<model>-<NN>.md
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── scripts/
│   └── ralph/
│       ├── ralph.sh
│       └── CLAUDE.md
├── tasks/                             # PRD-Markdown-Files
│   └── prd-<feature>.md
├── tests/
│   ├── e2e/
│   ├── integration/
│   └── unit/
├── src/
│   ├── client/
│   ├── server/
│   └── shared/
├── .env.example
├── .gitignore
├── AGENTS.md                          # Discovered Patterns (Ralph füllt)
├── CLAUDE.md                          # Projekt-Prompt + DoD
├── {{PROJEKT_NAME}}_MASTER.md          # CK Daily-Driver
├── VERSTAENDNIS_LUECKEN.md            # Q1, Q2, ... offene Fragen
├── docker-compose.local.yml
├── fly.toml
├── netlify.toml
├── package.json
├── prd.json                           # Ralph-Loop-State
├── progress.txt                       # Ralph-Loop-Learnings
├── README.md
└── tsconfig.json
```

---

## §8 Was NICHT zum Default gehört (mit Begründung)

| Tool / Pattern | Warum nicht |
|---|---|
| `husky` Git-Hooks | Auf CK-Windows-Maschine Flaky. Quality-Checks lieber im CI. |
| `commitizen` | Conventional Commits sind Konvention, brauchen kein Tool. Caveman-Commit reicht. |
| `lint-staged` | Lint im CI ist robuster. |
| `lefthook` | Wie husky — Plattform-Reibung. |
| Storybook | Erst wenn Design-System echt vorhanden. Initial Overkill. |
| Sentry / Datadog | Erst bei Prod-Traffic. Free-Tier-Fallen kosten Zeit. |
| `next-auth` + Custom | Auth ist meist JWT + Argon2, kein Social-Login-Bedarf in B2B-Klinik-Stack. |
| Plausible / Posthog | Erst bei MVP-Validierung mit echten Nutzern. |
| GitHub Issues als Tracker | Master-Cockpit + `tasks/` reichen für Solo+1-Agent-Setup. |
| Asana / Jira | Wie oben — Overhead für Solo. |
| Docker Compose für Prod | Fly.io macht das. Compose nur lokal. |
| `pre-commit` (Python-Tool) | Quality-Checks im CI sind robuster. |
| `dotenv-vault` | `.env.example` + Secrets in Hosting-Provider reicht. |
| ESLint + Prettier doppelte Regeln | Nur eines — Prettier für Format, ESLint für Lint, keine Style-Regeln in ESLint. |
| Multiple Lock-Files | Nur `package-lock.json` committen. |
| `nodemon` separat | `tsx watch` oder Vite-HMR reicht. |
