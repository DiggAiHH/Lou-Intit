# 02 — METHODIK & WORKFLOW

Wie CK + Agent in jedem Projekt zusammenarbeiten. Pattern, das in DiggAi seit ~6 Wochen trägt und in Jobetes übernommen wurde.

---

## §1 Feature-Lebenszyklus (ASCII-Diagramm)

```
+-------------------+
|  CK-Idee / Bug   |
|  (in 1-3 Sätzen) |
+---------+--------+
          |
          v
+--------------------+        +-----------------------------+
| /prd Skill (Ralph) |  --->  | tasks/prd-<slug>.md         |
| Klärfragen 2-5×    |        | (5-15 User-Stories mit AC)  |
+--------------------+        +-----------------------------+
                                       |
                                       v
                             +--------------------+
                             | /ralph Skill       |
                             | → prd.json         |
                             +---------+----------+
                                       |
                                       v
+----------------------------------------------------------+
| Ralph Loop (fresh context jede Iteration)                |
|                                                          |
|  pick highest-priority story, passes:false               |
|         |                                                |
|         v                                                |
|  implement single story                                  |
|         |                                                |
|         v                                                |
|  quality check (typecheck, test, build)                  |
|         |                                                |
|         v                                                |
|  if green: commit + update prd.json passes:true          |
|            + append progress.txt                          |
|         |                                                |
|         v                                                |
|  next iteration (or COMPLETE if all stories pass)        |
+----------------------------------------------------------+
                                       |
                                       v
                       +------------------------------+
                       | memory/runs/YYYY-MM-DD_<...>.md |
                       | (5 Bullets: Aktion/Blocker/   |
                       |  Fix/Ergebnis/Out)            |
                       +------------------------------+
                                       |
                                       v
                       +------------------------------+
                       | {{PROJEKT_NAME}}_MASTER.md  |
                       | "Was tust du jetzt?"-Tabelle |
                       | (CK Daily-Driver)            |
                       +------------------------------+
```

---

## §2 PRD-Pattern

### Richtig dimensionierte Story

Jede Story ist in **einem Context-Window** abschließbar (≤30 min Coding). Pattern: 1 sichtbare Änderung, 1 Datei + 0–3 begleitende Edits.

**Beispiele (richtig):**

- Add a database column `patient.consent_version` + Prisma migration
- Add `<ConsentBanner>`-Komponente unter `/pwa/login`
- Add filter dropdown for date-range in `<AppointmentList>`
- Update `POST /api/patients` to validate phone via zod
- Add E2E-Test für Login-Flow (1 Happy-Path + 1 Fehler)

### Zu groß (splitten!)

**Beispiele (zu groß):**

- ❌ "Build the entire patient dashboard"
- ❌ "Add authentication"
- ❌ "Refactor the API"
- ❌ "Internationalize the app"

Jede dieser müsste in 5–15 kleinere Stories zerlegt werden.

### PRD-Skelett

Siehe `05_REPORTS_TEMPLATES.md` § 5 für das vollständige Template. Kurzform:

```markdown
# PRD: {{Feature-Name}}

## Why
{{1 Satz Motivation}}

## Acceptance Criteria
- {{AC1: messbar, testbar}}
- {{AC2: messbar, testbar}}

## User Stories
1. As {{role}}, I want {{thing}}, so that {{benefit}}.
2. ...

## Branch
feat/{{slug}}

## Out of Scope
- {{explizit nicht}}
```

---

## §3 Run-Log-Pattern

**Pfad:** `memory/runs/<YYYY-MM-DD>_<agent>_<model>-<NN>.md`

**Naming:**
- `<agent>` = lowercase Kurzname. Etabliert: `claude-code`, `copilot`, `codex`, `kimi`, `gemini`, `cursor`, `opus`.
- `<model>` = Kurztag. Beispiele: `opus-4-7`, `sonnet-4-6`, `gpt-5`, `gemini-2-5`, `kimi-k2`.
- `<NN>` = monoton wachsender Counter pro (agent, model, Tag). `01`, `02`, …

