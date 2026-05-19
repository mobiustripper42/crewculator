# Crewculator

Internal payroll tool. External name: **Xola Tip Extractor**.

Pulls tip data from the Xola API for a biweekly pay period, splits each event's tip pool evenly among the guides assigned to that event, and downloads a Gusto-ready CSV (`Employee Name, Tips`). Replaces a manual ~30-minute export/cross-reference process with a 30-second one.

**Operators:** Spink (primary), Brendan (backup). One page, one button. No login.

## Shape

- Static `public/index.html` (vanilla JS, no framework, no build)
- One Netlify Function (`netlify/functions/tips.js`) holding the Xola API key server-side
- Deployed to a standalone Netlify site (see [DEC-005](docs/DECISIONS.md))

## Local dev

```bash
npm install
cp .env.example .env.local   # fill in XOLA_API_KEY, XOLA_SELLER_ID, XOLA_API_BASE
netlify dev                  # http://localhost:8888
```

## Tests

```bash
npm test                     # node --test (unit, mainly the split function)
npm run test:smoke           # Playwright smoke (one test, mocked function response)
```

## Docs

| File | Purpose |
|------|---------|
| [`CLAUDE.md`](CLAUDE.md) | Project context for Claude Code |
| [`docs/SPEC.md`](docs/SPEC.md) | Scope, philosophy, V1 vs Not-V1 |
| [`docs/PROJECT_PLAN.md`](docs/PROJECT_PLAN.md) | 3 phases, ~29 pts |
| [`docs/DECISIONS.md`](docs/DECISIONS.md) | Why each architectural choice (DEC-001..006 + DEC-TBDs) |
| [`docs/USER_STORIES.md`](docs/USER_STORIES.md) | Operator stories (OP-1..OP-9) |
| [`docs/BRAND.md`](docs/BRAND.md) | Voice + visual direction (minimal — it's a payroll tool) |
| [`docs/AGENTS.md`](docs/AGENTS.md) | Claude Code agents + slash-command skills |
| [`docs/API_NOTES.md`](docs/API_NOTES.md) | Created in Phase 0 — actual Xola response shapes |

## Status

Planning complete (2026-05-18). Phase 0 starts next session via `/its-alive`.

**Maintainer:** Brendan. (Eric planned; Brendan implements.)

## Handoff — first-time setup for Brendan

Hey Brendan — Eric here. This is your project. The notes below are the one-time setup; once it's done, Claude Code does most of the plumbing for you. You'll learn the dev tooling as we go — don't worry about understanding every term up front.

If anything below is unclear, ask Claude in your first session: it has instructions to translate dev-speak into plain English.

### One-time setup on your laptop

1. **Install Claude Code** — go to [claude.ai/code](https://claude.ai/code) and follow the install steps. This is the tool you'll spend most of your time in; it's a chat that can also edit files and run commands on your laptop.

2. **Tell Claude Code who you are.** Open a terminal (Terminal.app on Mac, Windows Terminal on Windows) and paste this in:
   ```bash
   echo "brendan" > ~/.claude/devname
   ```
   This writes your name to a small config file so the per-session logs are tagged with you instead of a generic placeholder.

3. **Get the code onto your laptop.** Still in the terminal:
   ```bash
   git clone https://github.com/mobiustripper42/crewculator.git
   cd crewculator
   npm install
   ```
   `git clone` downloads the repo. `cd crewculator` moves you into its folder. `npm install` downloads the libraries the project needs (Playwright for tests, Netlify CLI for local dev). Run these in order.

4. **Set up your secret keys.** The Xola API needs a key, and we keep it out of the public code. Make a file called `.env.local` in the project folder. Open `.env.example` to see the shape — copy it to `.env.local` and fill in the real values. Eric will send you:
   - `XOLA_API_KEY` — your Xola sandbox key
   - `XOLA_SELLER_ID` — your Xola account ID
   - `XOLA_API_BASE` — set to `https://sandbox.xola.com/api` for now (production comes in Phase 2)

   `.env.local` is in `.gitignore`, which means git will never upload it. Good — keys should stay on your laptop.

5. **Read the planning docs**, in this order. Don't try to memorize them — just skim so you know what's where.
   - [`CLAUDE.md`](CLAUDE.md) — context for Claude Code, plus a "Working with Brendan" section that tells Claude how to explain things to you
   - [`docs/SPEC.md`](docs/SPEC.md) — what we're building and what's explicitly out of scope
   - [`docs/DECISIONS.md`](docs/DECISIONS.md) — why we made each design choice; the four "DEC-TBD" entries are open questions you'll answer in Phase 0
   - [`docs/PROJECT_PLAN.md`](docs/PROJECT_PLAN.md) — phases and tasks, with point estimates
   - [`docs/AGENTS.md`](docs/AGENTS.md) and [`docs/CHEATSHEET.md`](docs/CHEATSHEET.md) — the slash-commands you'll use (`/its-alive`, `/kill-this`, `/its-dead`, `/retro`) and what each does

6. **Start your first session.** In the terminal, inside the `crewculator` folder:
   ```bash
   claude
   ```
   That opens Claude Code. Then type:
   ```
   /its-alive
   ```
   That's a slash-command — it triggers a setup routine that stamps the start time, reads where we left off, and recommends what to work on next (Phase 0 task 0.1).

### How the merge flow works

You'll be working on **branches** — copies of the code where you can make changes without touching the live `main` branch. When you finish a task, Claude runs `/kill-this`, which opens a **pull request** (PR) — a request to merge your branch into `main`.

Eric reviews every PR before it merges. This is intentional: he's responsible for payroll-day correctness, so he eyeballs every change. After he approves, either of you can click the merge button. If you push more commits to the branch after approval, the approval is dismissed and Eric re-reviews — so it's better to finish a task fully before asking for review.

You don't merge your own PRs. The merge button will look clickable, but the ruleset on the repo will block it without Eric's approval.

### Pushing back is encouraged

The plan was built before you were on the project. If a task doesn't make sense, the wording is confusing, or a point estimate feels wrong, say so in your session — Claude will either clarify (if the plan is fine but unclear) or help revise the plan (if the plan itself is wrong). You're not breaking anything by asking.
