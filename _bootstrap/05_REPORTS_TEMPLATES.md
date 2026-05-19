# 05 — REPORTS & TEMPLATES

13 Skelette zum Kopieren. Jedes für einen wiederkehrenden Output. Platzhalter sind `{{NAME}}`-Format, Pflicht-Felder oben, Optional-Felder unten.

---

## §1 `CLAUDE.md` (Repo-Root)

```markdown
# CLAUDE.md

Projekt-Prompt für Claude Code / Ralph / Opus 4.7. Wird automatisch in jeder Coding-Session geladen.

---

## Projekt-Identität

- **Name:** {{PROJEKT_NAME}}
- **Kurz:** {{KURZBESCHREIBUNG_2_SAETZE}}
- **Stack:** {{STACK_KURZ}} (Details: siehe `_bootstrap/01_TECH_STACK.md`)
- **Live-URLs:**
  - Frontend: `https://{{FRONTEND_DOMAIN}}`
  - Backend: `https://{{BACKEND_DOMAIN}}`
  - DB: Neon `{{NEON_PROJECT_ID}}` (eu-central-1 / Frankfurt)
- **Repo:** `https://github.com/{{REMOTE_NAME}}/{{PROJEKT_NAME}}`

---

## Quick Start

\`\`\`bash
git clone https://github.com/{{REMOTE_NAME}}/{{PROJEKT_NAME}}.git
cd {{PROJEKT_NAME}}
cp .env.example .env   # DATABASE_URL, JWT_SECRET (32+), ENCRYPTION_KEY (32) setzen
docker-compose -f docker-compose.local.yml up -d
npx prisma migrate dev --name init
npm install
npm run dev:all   # Frontend :5173 + Backend :3001
\`\`\`

---

## Definition of Done (11 Punkte)

1. Code geschrieben
2. Typecheck grün
3. Tests grün
4. Lint grün
5. Build grün
6. Manueller Smoke-Test
7. Commit (Conventional Commits)
8. Push auf richtigen Remote
9. `prd.json` updated
10. Run-Log-Eintrag in `memory/runs/`
11. Master-Cockpit nachgezogen

---

## Quality-Checks (vor jedem Commit)

\`\`\`bash
npm run lint
npm run typecheck
npm test
npm run build
\`\`\`

---

## Run-Log-Konvention

Pfad: `memory/runs/<YYYY-MM-DD>_<agent>_<model>-<NN>.md`

Format (5 Bullets exakt):
\`\`\`
YYYY-MM-DDTHH:MM+02:00 | Lauf <agent>-<NN> | <topic>
---
- Aktion: ...
- Blocker: ...
- Fix: ...
- Ergebnis: <commit hash / file path>
- Out: <verified state>
\`\`\`

---

## Memory / Learnings

(Aus `_bootstrap/04_MEMORY_LEARNINGS.md` § 6 — kopieren)

---

## Pflicht-Lektüre bei Session-Start

1. `CLAUDE.md` (diese Datei)
2. `AGENTS.md`
3. Letzte 3 Files in `memory/runs/` (sort by filename desc)
4. `{{PROJEKT_NAME}}_MASTER.md`
5. `VERSTAENDNIS_LUECKEN.md`

---

## Niemals

- Niemals `git push` ohne `git status`
- Niemals Force-Push auf shared Branches
- Niemals Auto-Send von Mails
- Niemals echte Secrets / Tokens commiten
- Niemals direkt auf `main` ohne PR
```

---

## §2 `AGENTS.md` (Repo-Root)