**Format:**

```markdown
YYYY-MM-DDTHH:MM+02:00 | Lauf <agent>-<NN> | <one-line topic>
---
- Aktion: <was du konkret gemacht hast>
- Blocker: <was hat dich aufgehalten, oder "—">
- Fix: <wie du es gelöst hast, oder "—">
- Ergebnis: <observable outcome — commit hash, PR #, file path>
- Out: <verified state — "tests green", "PR #N open", "blocked on F1-F8", …>
```

**Warum non-negotiable:** Agenten verlieren Context zwischen Sessions. Run-Log persistiert. Nächster Agent liest die letzten 3 Einträge und weiß, was probiert wurde, was funktioniert hat, was offen ist.

**DoD-Regel:** Ein Prompt mit observable outcome (Commit, File-Change, PR, Deployment, Entscheidung) ist NICHT done bis das Run-Log eingetragen ist.

---

## §4 Pareto-3 für CK

Bei Status-Reviews die **3 wichtigsten offenen Items** für CK rausarbeiten. Pattern:

```markdown
## CK-Pareto-3 (Tag {{DATUM}})

1. **{{Item 1}}** — {{warum jetzt, was ist die nächste Aktion, ETA}}
2. **{{Item 2}}** — {{...}}
3. **{{Item 3}}** — {{...}}

(Der Rest kann warten — siehe Open-Items-Tracker für vollständige Liste.)
```

Kein 7-Punkte-Plan. CK liest am Morgen ein 3er-Plan, der Rest hängt in der Liste.

---

## §5 Definition of Done (11 Punkte)

Eine Story / ein Prompt ist DONE wenn:

1. ✅ **Code geschrieben** und in Repo.
2. ✅ **Typecheck grün** (`npm run typecheck` oder `tsc --noEmit`).
3. ✅ **Tests grün** (`npm test`) — falls für die Story relevant.
4. ✅ **Lint grün** (`npm run lint`) — keine neuen Warnings.
5. ✅ **Build grün** (`npm run build`) — keine Compile-Fehler.
6. ✅ **Manuelles Smoke-Test** durchgeführt (Browser / Curl / Postman je nach Story).
7. ✅ **Commit** mit aussagekräftiger Message (Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`).
8. ✅ **Push** auf richtigen Remote (nicht `_bootstrap`-Origin!).
9. ✅ **prd.json updated** — `passes: true` für die abgeschlossene Story.
10. ✅ **Run-Log-Eintrag** in `memory/runs/`.
11. ✅ **Master-Cockpit** "Was tust du jetzt?"-Tabelle nachgezogen falls Item-Status sich ändert.

Wenn Punkt 2–6 rot ist: nicht commiten, sondern in `VERSTAENDNIS_LUECKEN.md` Frage eintragen und stoppen.

---

## §6 Audit-Pattern (vor Risiko-Aktionen)

Vor jeder dieser Aktionen explizit STOP + CK-Bestätigung:

- Force-Push auf irgendeinen Branch
- DB-Migration auf Prod-DB
- DNS-Cutover (Domain wechselt Backend)
- Externe Mail mit Anhang an Behörde / Kanzlei / Investor
- Vertragsbindender Schritt (Stripe-Aktivierung, Hosting-Plan-Upgrade)
- Löschung von ≥10 Files oder ≥1 Datenbank-Tabelle

Audit-Output-Format (an CK):

```markdown
## Audit vor {{Aktion}}

**Was passiert:** {{1 Satz}}
**Reversible?** {{Ja/Nein, wenn ja: wie zurück}}
**Risk-Score (1-5):** {{N}} — {{Begründung}}
**Pre-Conditions checked:**
- ✅ {{Check 1}}
- ✅ {{Check 2}}

**Bitte bestätigen mit "go" oder "stop".**
```

CK sagt "go" → ausführen. Alles andere = stop.

---

## §7 Verständnis-Lücken (`VERSTAENDNIS_LUECKEN.md`)

Wenn der Agent eine Frage hat, die er selbst nicht beantworten kann (Architektur-Entscheidung, Domain-Wissen, externe Constraint), trägt er sie als `Q<N>:` ein. Pattern:

```markdown
## Q12 (2026-05-19): Soll {{Feature}} synchron oder async sein?

**Kontext:** {{wann/wo entstanden}}
**Optionen:**
- A) Synchron — {{Pro/Contra}}
- B) Async via Queue — {{Pro/Contra}}
**Empfehlung:** A (für jetzt, B als Migration in Phase 3)
**Status:** OPEN | RESOLVED ({{Datum, Entscheidung, Begründung}})
```

CK schließt offene `Q<N>` mit Status `RESOLVED` und einer 1-Satz-Begründung. Resolved-Einträge bleiben drin — sind Architektur-Historie.

---

## §8 Browser-Tier-Regeln (Cowork-spezifisch)

In Cowork-Mode hat Chrome Tier `read` (Screenshots ja, Klicks/Eingabe nein). Daraus folgt:

| Aktion | Werkzeug |
|---|---|
| Web-App lesen (Hetzner-Konsole, GitHub-PR, Fly-Dashboard) | `mcp__computer-use__screenshot` |
| In Web-App klicken / tippen | `mcp__claude-in-chrome__*` (DOM-aware) |
| Web-Form ausfüllen | claude-in-chrome `form_input` |
| Native App (Excel, Mail, System-Settings) | `mcp__computer-use__*` (tier full) |
| Terminal / VS Code Editor-Tippen | **nicht möglich** in Cowork-tier-click — `mcp__workspace__bash` benutzen |

**Folge für CK-Aktionen:** Wenn der Agent CK bittet, in einer Web-Konsole zu klicken (Hetzner-Rescue, Stripe-Aktivierung, GitHub-Settings), läuft das ohne Hilfe des Agents. Schritt 1 = präzise klick-Anleitung mit Screenshot + URL, dann CK macht es selbst.

---

## §9 Codespace-Fallback (bei lokalem OOM)

CK-Maschine hat im Oktober/November 2025 mehrfach OOM beim `npm ci` mit aktivierten Workspaces gehabt. Fallback:

1. `.devcontainer/devcontainer.json` mit `node:20` ins Repo (siehe `05_REPORTS_TEMPLATES.md` § 8).
2. Branch erstellen (`feat/big-refactor`), pushen.
3. GitHub → Code → Codespaces → "Create Codespace on `feat/big-refactor`".
4. Machine-Type: **4-core / 16 GB RAM** (Personal Free-Tier reicht 60 h/Monat).
5. Secrets im Codespace-Settings: `DATABASE_URL`, `JWT_SECRET`, `ENCRYPTION_KEY`, ggf. `FLY_API_TOKEN`, `NETLIFY_AUTH_TOKEN`.
6. Im Terminal: `npm ci --legacy-peer-deps --ignore-scripts && npm run build`.
7. Wenn grün → commit + push, lokal weitermachen.

`NODE_OPTIONS=--max-old-space-size=4096` ist in `netlify.toml` und im Default-`package.json` schon drin.

---

## §10 Caveman-Modi (wann was)

| Modus | Wann |
|---|---|
| `/caveman lite` | Default bei normalen Codingsessions — drop filler, 30-40% Tokens gespart |
| `/caveman full` | CK Status-Reviews — kompaktes Status-Liste, 65% Tokens gespart |
| `/caveman ultra` | Telegraphic, nur bei sehr fokussierten "Fix-One-Thing"-Sessions |
| `/caveman wenyan` | Nie benutzen für DiggAi/Jobetes-Kontext — klassisches Chinesisch passt nicht |
| `normal mode` | Anwalts-/Behörden-Mails schreiben, Patient-facing Texte, CER-Dokumentation |

**Caveman-compress:** Memory-Files (`CLAUDE.md`, `AGENTS.md`, `progress.txt`) periodisch komprimieren — Pattern aus DiggAi: `/caveman-compress CLAUDE.md` einmal pro Woche, spart ~46% Input-Tokens jede Session.

---

## §11 "Was passiert wenn CK 'weiter' sagt"

Default-Verhalten bei knappen CK-Inputs:

| Input | Agent-Aktion |
|---|---|
| `weiter` / `mach das` / `ok` | Nächstes PRD-Item ziehen, autonom durchziehen, knapp berichten |
| `harness` | Aktuelle Quality-Checks durchlaufen, Status melden, falls grün next-item |
| `audit` | Risk-Audit für aktuelles Item (siehe § 6) |
| `plan` | Pareto-3 + offene Items aus `tasks/` + nächste 3 Schritte |
| `stop` | Stoppen, in `VERSTAENDNIS_LUECKEN.md` aktuellen Stand eintragen |
| `recovery` | Letzten 3 Run-Log lesen, Stand rekonstruieren, was-jetzt fragen |
| `[nichts] / Bestätigung` | "ok" = positives Quittieren, weiter mit nächstem Item |

---

## §12 Was CK selbst tun muss

Der Agent kann viel autonom — aber **diese Aktionen** macht immer CK:

1. **Mail-Senden** — Agent erstellt Draft (Gmail-MCP oder Markdown-Datei zum Copy-Paste), CK klickt "Senden".
2. **Hetzner-Klicks** (Rescue, Server-Reboot, IP-Routing) — Chrome ist read-only in Cowork.
3. **Stripe-Aktivierung** (Vertrag-Bindung) — kein Auto-Move-Money.
4. **DNS-Cutover** (Provider-UI) — manuell, mit `dig`-Verifikation hinterher.
5. **Anwalts-/Steuer-Entscheidungen** — Agent recherchiert, CK entscheidet.
6. **Force-Push auf shared Branches** — Agent fragt, CK gibt go.
7. **DSGVO-Lösch-Anfragen** — Agent erstellt Lösch-Skript, CK führt aus.
8. **Hosting-Plan-Upgrade / CC hinterlegen** (z. B. Fly-Trial-Expired) — Agent zeigt URL, CK macht.

---

## §13 Crash-Recovery-Pattern

Wenn eine Session crasht (Cowork stürzt ab, Context overflow, Model-Switch):

1. CK öffnet frischen Tab mit Opus 4.7.
2. Erste Message: `recovery — lies die letzten 3 run-logs und sag mir wo wir stehen`.
3. Agent liest `memory/runs/` (sortiert by filename desc), `progress.txt`, `prd.json`, `{{PROJEKT_NAME}}_MASTER.md`.
4. Agent gibt 5-Zeilen-Status: was gerade lief, was als letztes commit'd wurde, was offen ist, was die nächste Aktion ist, ob CK eine Entscheidung schuldet.
5. CK sagt "weiter" oder gibt neue Richtung.

Wenn die letzten Run-Logs widersprüchlich sind, in `VERSTAENDNIS_LUECKEN.md` schauen — letzte offene `Q<N>:` ist meist der Bruch.

## § 14 Fork-Workflow als Notfall-Push (ohne Collaborator-Access)

Wenn `git push origin <branch>` mit "Permission denied" antwortet und Collaborator-Status nicht in 5 Min besorgt werden kann:

1. Auf GitHub: Upstream-Repo öffnen → "Fork".
2. Lokal: `git remote add fork <fork-url>`.
3. `git push fork <branch>` (pro Branch).
4. Auf GitHub: PR aus Fork → Upstream öffnen.
5. Operator (mit Push-Access auf Upstream) mergt im Upstream-Repo.

Skript für mehrere Branches:
```bash
for b in feat/a feat/b feat/c; do git push fork $b; done
gh pr create --repo upstream/repo --head fork:<branch>
```

**Vorteil:** keine Wartezeit auf Collaborator-Status, kein Verlust von Commits.
**Nachteil:** PRs gehen aus Fork-Branch, Reviewer-UI ist etwas anders.

*Quelle: JoBetes-Sync 2026-05-19 (M05 in Zentrale-DiggAi-Bank).*
