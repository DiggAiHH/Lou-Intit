# 04 — MEMORY & LEARNINGS

Alle Gotchas, Patterns und CK-Präferenzen aus den letzten Projekten. Diese Datei in den `04_MEMORY`-Block von `CLAUDE.md` als Read-Only-Anhang einbinden — Ralph + Caveman + Opus haben es dann beim Bootstrap dabei.

---

## §1 Gotchas (häufigster zuerst)

### G01 — Windows-cmd, kein PowerShell, kein `&&`

CKs Maschine ist Windows 10/11. Default-Shell `cmd.exe`. Quirks:

- Kein `&&` für Verkettung — stattdessen `;` oder neue Zeile
- Kein `??` Null-Coalescing in Inline-Pfaden
- Backslash-Pfade (`D:\path\to\file`), Forward-Slash funktioniert in npm-scripts trotzdem
- UTF-16 LE BOM bei `Out-File` ohne `-Encoding utf8` (PowerShell-Gotcha)
- `set VAR=value` (kein `export`)

**Folge:** In `package.json`-Scripts immer Cross-Platform-tauglich (`cross-env`, `rimraf` statt `rm -rf`).

### G02 — Multi-Line-Commits via `.commitmsg.txt`

`git commit -m "line1\nline2"` funktioniert in cmd nicht — String wird flach. Pattern:

```cmd
echo feat: short subject> .commitmsg.txt
echo.>> .commitmsg.txt
echo Long body line 1>> .commitmsg.txt
echo Long body line 2>> .commitmsg.txt
git commit -F .commitmsg.txt
del .commitmsg.txt
```

Oder vorher den Body in einer Datei `commit-msg.tmp` schreiben und `git commit -F commit-msg.tmp`.

### G03 — Push-Remote-Regel: NIEMALS auf `origin` des Bootstrap-Kits

Wenn der Bootstrap-Kit aus einem fremden Remote gecloned wurde, ist `origin` auf dieses fremde Remote gesetzt. Nach `git push` landen die Commits dort. Pattern:

```bash
git remote -v   # vor jedem push prüfen
git remote rename origin upstream
git remote add origin git@github.com:{{REMOTE_NAME}}/{{PROJEKT_NAME}}.git
git push -u origin main
```

### G04 — Lokaler OOM beim `npm ci` mit aktiven Workspaces

Wenn `package.json` mit `"workspaces": ["packages/*"]` + viele Packages, kann `npm ci` >8 GB RAM brauchen. CK-Maschine hatte mehrere OOM-Episoden (Phase 1b in DiggAi).

**Fix:**
1. Erst `npm ci --legacy-peer-deps --ignore-scripts` (lädt nur, baut nicht).
2. Dann pro Package separat `npm run build`.
3. `NODE_OPTIONS=--max-old-space-size=8192` setzen.
4. Wenn weiterhin OOM → Codespace 4-core/16GB (siehe `02_METHODIK_WORKFLOW.md` § 9).

### G05 — i18next-Falle bei flat dotted Keys

Wenn `translation.json` so aussieht:
```json
{ "home.subtitle": "..." }
```

Default-i18next interpretiert den Punkt als Nesting-Separator und sucht `{ home: { subtitle: "..." } }`. Resultat: `[?] subtitle`-Fallbacks im UI.

**Fix in `src/i18n.ts`:**
```ts
i18n.init({
  keySeparator: false,
  nsSeparator: false,
  // ...
});
```

Nach Fix: Vite-Dev-Server neu starten (i18n-Init wird beim Boot geladen) + Browser-Hard-Refresh (LocalStorage-Cache leeren).

### G06 — Fly.io Cold-Start

Default Fly-App pausiert bei Inaktivität. Erster Request nach Pause = 8–15 Sek. Latenz. **Fix in `fly.toml`:**

```toml
[[services]]
  internal_port = 3001
  protocol = "tcp"
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
```

`min_machines_running = 1` heißt zwar laufende Kosten (~\$2/Mon Free-Tier), aber Health-Checks <500 ms.

### G07 — Neon-Branching für sichere Migrations-Tests

Vor Risiko-Migration auf Prod-DB:
```bash
neonctl branches create --name pre-migration-{{datum}} --project-id {{id}}
# Migration auf Branch testen
prisma migrate deploy --schema ./prisma/schema.prisma   # mit BRANCH_URL
# Wenn grün, auf Prod-Branch ausführen
```

### G08 — Browser-Tier in Cowork = read-only

