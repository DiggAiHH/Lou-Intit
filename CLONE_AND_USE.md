# Clone & Use — One-Page Workflow Guide

Follow these four steps to onboard **Claude Opus 4.7** onto a new project using this bootstrap kit.

---

## Step 1 — Clone the kit into your project

In your new project root, run:

```bash
git clone https://github.com/DiggAiHH/Lou-Intit _bootstrap
```

This creates a `_bootstrap/` directory containing the entire bootstrap kit.

---

## Step 2 — Fill in the mandatory placeholders

Open `_bootstrap/00_SYSTEM_PROMPT_BOOTSTRAP.md` and replace the four mandatory placeholders:

| Placeholder | Description |
|---|---|
| `{{PROJEKT_NAME}}` | Short identifier for the project (e.g. `MyApp`) |
| `{{KURZBESCHREIBUNG_2_SAETZE}}` | Two-sentence description of what the project does |
| `{{ZIELNUTZER}}` | Target user / persona (e.g. `Solo founder, B2B SaaS`) |
| `{{REMOTE_NAME}}` | Git remote name (usually `origin`) |

---

## Step 3 — Open a fresh Claude Opus 4.7 session

1. Open a new **Claude Cowork** tab with **Opus 4.7** selected.
2. Copy the entire block between `─── PROMPT START ───` and `─── PROMPT ENDE ───` from `00_SYSTEM_PROMPT_BOOTSTRAP.md`.
3. Paste it into the chat and send.

---

## Step 4 — Let Opus run Phase 0 automatically

Opus reads the 5 annex files (`01_TECH_STACK.md` → `05_REPORTS_TEMPLATES.md`) automatically and executes **Phase 0** in sequence:

1. Project skeleton
2. Memory files initialisation
3. Install Ralph + Caveman
4. First PRD draft
5. First run-log entry
6. Master cockpit setup
7. Initial commit & push

Once Phase 0 is complete, your project is fully bootstrapped and ready for feature development.
