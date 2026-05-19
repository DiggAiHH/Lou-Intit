# Lou-Intit — Project Bootstrap Kit (Master Index)

Ein wiederverwendbarer Bootstrap-Kit für Laith Alshdaifat (`DiggAi@tutanota.de`). Pro neuem Projekt einmal clonen, 4 Platzhalter füllen, in Opus 4.7 pasten — Phase 0 läuft autonom.

**Stand:** 2026-05-19 · **Owner:** Laith Alshdaifat · **Default-Stack:** React 19 + Vite + Express 5 + Prisma 6 + Postgres 16 / Fly.io + Netlify + Neon · **Methodik:** Ralph + Caveman + Run-Log-Disziplin.

---

## §1 Datei-Map

| Pfad | Zweck | Wann lesen |
|---|---|---|
| [`README.md`](README.md) | Repo-Front-Page (Copilot-generiert) | Beim ersten Anschauen des Repos |
| [`CLONE_AND_USE.md`](CLONE_AND_USE.md) | 4-Schritte-Onboarding | Bei jedem neuen Projekt |
| [`INDEX.md`](INDEX.md) | **Diese Datei** — Navigation + Reading-Order | Wenn du verloren bist |
| [`CROSS_PROJECT_PROMPT.md`](CROSS_PROJECT_PROMPT.md) | Paste-Ready-Prompt für JEDES neue Projekt | Bei neuem PC / Cowork-Account |
| [`LICENSE`](LICENSE) | MIT | — |
| [`.gitignore`](.gitignore) | Node + IDE + OS Ignores | Wird beim Clonen mitkopiert |
| [`_bootstrap/README.md`](_bootstrap/README.md) | Kit-eigener Index + 3 Nutzungs-Varianten | Vor Phase 0 |
| [`_bootstrap/00_SYSTEM_PROMPT_BOOTSTRAP.md`](_bootstrap/00_SYSTEM_PROMPT_BOOTSTRAP.md) | **Der Prompt** zum Pasten in Opus 4.7 | Schritt 3 von Onboarding |
| [`_bootstrap/01_TECH_STACK.md`](_bootstrap/01_TECH_STACK.md) | Default-Stack + Was-NICHT + Stack-Tabelle | Phase 0.1 (Skelett) |
| [`_bootstrap/02_METHODIK_WORKFLOW.md`](_bootstrap/02_METHODIK_WORKFLOW.md) | PRD/Loop/Run-Log/DoD/Pareto-3 | Phase 0.2 + jeder Loop |
| [`_bootstrap/03_TOOLS_REPOS.md`](_bootstrap/03_TOOLS_REPOS.md) | Ralph + Caveman + MCPs + Hosting-CLIs | Phase 0.3 (Install) |
| [`_bootstrap/04_MEMORY_LEARNINGS.md`](_bootstrap/04_MEMORY_LEARNINGS.md) | 14 Gotchas + Was-funktioniert + Owner-Style | Immer wenn was schiefgeht |
| [`_bootstrap/05_REPORTS_TEMPLATES.md`](_bootstrap/05_REPORTS_TEMPLATES.md) | 13 Skelette: CLAUDE.md, AGENTS.md, Run-Log, PRD, Mails | Wenn du was generieren musst |
| [`_bootstrap/06_PLATZHALTER_AUSFUELLEN.md`](_bootstrap/06_PLATZHALTER_AUSFUELLEN.md) | Was du füllen musst, was Opus selbst fragt | Vor Cowork-Tab-Öffnung |

---

## §2 Reading-Order für neuen Agent (Opus 4.7 in neuem Projekt)

1. **README.md** (Repo-Root, dieser Repo) — 30 Sekunden Überblick
2. **CLONE_AND_USE.md** — wie der Kit benutzt wird
3. **`_bootstrap/00_SYSTEM_PROMPT_BOOTSTRAP.md`** — der eigentliche Prompt (Hauptaktion: Block zwischen den Markern in Opus pasten)
4. Opus liest selbst: 01 → 02 → 03 → 04 → 05 (Pflicht-Lektüre vor Phase 0)
5. **`_bootstrap/06_PLATZHALTER_AUSFUELLEN.md`** — falls Klärfragen kommen

---

## §3 Workflow-Diagramm (ASCII)