Computer-Use granted Chrome im Cowork-Mode tier `read`. Screenshots ja, Klicks nein. Folge: Hetzner-/GitHub-/Stripe-UI-Klicks **muss CK selbst machen**. Agent gibt klick-Anleitung + Screenshot, CK führt aus.

### G09 — Gmail-MCP `create_draft` ohne Attachments

`mcp__gmail__create_draft` legt nur Text-Body an. Für Anhänge: Draft anlegen, CK klickt im Gmail-UI das Büroklammer-Symbol, hängt DOCX/PDF an, klickt "Senden".

### G10 — Web-Fetch + große GitHub-HTML-Seiten

`mcp__workspace__web_fetch` auf `https://github.com/owner/repo/blob/main/README.md` → "exceeds maximum allowed tokens". **Fix:** `raw.githubusercontent.com`-URL nehmen:

```
https://raw.githubusercontent.com/{{owner}}/{{repo}}/main/{{path}}
```

### G11 — Tailwind Dark-Mode nicht in Build wegoptimiert

Wenn `dark:bg-zinc-900` in JSX, aber Tailwind's PurgeCSS optimiert raus weil der Mode-Toggle dynamisch ist: in `tailwind.config.js` die Klasse safeListen:

```js
module.exports = {
  safelist: [
    { pattern: /^(bg|text|border)-(zinc|white|black)/, variants: ['dark'] },
  ],
  darkMode: 'class',
};
```

### G12 — Multiple Cowork-Sessions parallel teilen Context **nicht**

Zwei Tabs sind komplett isoliert. Memory-Files (`memory/runs/`) sind die einzige Brücke. Daraus folgt: Run-Log-Discipline ist Pflicht — sonst weiß Tab 2 nicht was Tab 1 tat.

### G13 — Prisma + Node 24 + tsx: default-Import-Patch

Bei `prisma/seed.ts` o. ä. unter Node 24 + `npx tsx`:

```ts
// SCHLECHT (wirft "does not provide an export named 'PrismaClient'")
import { PrismaClient } from '@prisma/client';

// GUT
import pkg from '@prisma/client';
const { PrismaClient } = pkg;
```

Plus `require.main === module` → `import.meta.url`-Vergleich:
```ts
import { fileURLToPath } from 'node:url';
const __filename = fileURLToPath(import.meta.url);
if (process.argv[1] === __filename) {
  main();
}
```

### G14 — NODE_PATH-Trick für globale Module unter Workspace-Root

CK-Maschine: `npm config get prefix` = `D:\npm-global`. Build-Skripte im Workspace-Root ohne eigenes `node_modules`:

```cmd
set NODE_PATH=D:\npm-global\node_modules
node build-tracker-xlsx.cjs
```

---

## §2 Was funktioniert hat (wiederverwenden!)

### W01 — Bootstrap-Pattern: 7 Sub-Phasen sequenziell

0.1 Skelett → 0.2 Memory → 0.3 Ralph+Caveman → 0.4 PRD → 0.5 Run-Log → 0.6 Master-Cockpit → 0.7 Commit. Zwischen jeder Sub-Phase 1-Satz-Status. So bleibt CK im Bilde, ohne dass der Agent halbwegs angefangene Mega-Updates zerschneidet.

### W02 — AGENT_HANDOUT.txt für Cross-Tab-Handover

Wenn ein Cowork-Tab voll ist, schreibt der Agent eine 1-Page-Datei `AGENT_HANDOUT.txt` mit: aktuelle Tasks, welche Tools nötig, alle Gotchas, wo Prod liegt, was als nächstes. Neuer Tab pastet die Datei als erste Message → in 30 Sek up-to-speed.

### W03 — Tenant via Subdomain (`{{tenant}}.{{rootdomain}}.de`)

Praxis-/Klinik-Multi-Tenant am saubersten via Subdomain. DNS-Wildcard `*.diggai.de` → Backend liest `req.hostname.split('.')[0]` als Tenant-ID. Single-Database mit `tenantId`-Column in jeder Tabelle + Row-Level-Security falls Postgres.

### W04 — Pricing als Single-Source-of-Truth

Pricing in **einer** Datei (`pricing.ts` oder `pricing.json`), nicht inline in der UI. Stripe-Produkte über Skript daraus generieren. Wenn Pricing-Update: nur Datei anfassen, Stripe-Sync-Skript laufen lassen.

### W05 — Brief-Index in `docs/letters/`

