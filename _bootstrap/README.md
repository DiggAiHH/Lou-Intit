# PROJECT BOOTSTRAP KIT

Wiederverwendbare Vorlage, um in 30 Sekunden jedes neue Projekt auf demselben Harness, derselben Methodik und demselben Tool-Setup zu starten.

**Eigentümer:** CK (`DiggAIPrakt@gmail.com`)
**Stand:** 2026-05-19
**Quellen:** Synthese aus den letzten ~30 Cowork-Sessions (DiggAi, Jobetes, Hetzner, Co-Reviewer-Audits) + Raw-READMEs von Ralph (`snarktank/ralph`) und Caveman (`JuliusBrussee/caveman`).

---

## Was ist drin

| # | Datei | Zweck |
|---|---|---|
| — | `README.md` | Du bist hier. Index + 3 Nutzungs-Varianten |
| 00 | `00_SYSTEM_PROMPT_BOOTSTRAP.md` | Der Prompt zum Pasten in Opus 4.7 für neue Projekte |
| 01 | `01_TECH_STACK.md` | Default-Stack-Tabelle + "Was wir NICHT nutzen"-Liste |
| 02 | `02_METHODIK_WORKFLOW.md` | PRD → Loop → Run-Log → Master-Cockpit, DoD, Pareto-3 |
| 03 | `03_TOOLS_REPOS.md` | Ralph, Caveman, MCP-Connectors, Hosting-CLIs, Repo-Tree |
| 04 | `04_MEMORY_LEARNINGS.md` | Gotchas, was-funktioniert / was-nicht, CK-Präferenzen |
| 05 | `05_REPORTS_TEMPLATES.md` | 13 Skelette: CLAUDE.md, AGENTS.md, Run-Log, PRD, Master, Mails … |
| 06 | `06_PLATZHALTER_AUSFUELLEN.md` | Was du füllen musst, was Opus selbst fragt, Konsistenz-Check |

---

## Wie du den Kit benutzt

### Variante A — Schnell (5 Minuten, Standard-Fall)

1. Ordner `PROJECT_BOOTSTRAP_KIT/` ins neue Repo kopieren (Empfehlung: als `_bootstrap/`).
2. In `00_SYSTEM_PROMPT_BOOTSTRAP.md` die **4 Pflicht-Platzhalter** füllen (`{{PROJEKT_NAME}}`, `{{KURZBESCHREIBUNG_2_SAETZE}}`, `{{ZIELNUTZER}}`, `{{REMOTE_NAME}}` — Details in 06).
3. Frischen Cowork-Tab mit **Opus 4.7** öffnen, kompletten Block zwischen `─── PROMPT START ───` und `─── PROMPT ENDE ───` aus 00 pasten.
4. Opus liest die 5 Anhänge (01–05) selbst, fragt den Rest mit `AskUserQuestion`, baut Phase 0.

### Variante B — Gründlich (30 Minuten, regulierte / komplexe Projekte)

1. Kit wie in Variante A kopieren.
2. **Alle** Platzhalter in 06 systematisch durchgehen — Stack-Entscheidungen, Compliance-Level, Domain-Spezifika.
3. Optional: `04_MEMORY_LEARNINGS.md` §4 (Regulatorik) aktivieren, falls Medical-Device / DiGA / Pharma.
4. `05_REPORTS_TEMPLATES.md` §9–§11 (BfArM-Mail, Anwalts-Mail, Status-Plan-Flip) sichten — Kontaktdaten in 06 hinterlegen.
5. Cowork-Tab öffnen, Prompt pasten, Opus arbeitet mit voll ausgefüllter Vorlage.

### Variante C — Nachträgliche Migration (bestehendes Projekt aufrüsten)

1. Kit in vorhandenes Repo kopieren.
2. Aus `05_REPORTS_TEMPLATES.md` nur die Templates ziehen, die fehlen: `CLAUDE.md`, `AGENTS.md`, `memory/runs/`, `master-cockpit.md`, `progress.txt`, `VERSTAENDNIS_LUECKEN.md`.
3. **Nicht** den ganzen System-Prompt laufen lassen — der überschreibt zu viel. Stattdessen: 1 Run mit Opus, der nur die fehlenden Files anlegt und die DoD-Liste in `CLAUDE.md` ergänzt.
4. Ab dem nächsten Feature normalen Loop fahren.

---

## Versionierung

Der Kit ist ein lebendiges Artefakt. Wenn du in einem Projekt einen neuen Gotcha entdeckst, einen besseren Workflow findest oder ein Template anpasst — zurück in den Kit committen, beim übernächsten Projekt steht es zur Verfügung.

**Vorschlag:** Privates Repo `{{REMOTE_NAME}}/project-bootstrap-kit`, dann beim neuen Projekt nur:

```bash
git clone https://github.com/{{REMOTE_NAME}}/project-bootstrap-kit _bootstrap
```

---

## Sicherheits-Disclaimer

Im Kit sind **keine** echten Anschriften, Telefonnummern, API-Tokens, Pricing-Werte oder personen-/projektbezogenen Daten. Alles als `{{PLATZHALTER}}` markiert. Vor Commit eines befüllten Bootstraps in ein öffentliches Repo: Platzhalter-Scan mit `grep -r "{{" .` durchführen — Treffer sind unbefüllte Felder, keine Treffer = sicher gefüllt.