```markdown
# AGENTS.md

Discovered Patterns, Gotchas, Useful Context, File Ownership und Quality Gates für dieses Projekt. Wird von Ralph nach jeder Iteration ergänzt.

---

## Discovered Patterns

(Ralph trägt hier Patterns ein, die in der Codebase wiederholt vorkommen)

- {{Beispiel: "Diese Codebase nutzt zod-Schemas für API-Validation + Frontend-Forms — Schema in `shared/schemas/` einmal definieren, beide Seiten importieren"}}

---

## Gotchas

(Ralph trägt hier ein, was beim ersten Mal schief gegangen ist)

- {{Beispiel: "Wenn `prisma generate` läuft während Backend-Watch aktiv ist, hängt der Prozess — vorher mit Strg+C stoppen"}}

---

## Useful Context

(Wo finde ich was)

- Auth-Logik: `src/server/auth/`
- DB-Schema: `prisma/schema.prisma`
- Shared Types: `src/shared/types.ts`
- E2E-Tests: `tests/e2e/`
- Deployment-Config: `fly.toml`, `netlify.toml`

---

## File Ownership

| Pfad | Wer pflegt |
|---|---|
| `prisma/schema.prisma` | Backend-Lead |
| `src/i18n/` | Frontend + Translations-Pool |
| `docs/decisions/` | Architektur-Owner |

---

## Quality Gates

Pro Commit:
- `npm run lint` keine neuen Warnings
- `npm run typecheck` clean
- `npm test` grün

Pro PR:
- E2E-Tests laufen in CI
- Coverage ≥80% für neue Files
- Code-Review von 1 Co-Owner (oder Self-Review mit Checkliste)
```

---

## §3 Run-Log-Template (`memory/runs/`)

Pfad: `memory/runs/<YYYY-MM-DD>_<agent>_<model>-<NN>.md`

```markdown
{{ISO-TIMESTAMP-MIT-TZ}} | Lauf {{AGENT}}-{{NN}} | {{ONE-LINE-TOPIC}}
---
- Aktion: {{Was konkret passiert ist}}
- Blocker: {{Was hat aufgehalten oder "—"}}
- Fix: {{Wie gelöst oder "—"}}
- Ergebnis: {{Commit hash / PR-Nummer / File-Path}}
- Out: {{Verifizierter State — "tests grün", "PR #12 open", "blocked on Q5"}}
```

**Beispiel:**

```markdown
2026-05-19T14:32+02:00 | Lauf opus-01 | Bootstrap Phase 0.3 - Ralph+Caveman installiert
---
- Aktion: ralph-marketplace + caveman install.sh ausgeführt, scripts/ralph/ralph.sh chmod +x
- Blocker: jq nicht installiert auf Windows
- Fix: winget install jqlang.jq, Terminal neu gestartet
- Ergebnis: Commit f3a8b21 "chore: install ralph + caveman tooling"
- Out: ralph.sh --help OK, /caveman trigger getestet
```

---

## §4 Master-Cockpit-Template (`{{PROJEKT_NAME}}_MASTER.md`)

≤95 Zeilen. CK Daily-Driver.

```markdown
# {{PROJEKT_NAME}} — MASTER COCKPIT

**Stand:** {{DATUM}}
**Phase:** {{PHASE_AKTUELL}} ({{PHASE_PROZENT_FERTIG}}%)

---

## Was tust du jetzt? (CK Pareto-3)

| # | Wer | Was | Bis | Status |
|---|---|---|---|---|
| 1 | CK | {{TASK_1}} | {{ETA_1}} | 🔴 / 🟡 / ✅ |
| 2 | CK | {{TASK_2}} | {{ETA_2}} | 🔴 / 🟡 / ✅ |
| 3 | Agent | {{TASK_3}} | {{ETA_3}} | 🔴 / 🟡 / ✅ |

---

## Live-Status

| Komponente | URL | Status |
|---|---|---|
| Frontend | `https://{{FRONTEND_DOMAIN}}` | ✅ |
| Backend | `https://{{BACKEND_DOMAIN}}` | ✅ |
| DB | Neon `{{PROJECT_ID}}` eu-central-1 | ✅ |
| CI | GitHub Actions | 🟢 last run green |

---

## Offene Items (Top-10, Rest im Tracker)

| ID | Titel | Owner | Priorität |
|---|---|---|---|
| {{ID_1}} | {{TITEL_1}} | {{OWNER_1}} | {{P_1}} |
| ... | ... | ... | ... |

(Vollständig in `{{PROJEKT_NAME}}-Open-Items-Tracker.xlsx` / `Open-Items.md`)

---

## Letzter Run-Log

`{{LATEST_RUN_LOG_PFAD}}` — {{LATEST_RUN_LOG_ONELINE}}

---

## Verständnis-Lücken

Offen: {{ANZAHL_OFFENE_QUESTIONS}} ({{Q-LIST}})

(Details in `VERSTAENDNIS_LUECKEN.md`)

---

## Next Milestones