Alle generierten Mails (BfArM, Anwälte, Investoren) als `.md` in `docs/letters/`. README dort listet was an wen ging mit Datum. Bei Audit/Anwalts-Frage einfach Repo-Search statt Mail-Suche.

### W06 — Master-Cockpit "Was tust du jetzt?"-Tabelle

`{{PROJEKT_NAME}}_MASTER.md` mit ≤95 Zeilen, oberste Sektion ist eine Tabelle mit Spalten: `Wer | Was | Bis-Wann | Status`. CK schaut morgens drauf, weiß sofort was zu tun ist. Längere Detail-Tabellen drunter, aber Top-Tabelle = Pareto-3-View.

### W07 — Konvergenz-Erkennung (zweimal denselben Versuch → STOP)

Wenn Ralph oder Agent zweimal hintereinander den gleichen Fix probiert und scheitert, NICHT dreimal probieren. In `VERSTAENDNIS_LUECKEN.md` `Q<N>` eintragen, an CK eskalieren.

### W08 — Run-Log-Lauf-Nummer pro (agent, model, Tag)

Format `<NN>` ist monoton wachsend pro (agent, model) pro Kalendertag. Beispiel Lauf 14 in DiggAi: `2026-05-13_kimi_k2-14.md`. Beim nächsten Tag startet `01` neu. Suchbarkeit hoch, Konflikte vermieden.

### W09 — DoD-Liste direkt in `CLAUDE.md`

Die 11-Punkte-DoD aus `02_METHODIK_WORKFLOW.md` § 5 wörtlich in `CLAUDE.md` einkopieren. Ralph + Claude Code lesen `CLAUDE.md` automatisch — DoD-Check passiert nicht mehr "wenn ich daran denke" sondern bei jedem Run.

### W10 — Status-Plan-Flip-Pattern

Wenn Strategie/Regulatorik sich ändert, ein `docs/STATUS_PLAN_FLIP.md` schreiben — vorher/nachher + Begründung + Pareto-3 für CK. Behält Architektur-Historie und macht klar was kippt.

---

## §3 Was NICHT funktioniert hat (vermeiden!)

### F01 — Multiple Entry-Points (z. B. `dev:client`, `dev:server`, `dev:agent` als 3 separate Bats)

Drei Bats = drei Terminals = drei Stellen wo was schief gehen kann. **Pattern:** `npm run dev:all` mit `concurrently`, alles in einem Fenster, sauberes Color-Prefixing.

### F02 — Drift-fähige Platzhalter (`<HIER WERT>`, `TODO`, `XXX`)

Im Code unsichtbar, im PR-Review übersehen, in Prod gefunden. **Fix:** Einheitlich `{{PLATZHALTER}}` + Grep-Check vor Commit.

### F03 — Mega-Stories ("Build the dashboard")

Context-Overflow in Ralph-Iteration garantiert. Stories so dimensionieren, dass eine Story ≤30 Min Coding ist.

### F04 — Stack-Sprünge mid-Projekt (z. B. Express → Fastify)

Migrations-Kosten >>> Performance-Gewinn. Stack festlegen, durchziehen. Refactor erst wenn echter Schmerz.

### F05 — Direkter `master`/`main`-Push für nicht-trivial Changes

Selbst Solo-Projekte: Feature-Branch + Self-PR + Quality-Checks im CI. Direkt-Push für `docs/*` und `chore: bump version` ok.

### F06 — Halluzinierte Bootstrap-Refs ("siehe Section X.Y" wenn X.Y nicht existiert)

Beim Schreiben von Doku immer mit Grep verifizieren, dass referenzierte Sections wirklich existieren.

### F07 — Lokaler Build "zu klein" probieren (4 GB RAM nicht ausreichend)

Wenn `npm run build` failed wegen OOM, nicht 5× retryen — Codespace 16 GB öffnen.

### F08 — Chrome für Forms in Cowork (read-only → kein Klick möglich)

Wenn das Ziel ist, in einem Web-Form was einzutippen: claude-in-chrome MCP (tier full, kann Form-Input) statt computer-use auf Chrome (tier read, blockt).

### F09 — Auto-Send von Mails

NIE. Agent legt Draft an, CK klickt "Senden". Auch wenn der Inhalt 100% sicher korrekt ist — Compliance + CK-Kontrolle = nie automatischer Versand.

### F10 — Force-Push auf shared Branches