```
+----------------------------------+
|  Neues Projekt starten           |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
| git clone Lou-Intit _bootstrap   |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
| 4 Platzhalter in 00_SYSTEM_*.md  |
| füllen (PROJEKT_NAME, KURZBE-    |
| SCHREIBUNG, ZIELNUTZER, REMOTE)  |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
| Cowork-Tab Opus 4.7              |
| Prompt-Block zwischen            |
| ─── START ─── und ─── ENDE ───   |
| als erste Nachricht pasten       |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
| Opus liest 5 Anhänge (01-05)     |
| automatisch                      |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
| Phase 0 in 7 Sub-Phasen          |
| 0.1 Skelett → 0.2 Memory →       |
| 0.3 Ralph+Caveman → 0.4 PRD →    |
| 0.5 Run-Log → 0.6 Master →       |
| 0.7 Commit & Push                |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
| Loop-Modus:                      |
| Ralph zieht PRD-Stories, baut    |
| jede in fresh Context, commit,   |
| Run-Log, Master-Cockpit-Update   |
+----------------------------------+
```

---

## §4 Versionierung

Der Kit ist ein **lebendiges Artefakt**. Wenn du in einem Projekt einen neuen Gotcha entdeckst oder einen besseren Workflow findest:

1. In deinem Projekt-`_bootstrap/` Änderung machen
2. Im Lou-Intit-Repo dieselbe Änderung committen (PR oder direkt)
3. Beim nächsten Projekt steht es zur Verfügung

**Was sich ändert sollte:**
- `04_MEMORY_LEARNINGS.md` — neue Gotchas, neue Was-funktioniert-Patterns
- `05_REPORTS_TEMPLATES.md` — neue Skelette wenn du wiederholt was schreibst
- `03_TOOLS_REPOS.md` — neue MCP-Connectors, Tool-Updates

**Was selten ändert:**
- `00_SYSTEM_PROMPT_BOOTSTRAP.md` — der Prompt-Kern (stable)
- `01_TECH_STACK.md` — Stack-Defaults (selten Stack-Wechsel)
- `02_METHODIK_WORKFLOW.md` — Methodik ist etabliert

---

## §5 Bekannte Issues

- **`05_REPORTS_TEMPLATES.md` §1 (CLAUDE.md-Template):** Innere Code-Fence-Backticks haben Backslash-Artefakte. Inhalt ist korrekt und lesbar, nur die Code-Fence-Rendering ist suboptimal. Fix beim nächsten Kit-Update.

---

## §6 Cross-Project-Pattern (für Multi-Project-Setup)

Laith arbeitet an mehreren Projekten parallel. Jedes Projekt clont diesen Kit. Wenn ein Projekt einen Lessons-Learned generiert, der für die anderen relevant ist:

1. In Lou-Intit als PR oder direkt-commit hinzufügen
2. In den anderen Projekten: `cd _bootstrap && git pull origin main` (falls _bootstrap als Submodule oder separate Clone)

Empfehlung: `_bootstrap/` als **Git-Submodule** statt Subdirectory — dann ist `git pull --recurse-submodules` der Sync-Befehl.

```bash
# Beim neuen Projekt
git submodule add https://github.com/DiggAiHH/Lou-Intit _bootstrap
git commit -m "chore: add Lou-Intit as _bootstrap submodule"

# Bei Update
cd _bootstrap && git pull origin main && cd ..
git add _bootstrap && git commit -m "chore: update _bootstrap to latest"
```

---

## §7 Quick-Reference

| Aktion | Datei / Pfad |
|---|---|
| Neues Projekt starten | `CLONE_AND_USE.md` Schritt-für-Schritt |
| Stack-Frage klären | `_bootstrap/01_TECH_STACK.md` §10 (Stack-Tabelle) |
| Run-Log-Format vergessen | `_bootstrap/02_METHODIK_WORKFLOW.md` §3 |
| DoD-Liste prüfen | `_bootstrap/02_METHODIK_WORKFLOW.md` §5 (11 Punkte) |
| Gotcha nachschlagen | `_bootstrap/04_MEMORY_LEARNINGS.md` §1 (G01-G14) |
| Template kopieren (CLAUDE.md, PRD, Mails) | `_bootstrap/05_REPORTS_TEMPLATES.md` |
| Platzhalter-Liste durchgehen | `_bootstrap/06_PLATZHALTER_AUSFUELLEN.md` |

---

## §8 Kontakt + Sicherheit

- **Owner:** Laith Alshdaifat · `DiggAi@tutanota.de`
- **Repo-Sichtbarkeit:** Public — keine echten Secrets, Anschriften oder Tokens commiten
- **Lizenz:** MIT (LICENSE-Datei)
- **GitHub:** `https://github.com/DiggAiHH/Lou-Intit`

Bei Public-Repos vor Commit: `grep -rn "{{" .` zur Kontrolle, dass keine ungefüllten Platzhalter im Code landen.