- {{MILESTONE_1}} — {{DATUM_1}}
- {{MILESTONE_2}} — {{DATUM_2}}
- {{MILESTONE_3}} — {{DATUM_3}}
```

---

## §5 PRD-Markdown-Template (`tasks/prd-{{slug}}.md`)

```markdown
# PRD: {{FEATURE_NAME}}

**Slug:** `{{slug-kebab-case}}`
**Branch:** `feat/{{slug-kebab-case}}`
**Owner:** {{OWNER}}
**Erstellt:** {{DATUM}}

---

## Why

{{1-2 Sätze: warum dieses Feature, was ist der Pain-Point}}

---

## Acceptance Criteria (messbar)

- [ ] {{AC_1: messbar, testbar — z.B. "User kann sich mit Tel-Nummer einloggen, JWT wird ausgestellt"}}
- [ ] {{AC_2}}
- [ ] {{AC_3}}

---

## User Stories

| ID | Story | Estimated Iterations | Passes |
|---|---|---|---|
| US-01 | As {{role}}, I want {{thing}}, so that {{benefit}}. | 1 | false |
| US-02 | ... | 1 | false |
| US-03 | ... | 1-2 | false |

(Konvertierung nach `prd.json` via `/ralph`-Skill)

---

## Out of Scope (explizit nicht!)

- {{Was bewusst NICHT in diesem PRD ist — z.B. "Social-Login (Google/Apple) — separates PRD"}}

---

## Dependencies

- {{Andere PRDs / Externes — z.B. "DB-Migration für `patient.phone` muss vorher laufen (PRD-007)"}}

---

## Verification Steps (für Browser-Stories)

- [ ] {{Manueller Smoke-Test-Schritt 1}}
- [ ] {{Smoke-Test 2}}
- [ ] {{E2E-Test in `tests/e2e/<slug>.spec.ts` erstellt}}
```

---

## §6 `progress.txt` (Repo-Root)

Append-only. Ralph schreibt nach jeder Iteration eine Zeile rein.

```
2026-05-19T14:32: US-01 done — JWT-Issuance OK, edge-case "expired-token" needs separate test (US-04 added)
2026-05-19T15:18: US-02 partial — Phone-Validation works for DE, AT/CH needs separate format-handling
2026-05-19T15:55: US-04 done — expired-token returns 401 with WWW-Authenticate header
```

Format: ISO-Timestamp + Story-ID + Status + Inline-Learning. Eine Zeile pro Eintrag.

---

## §7 `VERSTAENDNIS_LUECKEN.md` (Repo-Root)

```markdown
# Verständnis-Lücken

Offene Fragen, die der Agent selbst nicht beantworten kann. CK schließt mit `RESOLVED`.

---

## Q1 (2026-05-19): Soll Tel-Nummer als Pflichtfeld?

**Kontext:** Login-Flow PRD-001. DE-Klinik will Tel, AT-Klinik nicht.

**Optionen:**
- A) Pflicht überall — einfach, aber AT blockt
- B) Pflicht per Tenant-Setting — flexibel, mehr Config
- C) Optional überall — Datenqualität sinkt

**Empfehlung:** B

**Status:** OPEN

---

## Q2 (2026-05-19): Auth-Token-Refresh-Strategie?

**Kontext:** ...

**Optionen:**
- A) Sliding-Window 1h, refresh bei jedem Request
- B) Fixed 24h, harter Logout
- C) Refresh-Token-Pair

**Empfehlung:** A

**Status:** RESOLVED (2026-05-19 — CK entscheidet B, weil Security-Audit-Anforderung gegen Token-Leak nach Stehlen)
```

---

## §8 `.devcontainer/devcontainer.json`

```json
{
  "name": "{{PROJEKT_NAME}} Dev",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:1-20-bullseye",
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {},
    "ghcr.io/devcontainers/features/common-utils:2": {
      "installZsh": true,
      "configureZshAsDefaultShell": true
    }
  },
  "forwardPorts": [3001, 5173, 5432],
  "postCreateCommand": "npm install --legacy-peer-deps && npx prisma generate",
  "remoteEnv": {
    "NODE_OPTIONS": "--max-old-space-size=4096"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "prisma.prisma",
        "bradlc.vscode-tailwindcss"
      ]
    }
  },
  "hostRequirements": {
    "cpus": 4,
    "memory": "16gb",
    "storage": "32gb"
  }
}
```

Secrets im Codespace-Settings hinterlegen (NICHT in dieser Datei):
- `DATABASE_URL`
- `JWT_SECRET` (32+ chars)
- `ENCRYPTION_KEY` (32 chars)
- `FLY_API_TOKEN`
- `NETLIFY_AUTH_TOKEN`
- Projekt-spezifische Keys

---

## §9 BfArM-Sprechstunde-Mail (Template, generisch)

```
An: mp-beratung@bfarm.de
Betreff: Antrag auf Beratungs-Sprechstunde — Klassifizierung {{PRODUKT_NAME}}