`git push --force` auf `main` oder einen Branch wo eine andere Session läuft = Datenverlust. Nur `--force-with-lease` auf eigene Feature-Branches mit expliziter CK-Bestätigung.

### F11 — 200+ offene Items im Tracker (Backlog-Friedhof)

Wenn Open-Items >50, ist die Tracker-Disziplin tot. Stattdessen: Pareto-3-View oben, Detail-Items darunter, "Done"-Items in `docs/done.md` archivieren. Pro Quartal Open-Items radikal kürzen.

---

## §4 Domain-Regulatorik (nur wenn Medical-Device / DiGA / Fintech / DSGVO Art. 9)

Wenn `{{DOMAIN_REGULATORIK}}` = `KEINE`, kannst du diese Sektion überspringen. Sonst: bewusst aktivieren.

### Klassifizierungs-Spielraum nutzen

Bei Medical-Device-Software ist die Klassen-Frage (Class I vs Class IIa vs IIb) **massiv** Outcome-bestimmend. Pattern aus DiggAi:

- **Produkt-Split:** Daten-Erfassung (Class I) vs Decision-Support (Class IIa) trennen — separate Bundles, ESLint-Guard zwischen den Ordnern, Bundle-Audit im CI.
- **Intended-Purpose-Disziplin** nach MDCG 2019-11 § 3.5 / § 4.2 / § 6.1: keine Sätze, die "Diagnose-Unterstützung" andeuten, in marketing-facing Strings.
- **Technische Bundle-Trennung:** Postgres-Rollen pro Modul, separate Container-Builds.
- **Selbstverifizierung (Class I)** spart 50–70 k€ Erst-Zertifizierung + 15–30 k€/Jahr Surveillance vs Notified-Body-Pfad.
- **Fast-DiGA** als separates Patienten-Modul nach § 139e SGB V — eigener Pfad, nicht im Hauptprodukt.

### Pflicht-Dokumente (Minimum für Class I Selbstverifizierung)

1. Konformitätserklärung MDR Anhang IV (10 Sektionen, unterschriftsreif)
2. Clinical Evaluation Report (CER) — auch für Class I, kürzer
3. Instructions for Use (IFU)
4. Risk Management File (ISO 14971-konform)
5. Post-Market-Surveillance-Plan (PMS)
6. Vigilance-Verfahren (Meldewege für Vorkommnisse)

### Externe Stakeholder

- **BfArM** (`mp-beratung@bfarm.de`) — kostenlose Sprechstunde für Klassifizierungs-Fragen. Antrag-Template in `05_REPORTS_TEMPLATES.md` § 9.
- **Anwalts-Pool:** spezialisierte Med-Tech / Health-IT-Kanzleien (Search-Terms: "MDR Klassifizierung Anwalt", "Digital Health Recht"). Erstgespräch typischerweise 500–2.500 €, Class-I-Begutachtung 2.000–5.000 €. Mail-Template in § 10.
- **Notified Body** (nur wenn Class IIa+): TÜV SÜD, DEKRA, MDC. Erst-Zertifizierung 50–70 k€.
- **AVV-Verträge mit Subprozessoren:** Fly.io, Neon, Netlify, GitHub. Templates vom Anwalt nach DSGVO Art. 28.

### DSGVO-Pflicht-Trio

- AGB
- Datenschutzerklärung (DSE) mit SCC + TIA
- Auftragsverarbeitungsvertrag (AVV)

Pattern: pro neuem Projekt einmal Anwalts-Review aller drei Dokumente (~3–5 k€), dann Templates wiederverwenden und nur die Tenant-/Produkt-spezifischen Stellen anpassen.

---

## §5 CK Soft-Preferences (Tabelle)

| Aspekt | CK-Default |
|---|---|
| Sprache Konversation | DE |
| Sprache Tech-Begriffe | EN |
| Listen-Länge | max 5, sonst Tabelle |
| Emojis | NIE dekorativ. Funktional ok (✅/❌/🔴/🟡) |
| Anrede | direkt, kein "Sie", oft kein "Du" |
| Bestätigungs-Stil | knapp ("ok", "weiter", "mach das") |
| Frustrations-Trigger | zu viel Hand-holding, zu konservativ, zu viele Klärfragen |
| Pareto-Denken | 3 Items, Rest wartet |
| Run-Log-Disziplin | non-negotiable |
| Multi-Tab-Workflow | parallel mehrere Sessions, Memory-Files überbrücken |
| Hosting-Region | EU-Frankfurt durchgängig (DSGVO) |
| Repo-Sichtbarkeit | privat default, public nur explizit |
| Lizenz-Default | MIT (ändern wenn anders gesagt) |

