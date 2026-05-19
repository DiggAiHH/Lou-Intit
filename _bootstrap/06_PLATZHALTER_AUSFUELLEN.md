# 06 — PLATZHALTER AUSFÜLLEN

Was du vor dem Bootstrap-Start einmal füllst — und was Opus zwischendurch selbst nachfragt.

---

## §1 Pflicht-Tabelle (4 Felder Minimum)

Diese **4 Felder** musst du füllen, sonst kann Opus nicht starten. Trage sie in `00_SYSTEM_PROMPT_BOOTSTRAP.md` § 1 ein.

| Platzhalter | Beispiel | Wo zu füllen |
|---|---|---|
| `{{PROJEKT_NAME}}` | `diggai-anamnese` | 00_SYSTEM_PROMPT § 1 |
| `{{KURZBESCHREIBUNG_2_SAETZE}}` | "DSGVO-konforme Patienten-Anamnese-Plattform für deutsche Arztpraxen. React + Express + Prisma." | 00_SYSTEM_PROMPT § 1 |
| `{{ZIELNUTZER}}` | "Ärzte in Allgemeinmedizin-Praxen, 1-5 Behandler pro Praxis" | 00_SYSTEM_PROMPT § 1 |
| `{{REMOTE_NAME}}` | `DiggAiHH` | 00_SYSTEM_PROMPT § 1 + README.md |

Das reicht für Variante A (Schnell-Bootstrap). Opus fragt den Rest.

---

## §2 Empfohlen-Tabelle (vor Phase 0.3 nützlich)

Wenn du diese **10 Felder** auch füllst, läuft Phase 0 ohne weitere Klärfragen durch.

| Platzhalter | Beispiel | Wo zu füllen |
|---|---|---|
| `{{GRUEN_WIESE\|LEGACY_REWRITE\|PROTOTYPE_TO_PROD}}` | `GRUEN_WIESE` | 00_SYSTEM_PROMPT § 1 |
| `{{ZEITHORIZONT}}` | "MVP bis 8 Wochen, Prod-Launch 16 Wochen" | 00_SYSTEM_PROMPT § 1 |
| `{{KEINE\|MEDICAL_DEVICE\|FINTECH\|DSGVO_ART_9\|ANDERE}}` | `MEDICAL_DEVICE` | 00_SYSTEM_PROMPT § 1 |
| `{{HOSTING_DEFAULT_FLY_NETLIFY_NEON\|HETZNER_DOCKER\|ANDERE}}` | `HOSTING_DEFAULT_FLY_NETLIFY_NEON` | 00_SYSTEM_PROMPT § 1 |
| `{{FRONTEND_DOMAIN}}` | `diggai.de` | Wird in 0.1 gebraucht (`fly.toml`, `netlify.toml`) |
| `{{BACKEND_DOMAIN}}` | `api.diggai.de` oder `diggai-api.fly.dev` | Wie oben |
| `{{NEON_PROJECT_ID}}` | `wandering-truth-12345` | Aus Neon-Console kopieren |
| `{{FLY_APP_NAME}}` | `diggai-api` | Beim `flyctl launch` festgelegt |
| `{{NETLIFY_SITE_NAME}}` | `diggai-anamnese` | Beim `netlify init` festgelegt |
| `{{STACK_KURZ}}` | "React 19 + Vite + Express 5 + Prisma 6 + Postgres 16" | CLAUDE.md § Projekt-Identität |

---

## §3 Optional-Tabelle (Opus fragt zwischendurch wenn nötig)

Diese Felder fragt Opus selbst mit `AskUserQuestion` zur richtigen Zeit. Du kannst sie auch vorab füllen.

| Platzhalter | Wofür | Wann gefragt |
|---|---|---|
| `{{VOLL_NAME}}` | Mails an Behörden / Anwalt | Bei erster Mail-Erstellung |
| `{{TELEFONNUMMER}}` | Mails | Wie oben |
| `{{ANSCHRIFT_VOLL}}` | BfArM-Antrag | Bei BfArM-Template |
| `{{RECHTSFORM}}` | Einzelunternehmer / GmbH / UG | BfArM + Anwalts-Mail |
| `{{EMAIL}}` | Korrespondenz-Email | Bei externer Mail (Default: DiggAIPrakt@gmail.com) |
| `{{TITEL}}` | "Dr. med." / "M.Sc." | Mails |
| `{{ORGANISATION}}` | Praxis-/Firmen-Name | Mails |
| `{{INTENDED_PURPOSE_3_5_SAETZE}}` | BfArM-Klassifizierung | BfArM-Template |
| `{{KLASSE_HYPOTHESE}}` | "Class I nach Regel 11" | BfArM + Anwalt |
| `{{KANZLEI_EMAIL}}` | Anwalts-Erstkontakt | Bei Anwalts-Pool-Mail |
| `{{HINTERGRUND_2_SAETZE}}` | Anwalts-Erstkontakt | Wie oben |
| `{{PRODUKT_NAME}}` | Für externe Kommunikation | BfArM + Anwalt + Pitch-Decks |
| `{{MULTI_TENANT_STRATEGIE}}` | Subdomain / Path / Single-Tenant | Bei Auth-PRD |
| `{{PRICING_STRATEGIE}}` | B2B / B2C / Internal | Bei Stripe-Setup |

---

## §4 Minimum-Set zum schnellen Start (4 Felder)

Wenn du nur 4 Felder füllst und sofort starten willst:

```
{{PROJEKT_NAME}}              → z.B. mein-neues-projekt
{{KURZBESCHREIBUNG_2_SAETZE}} → z.B. Tool für X, das Y macht. Stack: Z.
{{ZIELNUTZER}}                → z.B. Ärzte in DE-Praxen
{{REMOTE_NAME}}               → z.B. DiggAiHH
```

Opus startet damit und fragt den Rest mit `AskUserQuestion` — bis zu 4 Fragen pro Karte, max 2 Karten bevor er anfängt zu bauen.

---

## §5 Konsistenz-Checkliste (vor Cowork-Tab-Öffnung)

Schnelldurchlauf, ~30 Sekunden:

- [ ] Ordner `PROJECT_BOOTSTRAP_KIT/` im neuen Repo kopiert (als `_bootstrap/` empfohlen)
- [ ] Alle 7 Dateien drin: `README.md`, `00_SYSTEM_PROMPT_BOOTSTRAP.md`, `01_TECH_STACK.md`, `02_METHODIK_WORKFLOW.md`, `03_TOOLS_REPOS.md`, `04_MEMORY_LEARNINGS.md`, `05_REPORTS_TEMPLATES.md`, `06_PLATZHALTER_AUSFUELLEN.md`
- [ ] Mindestens die 4 Pflicht-Platzhalter aus § 1 oben in `00_SYSTEM_PROMPT_BOOTSTRAP.md` § 1 ersetzt
- [ ] Ralph-URL im 03er-File noch aktuell: `https://github.com/snarktank/ralph` ✅
- [ ] Caveman-URL im 03er-File noch aktuell: `https://github.com/JuliusBrussee/caveman` ✅
- [ ] CK-Email-Default (`DiggAIPrakt@gmail.com`) prüfen — anders falls anderes Projekt-Email
- [ ] Repo lokal initialisiert (`git init`) ODER bestehendes Repo ist gecloned
- [ ] Neuer Cowork-Tab mit **Opus 4.7** als Modell (NICHT Sonnet — der ist günstiger, aber für Bootstrap-Phase ungeeignet)

---

## §6 Wenn Opus zwischendurch fragt

Antworte knapp:

- "Welcher Stack?" → "Default aus 01_TECH_STACK.md übernehmen" oder spezifische Abweichungen nennen
- "Wie heißt das Backend?" → `{{PROJEKT_NAME}}-api` als Default
- "Wie groß soll das erste PRD sein?" → "5-7 User-Stories für MVP-Login + Patient-Anlegen" (oder Projekt-spezifisch)
- "Welche Tests?" → "Vitest für Unit, Playwright für E2E" (Default)
- "Welche Auth?" → "JWT + Argon2 selbst implementiert" (Default) oder "Auth.js falls Social-Login"
- "Stripe schon dazu?" → "Nein, erst nach MVP-Validation" (Default)
- "Multi-Tenant?" → Konkret antworten: "Ja, Subdomain" / "Ja, per User" / "Nein, Single-Tenant"
- "Soll ich DSGVO-Texte schon generieren?" → "Nein, erst nach Anwalts-Pass" (Default für regulierte Projekte)

Wenn Opus eine Frage stellt, die du selbst nicht beantworten kannst: "Trag's in `VERSTAENDNIS_LUECKEN.md` ein und mach mit dem Rest weiter" — dann hängt es nicht.

---

## §7 Anti-Patterns (nicht so füllen!)

| Falsch | Richtig |
|---|---|
| `{{PROJEKT_NAME}} = "Mein Projekt"` (mit Leerzeichen + Anführungszeichen) | `mein-projekt` (kebab-case, kein Quote) |
| `{{REMOTE_NAME}} = github.com/foo` | `foo` (nur User/Org, ohne URL-Prefix) |
| `{{KURZBESCHREIBUNG}} = "Es geht um eine App, die ..."` (1 Satz fließend) | "Tool für X. Stack: Y." (2 telegraphische Sätze, sind besser für Agent-Parsing) |
| `{{TELEFONNUMMER}} = leer lassen mit Platzhalter im Mail-Draft` | Konkret füllen ODER mit `[Telefonnummer hier]`-Marker, den du beim Senden ersetzt |
| Mehrere Platzhalter mit komma-separierten Werten (`{{X}} = "a, b, c"`) | Pro Wert separater Platzhalter, oder erst Wahl-Tabelle in `01_TECH_STACK.md` § 10 ausfüllen |

---

## §8 Nach erfolgreichem Bootstrap

Nach Phase 0.7 (Commit & Push) hat dein Repo:

- ✅ Komplettes Skelett (alle Verzeichnisse, `.gitignore`, `LICENSE`)
- ✅ `CLAUDE.md`, `AGENTS.md`, `progress.txt`, `VERSTAENDNIS_LUECKEN.md`
- ✅ `{{PROJEKT_NAME}}_MASTER.md` mit erstem CK-Pareto-3
- ✅ Ralph + Caveman installiert
- ✅ Erstes PRD in `tasks/prd-<slug>.md` + `prd.json`
- ✅ Erstes Run-Log in `memory/runs/`
- ✅ Commit gepusht auf richtiges Remote

**Nächster Schritt:** `./scripts/ralph/ralph.sh --tool claude 5` (5 Iterations für erste PRD-Stories). Wenn alle grün → MVP weiter ausbauen, neue PRDs schreiben.

Wenn Phase 0 schief geht (ein Sub-Schritt rot, Tool nicht installiert, Auth-Fehler bei `gh` / `flyctl`): zurück in `_bootstrap/04_MEMORY_LEARNINGS.md` § 1 — die Gotchas sind chronologisch nach Häufigkeit sortiert, der häufige Fall ist oben.