Sehr geehrte Damen und Herren,

ich beantrage einen Termin in der kostenlosen Sprechstunde des BfArM zur Klassifizierung der Software {{PRODUKT_NAME}} nach MDR (Verordnung (EU) 2017/745).

Antragsteller:
- Name: {{VOLL_NAME}}
- Rechtsform: {{RECHTSFORM}}
- Anschrift: {{ANSCHRIFT_VOLL}}
- Telefon: {{TELEFONNUMMER}}
- E-Mail: {{EMAIL}}

Produkt-Beschreibung (Intended Purpose):
{{INTENDED_PURPOSE_3_5_SAETZE}}

Funktionsumfang (Kernfunktionen):
- {{FUNKTION_1}}
- {{FUNKTION_2}}
- {{FUNKTION_3}}

Was ist ausdrücklich NICHT Funktion (Grenzziehung):
- {{NICHT_FUNKTION_1}}
- {{NICHT_FUNKTION_2}}

Klassifizierungs-Hypothese:
Wir sehen das Produkt als {{KLASSE_HYPOTHESE — z.B. "Class I nach Regel 11 MDR Anhang VIII"}} unter Selbstverifizierung.

Begründung:
{{KURZE_REGEL_BEGRUENDUNG}}

Wir freuen uns über Ihren Rückruf oder Terminvorschlag.

Mit freundlichen Grüßen
{{VOLL_NAME}}
{{TITEL}}
{{ORGANISATION}}
{{TELEFONNUMMER}}
```

---

## §10 Anwalts-Erstkontakt-Mail (Template, generisch)

```
An: {{KANZLEI_EMAIL}}
Betreff: Erstgespräch — Klassifizierung Software {{PRODUKT_NAME}} (MDR/DSGVO)

Sehr geehrte Damen und Herren {{ANREDE_FALLS_BEKANNT}},

ich suche eine Erstberatung zur regulatorischen Einordnung der Software "{{PRODUKT_NAME}}" und zur DSGVO-konformen Vertragsgestaltung.

Hintergrund in zwei Sätzen:
{{HINTERGRUND_2_SAETZE}}

Konkrete Fragen für das Erstgespräch:
1. Klassifizierung nach MDR Anhang VIII (Regel 11) — unsere Hypothese ist {{KLASSE_HYPOTHESE}}. Stützen Sie das?
2. Welche Pflicht-Dokumente sind für die Selbstverifizierung mindestens nötig?
3. Welche AVV-Templates empfehlen Sie für Sub-Prozessoren (Hosting US-EU SCC + TIA)?
4. {{FRAGE_4_DOMAIN_SPEZIFISCH}}

Anhang:
- {{DOCX_1 — z.B. Konformitätserklärung-Entwurf v0.1}}
- {{DOCX_2 — z.B. Intended-Purpose-Statement}}

Verfügbarkeit:
{{TERMINE_PRAEFERIERT — z.B. "Mo-Fr 15-17 Uhr, kommende Woche auch ganztägig"}}

Mit freundlichen Grüßen
{{VOLL_NAME}}
{{TITEL}}
{{ORGANISATION}}
{{TELEFONNUMMER}}
```

---

## §11 Status-Plan-Flip-Template (`docs/STATUS_PLAN_FLIP_{{DATUM}}.md`)

Wenn Strategie/Regulatorik/Roadmap fundamental kippt.

```markdown
# Status-Plan-Flip {{DATUM}}

## Zusammenfassung in 4 Sätzen

{{4_SAETZE_TLDR}}

## Vorher (alter Plan)

- Klassifizierung: {{ALT_KLASSE}}
- Hosting: {{ALT_HOSTING}}
- Roadmap: {{ALT_ROADMAP}}