---

## §6 Memory-Snippets für `CLAUDE.md` (Repo-Root)

Zum direkten Kopieren in das `CLAUDE.md` neuer Projekte (im Block `## Memory / Learnings`):

```markdown
## Memory / Learnings (kopiert aus 04_MEMORY_LEARNINGS.md)

### Hard-Rules (NIEMALS verletzen)

- Niemals `git push` ohne `git status` (3× verbrannt)
- Niemals Force-Push auf shared Branches — nur `--force-with-lease`
- Niemals Auto-Send von Mails — CK klickt "Senden"
- Niemals echte Secrets / Tokens commiten — `.env` + `{{PLATZHALTER}}`
- Niemals auf `master` ohne PR mergen

### Stack-Gotchas (dieses Projekt)

- Windows-cmd: kein `&&`, kein `??`, Backslash-Pfade
- Multi-Line-Commits: `.commitmsg.txt`-Trick
- Node 24 + Prisma 6 + tsx: default-Import-Patch
- `npm install` im Workspace-Root: `NODE_PATH=D:\npm-global\node_modules`
- `tsc -b`: `NODE_OPTIONS=--max-old-space-size=8192`
- Lokaler OOM bei Workspaces: Codespace 4-core/16GB fallback
- i18next + flat dotted keys: `keySeparator: false, nsSeparator: false`
- Fly.io Cold-Start: `min_machines_running = 1` in `fly.toml`
- Cowork-Chrome ist tier "read" → CK-Klicks oder claude-in-chrome MCP
- Gmail-MCP `create_draft` ohne Attachments → CK hängt manuell an
- web_fetch + GitHub-HTML: `raw.githubusercontent.com` benutzen

### CK-Style

- DE Konversation, EN Tech-Begriffe
- max 5 Listen-Items, sonst Tabelle
- 3-Punkte-Pareto bei Status-Reviews
- Knappes Bestätigen ("ok", "weiter")
- Niemals "großartig", "wunderbar", "Super-Tipp" — keine Marketing-Sprache
```

### G15 — Browser drosselt JS in Background-Tab

Polling-basiertes UI funktioniert im Foreground-Tab, hängt im Background-Tab. Chrome/Safari drosseln Background-JS auf ≥1 min Intervall (Battery-Saving).

**Symptom:** Realtime-Chat/Notifications "verzögert sich", User wartet, Tab in Vordergrund — alles da.
**Fix kurzfristig:** UI-Hinweis "Tab im Vordergrund halten für Live-Updates".
**Fix mittelfristig:** WebSocket statt Polling, oder Service-Worker mit Push-Notifications.
**Verifikation:** DevTools → Performance → "Throttling: Background" simulieren.

### G16 — Service-Worker cached kaputten Build nach Deploy

Nach Deploy mit Bug serviert der SW weiter den alten Build. Reload zeigt weiterhin kaputten Stand, selbst Hard-Refresh hilft nicht.

**Schnell-Diagnose:** Inkognito-Tab öffnen — funktioniert es da, ist's der SW-Cache.
**Fix:** `skipWaiting()` + `clientsClaim()` im SW, Cache-Version-Bump bei jedem Build via Git-SHA in `manifest.json`. Notfall für User: Browser → Site-Daten löschen.

### G17 — Bash-Sandbox kann mit HYPERVISOR_SERVICE_ERROR ausfallen

`mcp__workspace__bash` antwortet `HYPERVISOR_SERVICE_ERROR`. Alle `gh`/`git`/`curl`-Operationen aus der Sandbox brechen.

**Fix:** Cowork-Tab schließen + neu öffnen (Sandbox wird neu provisioniert). **Besser:** Statt `workspace__bash` direkt `mcp__Desktop_Commander__*` benutzen — DC läuft direkt auf dem Operator-PC, kein Sandbox-Risiko, gh/git funktionieren wie nativ. Vor jeder bash-abhängigen Phase Smoke-Test: `echo ready`.

### G18 — web_fetch Provenance ist URL-strikt, nicht Domain-basiert

Auch wenn URLs im Content eines früheren `web_fetch`-Results stehen, sind sie nicht automatisch in der Provenance. Folge-Fetches werden mit "URL not in provenance set" abgelehnt.

