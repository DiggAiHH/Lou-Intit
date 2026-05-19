# 01 — TECH STACK

Default-Stack, der in den letzten Projekten (DiggAi, Jobetes, Hetzner-Wundzentrum-Demo) trägt. Wo eine Variante zur Diskussion steht, ist es markiert. Am Ende dieser Datei eine **Stack-Tabelle zum Ausfüllen** — CK trägt vor Bootstrap-Start ein, was er für dieses Projekt will.

---

## §1 Runtime

| Aspekt | Default | Begründung |
|---|---|---|
| Node-Version | `20 LTS` (NICHT 24) | Node 24 + Prisma 6 + tsx haben ESM-Inkompatibilitäten (default-Import-Patch nötig). 20 ist stabil. |
| Package-Manager | `npm` | Default. Yarn/pnpm nur wenn explizit nötig — npm hat weniger Reibung mit Windows-cmd. |
| Workspaces | `npm workspaces` (in `package.json` root) | Mono-Repo-Default. Aktivieren erst nach Build-Test (Phase 1b in DiggAi hatte OOM). |
| TypeScript | `5.4+`, strict mode | Default. `tsc -b` peakt 6–8 GB → `NODE_OPTIONS=--max-old-space-size=8192`. |

---

## §2 Frontend

| Aspekt | Default | Alternative (wann) |
|---|---|---|
| Framework | `React 19` + `TypeScript` | Vue 3 nur wenn Team-Setup das verlangt |
| Bundler | `Vite 5+` | Next.js wenn SSR/SSG zwingend |
| Styling | `Tailwind CSS 3.4+` | Mit Dark-Mode via `dark:`-Variante. Keine inline CSS-in-JS. |
| State | React Hooks first, dann `Zustand`/`Jotai` | Redux nur in Legacy |
| i18n | `i18next` + `react-i18next` | **Pflicht-Konfig:** `keySeparator: false, nsSeparator: false` falls flat dotted keys (DiggAi-Gotcha) |
| Forms | `react-hook-form` + `zod` | Default |
| Routing | `react-router-dom 6+` oder File-Router (Next) | — |
| Icons | `lucide-react` | Heroicons nur wenn Design-System es vorgibt |

---

## §3 Backend

| Aspekt | Default | Alternative |
|---|---|---|
| Framework | `Express 5` (TypeScript) | Fastify wenn Performance-kritisch, FastAPI (Python) für Agent/AI-Logik |
| ORM | `Prisma 6` | Drizzle wenn raw-SQL-näher gewünscht |
| Validation | `zod` | Auch im Frontend dieselben Schemas |
| Auth | `JWT` + Argon2 (Passwort) | Auth.js wenn Social-Login |
| Verschlüsselung | AES-256-GCM (32-Byte Key) | Pflicht bei DSGVO Art. 9 |
| API-Stil | REST (Express) | tRPC nur wenn Full-Stack-TS und kein Multi-Client |
| Logging | `pino` + Strukturiertes JSON | Datadog/Loki je nach Hosting |

---

## §4 Datenbank

| Aspekt | Default | Alternative |
|---|---|---|
| Engine | PostgreSQL 16 | MySQL nur in Legacy, SQLite für Prototyp |
| Cloud-Host | **Neon** (eu-central-1 / Frankfurt) | Supabase wenn auch Auth nötig, RDS wenn AWS-Stack |
| Migrations | `prisma migrate` (dev/deploy) | — |
| Branching für Tests | Neon-Branch pro PR | Saubere Migrations-Tests ohne Prod zu kippen |
| Backup | Neon PITR (Point-in-Time-Recovery) 7 Tage | Manual `pg_dump` für Snapshot vor riskanten Migrations |
| Lokales Dev | `docker-compose` mit `postgres:16` + `redis:7` | Native nur wenn Docker-Setup fehlt |

---

## §5 Hosting