## Nachher (neuer Plan)

- Klassifizierung: {{NEU_KLASSE}}
- Hosting: {{NEU_HOSTING}}
- Roadmap: {{NEU_ROADMAP}}

## Was hat den Flip ausgelöst?

{{TRIGGER_1_2_ABSAETZE}}

## 3 Hebel die den Flip technisch ermöglichen

1. **{{HEBEL_1}}** — {{ERKLAERUNG}}
2. **{{HEBEL_2}}** — {{ERKLAERUNG}}
3. **{{HEBEL_3}}** — {{ERKLAERUNG}}

## Pareto-3 für CK in den nächsten Tagen

1. {{AKTION_1}} ({{ETA}})
2. {{AKTION_2}} ({{ETA}})
3. {{AKTION_3}} ({{ETA}})

## Risiken

- {{RISIKO_1}} — Mitigation: {{MITIGATION_1}}
- {{RISIKO_2}} — Mitigation: {{MITIGATION_2}}

## Quantifizierte Vorteile

- Kosten-Einsparung: {{X} k€ Erst + {{Y}} k€/Jahr
- Time-to-Market: {{Z}} Monate früher
- {{ANDERES}}
```

---

## §12 Open-Items-Tracker (XLSX-Struktur)

**Datei:** `{{PROJEKT_NAME}}-Open-Items-Tracker.xlsx` (oder `.md` mit Tabellen)

**4 Sheets:**

### Sheet 1: Items (Master-Liste)

| ID | Titel | Kategorie | Owner | Priorität | Status | ETA | Notizen |
|---|---|---|---|---|---|---|---|
| OI-001 | {{TITEL}} | {{CODE/DOC/AUTH/INFRA/REGULATORIK}} | {{OWNER}} | {{P0/P1/P2/P3}} | {{OPEN/IN_PROG/BLOCKED/DONE}} | {{DATUM}} | {{...}} |

### Sheet 2: Pivot (Auto-aggregiert aus Sheet 1)

| Kategorie | Open | In-Prog | Blocked | Done | Total |
|---|---|---|---|---|---|
| CODE | 12 | 3 | 1 | 28 | 44 |
| DOC | 5 | 1 | 0 | 8 | 14 |
| ... | ... | ... | ... | ... | ... |

### Sheet 3: CK-Pareto-3 (manuell kuratiert)

| Rang | ID | Titel | Warum jetzt? | Action |
|---|---|---|---|---|
| 1 | OI-007 | BfArM-Sprechstunde buchen | Cleared Class-I-Hypothesis | Mail aus Template § 9 senden |
| 2 | OI-012 | Anwalts-Termin | Vertragsbestätigung nötig | Mail aus Template § 10 |
| 3 | OI-019 | DNS-Cutover Backend | Alter Server hat broken TLS | flyctl ips für neue DNS-Records |

### Sheet 4: ENG-Top-Items (Eng-Owner-View)

| Rang | ID | Titel | Blocker | ETA |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

---

## §13 `FASTEST_PATH.md` (Decision-Router, optional)

Bei mehreren parallelen Optionen ein 1-Seiten-Decision-Router.

```markdown
# FASTEST_PATH — {{ENTSCHEIDUNG_TITEL}}

## Ausgangslage

{{1_2_SAETZE}}

## Optionen

### Option A: {{NAME_A}}
- Pro: {{PRO_A}}
- Contra: {{CONTRA_A}}
- ETA: {{ETA_A}}
- Kosten: {{KOSTEN_A}}

### Option B: {{NAME_B}}
- Pro: {{PRO_B}}
- Contra: {{CONTRA_B}}
- ETA: {{ETA_B}}
- Kosten: {{KOSTEN_B}}

### Option C: {{NAME_C}}
- Pro: {{PRO_C}}
- Contra: {{CONTRA_C}}
- ETA: {{ETA_C}}
- Kosten: {{KOSTEN_C}}

## Empfehlung

{{OPTION_X}}, weil: {{BEGRUENDUNG_1_2_SAETZE}}

## Was muss CK entscheiden?

- [ ] Wahl A/B/C
- [ ] Budget-Freigabe für {{KOSTEN}}
- [ ] {{WEITERE_ENTSCHEIDUNG}}

Bitte mit `go A` / `go B` / `go C` antworten.
```