**Fix:** Operator pastet URL explizit in nächster Nachricht. **Besser:** Chrome-MCP (`mcp__Claude_in_Chrome__navigate` → `get_page_text`) oder Desktop-Commander `gh api`/`curl` statt web_fetch — beide haben keinen Provenance-Check.

---

## §X — Anhang aus JoBetes-Sync 2026-05-19

> Diese Einträge stammen aus dem JoBetes-Sync vom 19. Mai 2026. **Reviewer-Aktion beim Merge:** in die thematisch passenden Sektionen oben einsortieren (G15-G18 zu § 1 Gotchas, W11-W14 zu § 2 Was funktioniert hat). Alternativ: als eigene Sektion "Sync-Anhänge" stehen lassen, dann ist die Herkunft der Einträge nachvollziehbar.

### W11 — Mock-Fallback für externe API-Provider

Vision/LLM-Provider liefert deterministisch dieselbe Antwort, wenn API-Key fehlt. Dev-Build ohne Cloud-API, Test-Suite ohne Mocking-Library-Overhead.

```ts
export async function callProvider(args) {
  if (!process.env.PROVIDER_API_KEY) {
    console.warn('[MOCK] no API key, returning deterministic mock');
    return mockResponse(args);
  }
  return realCall(args);
}
```

**Vorteil:** Test-Suite läuft ohne API-Keys, Dev-Build ohne Cloud-Zugriff, CI ohne Secret-Leak-Risiko.
**Risiko:** In Production prüfen, dass API-Key gesetzt ist — sonst läuft Mock live.

### W12 — shared-schemas-Package als FE/BE-Single-Source-of-Truth

`packages/shared-schemas/` exportiert alle Zod-Schemas. Frontend nutzt für Form-Validation, Backend für Request-Body-Validation, beide via `z.infer` denselben TS-Type.

```ts
// packages/shared-schemas/patient.ts
export const PatientSchema = z.object({ name: z.string().min(1), ... });
export type Patient = z.infer<typeof PatientSchema>;

// apps/web/PatientForm.tsx
import { PatientSchema } from '@<projekt>/shared-schemas';

// apps/api/routes/patients.ts
import { PatientSchema } from '@<projekt>/shared-schemas';
```

Kein Schema-Drift mehr möglich.

### W13 — Triple-Output für Konzept-Docs (.md + .html + ASCII .txt)

Bei jedem strategischen Doc drei Versionen:
- `.md` für GitHub/IDE-Editierung.
- `.html` für Browser-Screenshot (Stakeholder-Versand).
- ASCII `.txt` für maschinen-getrennten Ground-Truth (Diff-friendly, kein Renderer-Overhead, canonical für Tool-zu-Tool-Übergaben).

CK-Brand-Regel: gilt projektübergreifend für Konzepte/Architekturen.

### W14 — Operator-passiv-Pattern via Desktop Commander

Operator klickt nur wenn nötig (max 1 Klick, alles vorbereitet). Agent macht git/push/deploy direkt via `mcp__Desktop_Commander__*` auf dem Operator-PC. Vorteile gegenüber `mcp__workspace__bash`:

- Kein Sandbox-Risiko (HYPERVISOR-Fehler etc.).
- Native gh/git/curl-Zugriff mit existierender Auth.
- Direktzugriff auf Operator-Dateisystem ohne Mount-Pfad-Übersetzung.

Niemals `.bat`-Files zum manuellen Ausführen vorbereiten — wenn etwas geklickt werden muss, sagt Agent EXAKT welcher Button.

---

## §X — Anhang aus Daily-Sync 2026-08-05

> Diese Einträge stammen aus dem Daily-Sync vom 5. August 2026 (Bank-Commit `9b63d38`). Ausgewählt wurde nur, was in **jedem** Projekt gilt — Domänenwissen blieb in der Bank. **Reviewer-Aktion beim Merge:** in die thematisch passenden Sektionen oben einsortieren (G19–G23 zu §1 Gotchas, M06–M09 zu §2/§3). Alternativ als Sync-Anhang stehen lassen, dann bleibt die Herkunft nachvollziehbar.
>
> Roter Faden dieses Syncs: **Werkzeuge, die Erfolg melden und nichts bewirken.** Vier der Einträge sind derselbe Fehler in vier verschiedenen Werkzeugen.

### G19 — `npm audit --omit=dev` filtert nicht zuverlässig