| Aspekt | Default | Wann anders |
|---|---|---|
| Backend / API | **Fly.io** (Region `fra` / Frankfurt — DSGVO) | Hetzner Docker wenn Self-Host-Pflicht oder Kosten ein Faktor |
| Frontend / SPA | **Netlify** | Vercel wenn Next.js, Cloudflare Pages für statisches |
| Custom-Domain | Beide via Netlify-/Fly-DNS-Records | DNS-Provider egal (Cloudflare empfohlen für Caching + Bot-Schutz) |
| TLS | Auto (Let's Encrypt via Provider) | Hetzner: Caddy autocert |
| Region | EU-Frankfurt durchgängig | Bei DSGVO Art. 9 zwingend, sonst Empfehlung |

---

## §6 CI / Deploy

| Aspekt | Default |
|---|---|
| CI | GitHub Actions |
| Pipeline | lint → typecheck → test → build → deploy |
| Branches | `main` = Prod, `develop` = Staging (optional), Feature-Branches `feat/<slug>` |
| Deploy-Auslöser | Push auf `main` → Fly+Netlify automatisch |
| Secrets | GitHub Actions Secrets + Fly `flyctl secrets set` + Netlify Env-Vars |
| Hetzner-Variante | `deploy-hetzner.yml` mit SSH + Docker-Compose pull |

---

## §7 Lokale Dev

| Aspekt | Default |
|---|---|
| OS-Annahme | Windows 10/11 (CK-Maschine) — Pfade mit Backslash in cmd, kein `&&`, kein `??` |
| Shell | `cmd.exe` (Default) + `Git Bash` für Scripts. PowerShell nur explizit. |
| Editor | VS Code mit ESLint/Prettier/TS-Plugins |
| Multi-Line-Commits | `.commitmsg.txt`-Trick (siehe `04_MEMORY_LEARNINGS.md`) |
| Dev-Server | `npm run dev:all` (Concurrently: Frontend `:5173` + Backend `:3001`) |
| Docker | `docker-compose -f docker-compose.local.yml up -d` für DB+Redis |
| Codespace-Fallback | `.devcontainer/devcontainer.json` mit `node:20`, 4-core/16 GB beim OOM-Fall |

---

## §8 Was wir NICHT nutzen (bewusst)

| Tool | Warum nicht |
|---|---|
| Yarn / pnpm | Mehr Reibung mit Windows-cmd, npm reicht. Keine echten Performance-Wins für Projekt-Größen ≤500 deps. |
| Webpack direkt | Vite/Next decken alles ab, Webpack-Config ist Wartungs-Last. |
| Redux | React-Hooks + Zustand für 99% der Fälle. Redux nur in Legacy. |
| MongoDB | Schema-Drift killt jedes regulierte Projekt. Postgres + JSONB wenn Flex-Schema nötig. |
| Heroku | EU-Region inkonsistent, teurer als Fly. |
| Vercel + Postgres | Vercel-DB ist young, Neon ist mature. |
| OpenAPI-First-Code-Gen | Manuelle Route + zod-Schema reicht für Projektgrößen ≤50 Endpunkte. |
| Microservices | Default-Architektur ist Modulith (1 Backend + 1 Agent-Service falls AI). Aufsplitten erst bei echtem Skalierungsgrund. |
| Kubernetes | Fly.io macht den 95%-Fall ohne. K8s erst bei eigenem Cluster. |
| GraphQL | REST + zod. GraphQL nur wenn Mobile-Client + Public-API. |
| jQuery / Bootstrap | Modern-Stack hat Tailwind + React. |

---

## §9 Tools im Workflow (täglich angefasst)

| Tool | Zweck |
|---|---|
| Ralph (`snarktank/ralph`) | Autonomer Loop, PRD → prd.json → fresh-context Iterations |
| Caveman (`JuliusBrussee/caveman`) | Output-Token-Kompression, Memory-File-Shrinking, Stats-Statusline |
| Claude Code CLI | `claude` lokal — Coding-Agent, liest CLAUDE.md automatisch |
| `gh` CLI | GitHub PRs, Actions, Workflow-Runs prüfen |
| `flyctl` | Fly.io Deploy + Secrets + Logs |
| `netlify` CLI | Site-Deploy, Functions, Env-Vars |
| `neonctl` (optional) | Neon-Branches per CLI, sonst Web-UI |
| `prisma` CLI | Migrations + Studio (DB-GUI) |
| Cowork (Anthropic Desktop) | CK-Daily-Driver, Multi-Tab-Sessions, Artifacts, Connectors |

---

## §10 Stack-Tabelle für CK vor Bootstrap-Start

Bitte einmal durchgehen und für `{{PROJEKT_NAME}}` festlegen. Felder mit `(D)` sind Default, einfach übernehmen wenn nichts dagegen spricht.

| Aspekt | Wert |
|---|---|
| Node-Version | `{{NODE_VERSION_20_LTS_D | 18 | 22}}` |
| Frontend-Framework | `{{REACT_D | VUE | NEXT}}` |
| Bundler | `{{VITE_D | NEXT | ASTRO}}` |
| Styling | `{{TAILWIND_D | CSS_MODULES | STYLED_COMPONENTS}}` |
| Backend-Framework | `{{EXPRESS_D | FASTIFY | FASTAPI_PYTHON}}` |
| ORM | `{{PRISMA_D | DRIZZLE | RAW_SQL}}` |
| DB-Engine | `{{POSTGRES_16_D | MYSQL | SQLITE}}` |
| DB-Host | `{{NEON_D | SUPABASE | LOKAL_DOCKER}}` |
| Backend-Host | `{{FLY_FRA_D | HETZNER_DOCKER | RAILWAY}}` |
| Frontend-Host | `{{NETLIFY_D | VERCEL | CLOUDFLARE_PAGES}}` |
| CI | `{{GITHUB_ACTIONS_D | GITLAB_CI}}` |
| Auth | `{{JWT_HOMEMADE_D | AUTH_JS | CLERK}}` |
| i18n | `{{I18NEXT_D | NONE | OTHER}}` |
| Multi-Tenant | `{{JA_SUBDOMAIN | JA_PATH | NEIN_D}}` |
| Pricing-Strategie | `{{B2B_PER_SEAT | B2B_FLAT | B2C_FREEMIUM | INTERNAL_NONE_D}}` |
| Stripe-Integration | `{{JA | NEIN_D}}` |
| Domain-Regulatorik | `{{KEINE_D | MEDICAL_DEVICE | DSGVO_ART_9 | FINTECH}}` |
| Mehrsprachig nötig? | `{{JA_DE_EN | JA_DE | NEIN_D}}` |
| Dark-Mode-Support | `{{JA_D | NEIN}}` |

Sobald die Tabelle ausgefüllt ist, kann der System-Prompt aus `00_SYSTEM_PROMPT_BOOTSTRAP.md` ohne weitere Klärfragen Phase 0 starten.