Gemessen: `npm audit --omit=dev` lieferte **dieselben 34 Meldungen wie ohne den Schalter**, inklusive eines Pakets, das unstrittig reines Bauwerkzeug ist. Dasselbe gilt für `npm ls --omit=dev --all`. Wer sich darauf verlässt, hält Bauwerkzeug für ausgelieferten Code oder umgekehrt — hier lagen **14 von 18 hohen** Meldungen tatsächlich im Produktions-Baum.

Verlässlich ist das **Lockfile** — genau daraus leitet `npm ci --omit=dev` das Produktions-Abbild ab:

```python
import json
pkgs = json.load(open('package-lock.json'))['packages']
prod = {}
for pfad, info in pkgs.items():
    if not pfad.startswith('node_modules/'): continue
    name = pfad.split('node_modules/')[-1]
    prod[name] = prod.get(name, False) or not (info.get('dev') or info.get('devOptional'))
```

Ein Paket gehört zur Produktion, sobald es **mindestens einen** Pfad ohne `dev`-Kennzeichen hat.

**Billigste Gegenprobe:** ein Paket suchen, von dem man sicher weiß, dass es Bauwerkzeug ist — taucht es in der „Produktions"-Liste auf, filtert der Befehl nicht.
**Nebenbefund:** `npm audit fix --package-lock-only` **ohne** `--force` brachte 34 → 7 und ließ `package.json` unverändert. ⚠️ Eine genannte „Fix-Version" kann ein **Rückschritt** sein (Artefakt der Advisory-Spannen), also vor dem Übernehmen gegen die aktuelle Version halten.

### G20 — `mktemp` mit Platzhaltern in der MITTE: BSD ersetzt `XXXXXX` nur am Ende

Eine Vorlage wie `job.XXXXXX.script` erzeugt unter BSD-`mktemp` (macOS) eine Datei mit dem **wörtlichen** Namen. Erster Lauf unauffällig; bleibt die Datei liegen, scheitert **jeder** weitere mit `File exists` — und der Vorgang bleibt still liegen.

```bash
# falsch (nur unter GNU-mktemp harmlos)
tmp=$(mktemp /tmp/job.XXXXXX.script)
# richtig, portabel
dir=$(mktemp -d /tmp/job.XXXXXX); tmp="$dir/job.script"
```

Steckte wochenlang unentdeckt drin, weil jeder Test nur **einen** Vorgang fuhr → vgl. M08.

### G21 — Ein FESTER Ersatzwert zur Verfälschung eines Zufallswerts ⇒ Flaky-Test

Ein Krypto-Zweigtest ersetzte die letzten zwei Hex-Zeichen eines Chiffrats fest durch `ff`, um eine Verfälschung zu erzeugen. Endet das Chiffrat selbst auf `ff`, verändert das nichts ⇒ Test rot in rund **1 von 256** Läufen. Kostete einen Pre-Push bei 1983 grünen Tests und sieht dabei wie ein echter Regressionsfund aus.

**Regel:** nie einen FESTEN Wert als Verfälschung eines Zufallswerts annehmen — den Ersatz **aus dem Ist-Wert ableiten** (Byte kippen/inkrementieren), dann ist die Verfälschung garantiert. Trifft jede Testhilfe, die „irgendetwas anderes" durch eine Konstante ausdrückt.

### G22 — Verändernde Browser-Automation meldet Erfolg ohne Wirkung

Drei Aufrufe an einem Vormittag meldeten Erfolg und wirkten nicht:

| Handlung | meldete | tatsächlich |
|---|---|---|
| `resize_window` 768×1024 | „Successfully resized" | `innerWidth` blieb 1858 (maximiertes Fenster, `outerWidth` = 0) |
| Sprache via `localStorage` | gesetzt | Auflösung ist `querystring → **cookie** → localStorage`; das Cookie schrieb den Wert beim Boot zurück |
| Kontrast per Canvas | 12 Mängel | **4** (Canvas schwarz vorgefüllt ⇒ Deckkraft immer 255) |

Fall 1 erklärte **rückwirkend**, warum Responsivität zuvor als „nicht messbar" galt: das Werkzeug meldete Erfolg statt Fehlschlag.

**Regel:** jede verändernde Messung liest die Veränderung **im selben Aufruf** nach. Und bei Zustands-Auflösung über mehrere Quellen immer die Reihenfolge nachlesen, statt die bequemste Quelle zu setzen.

### G23 — Zwei-Kopien-Sync: die falsche Richtung löscht die Arbeit in BEIDEN Kopien

Ein Harness-Werkzeug mit `capture` (Benutzerkopie → Repo) und `install` (Repo → Benutzerkopie) wurde in der falschen Richtung aufgerufen. Zwei fertige Ergänzungen im Repo waren danach in **beiden** Kopien weg, weil die ältere Benutzerkopie darüber schrieb. Der Aufruf meldete Erfolg.

**Merksatz ans Werkzeug hängen:** Repo bearbeitet ⇒ `install`. Benutzerkopie bearbeitet ⇒ `capture`. Nach jedem Lauf per `grep` auf **beiden** Seiten nachsehen. Werkzeuge, die in eine Richtung überschreiben, gehören mit Trockenlauf oder Vorab-Differenz versehen.

### M06 — Eine Datei ohne Indexzeile ist wie nicht geschrieben

In einem dateibasierten Gedächtnis wird pro Sitzung nur die **Index-Datei** geladen. Gemessen: **166 Dateien im Ordner, 133 Zeilen im Index** — 32 Erinnerungen lagen auf der Platte und waren unsichtbar, ausgerechnet die zur Übergabe und zum Betriebswissen.

**Vor jeder Standsaufnahme und jeder Übergabe:** Dateizahl gegen Indexzeilen zählen.

```bash
ls memory/*.md | wc -l          # gegen die Zahl der Indexzeilen halten
```

Weichen sie ab, ist der Stand unvollständig — **bevor** man ihn aufschreibt. Gilt für jedes Register, das seinen Inhalt nicht selbst aufzählt: Skill-Verzeichnisse, Migrationslisten, Test-Suiten mit expliziter Dateiliste.

### M07 — „Liegt nur lokal" ist eine Behauptung, kein Befund

Eine bereits versionierte Datei wurde überschrieben, weil der Ordner lokal anders hieß und die Dateien im Repo **anders nummeriert** waren — byte-gleicher Inhalt, anderer Name. Zweite Ausprägung am selben Tag: ein versioniertes Dokument verwies auf einen Pfad, der wie ein Repo-Pfad aussieht, aber außerhalb lag.

- Vor dem Kopieren **Inhalts-Hashes gegen den ganzen Baum** halten, nicht Dateinamen vergleichen.
- **Ein Pfad in einem versionierten Dokument ist kein Beleg, dass die Datei versioniert ist** — einmal messen (`git ls-files <pfad>`), statt es aus dem Kontext zu schließen.

Verwandter Fehler derselben Klasse: den Stand **einer** Arbeitsmaschine messen und als Gesamtstand ausgeben. Die Frage lautet nicht „was liegt hier?", sondern **„gab es für diese Art Material überhaupt je einen Kanal auf diese Maschine?"**

### M08 — Ein Fehler, der erst beim ZWEITEN Lauf auftritt, ist mit EINEM Testvorgang unauffindbar

Ein Durchlauf über zehn Dienste mit je eigenem Datensatz förderte drei Fehler zutage, die **alle** seit Wochen drinsteckten und **alle** diese Bauart hatten (u.a. G20, G21). Ein einzelner Testvorgang hätte keinen davon gezeigt.

**Abnahme über die Matrix fahren, nicht am Beispiel:** jeden Dienst/Pfad einmal echt durchspielen, mit je eigenem Datensatz — und mindestens einen Pfad **zweimal hintereinander ohne Aufräumen**. Idempotenz und Wiederholbarkeit sind eine eigene Testdimension, keine Folge eines grünen Erstlaufs.

### M09 — Zu jeder Positiv-Messung gehört die Gegenrichtung

Eine Zustandserkennung war als „belegt" gemeldet worden — gemessen war nur **eine** Richtung. Tatsächlich stimmte die Abfrage nur für die Vordergrund-Anwendung; lag die Gegenseite im Hintergrund, galt das fehlende Merkmal als Beweis für den Negativzustand. Folge: **jeder** Vorgang blieb mit einer überzeugend klingenden, falschen Begründung liegen.

Ein Prädikat gilt erst als belegt, wenn **beide** Richtungen gemessen sind: der Positivfall *und* mindestens ein Fall, in dem es `false` liefern muss und tatsächlich liefert. Besonders bei Erkennungen, deren Negativausgang einen stillen Halt auslöst — dort ist ein falsches `false` teurer als ein Absturz, weil es sich als Diagnose tarnt.

*Quelle: Daily-Sync 2026-08-05 (G197/G201/G202/G203/G205 + M104–M107 in Zentrale-DiggAi-Bank).*
