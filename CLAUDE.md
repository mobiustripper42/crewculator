# Crewculator — Claude Code Project Context

## What We're Building

Crewculator (external name: **Xola Tip Extractor**) is a single-page web tool that pulls tip data from the Xola API, splits tips evenly per guide per event for a biweekly pay period, and downloads a Gusto-ready CSV. Replaces a manual ~30-minute export-and-cross-reference process with a 30-second one.

Roles:
- **Payroll Operator** — runs the tool on payroll day. Spink is primary, Brendan is backup. There is exactly one role; the tool has no auth, no multi-user state.

## Working with Brendan (the dev on this project)

Brendan is a smart, tech-comfortable business major. He is **not a developer** — zero formal dev experience. He's running Claude Code because Eric set him up; he is not coming in with a CS background or years of shell time. Treat him like a sharp junior pair-programmer, not a peer who already speaks the language.

**He's fluent in:** using a computer, GitHub the website (browsing, reading PRs), copy/paste, environment variables as a concept, what an API is at a high level, Markdown, CSVs. He picks up new tools fast.

**He is NOT fluent in:** git internals (branches, merges, rebases, force-push, fast-forward, orphan branches), the `gh` CLI, npm/Node, package.json, what a test runner does under the hood, the difference between a serverless function and a server, async/await, the seeds workflow vocabulary (`/its-alive`, "task branch", "kill-this", `/retro`). Assume zero of this is known until he demonstrates it.

**How to communicate with him:**

- **Translate dev vocabulary the first time it appears in a session.** "We're going to make a *branch* — that's a copy of the code where we can experiment without touching the live version" beats "cut a task branch." Once he's seen the term, don't keep re-explaining — he'll retain it.
- **Show the exact command + one sentence on what it does.** Not "push your branch" — instead `git push -u origin task/0.1-skeleton` (this uploads our work to GitHub so it's safe off your laptop). Always the command, always the human translation.
- **Narrate scary or destructive actions before doing them.** Before `rm`, force-push, branch deletes, `git reset`, or anything that could lose work: stop, explain plainly what will happen, wait for a yes.
- **Lean on the seeds skills.** `/its-alive`, `/kill-this`, `/its-dead`, `/retro` exist precisely so he doesn't have to learn git plumbing. When he hits a point a skill handles, use the skill — don't walk him through the underlying commands manually.
- **Translate jargon-heavy agent output.** When @code-review or @architect produces dense findings, lead with one or two plain-English bullets ("the math function isn't being tested" / "we're calling the API twice when once would do") before showing the raw output.
- **Don't condescend.** He's sharp. The rule is "don't assume dev vocabulary," not "explain every line." If he says "got it, move on," move on. Skip the "great question!" prefaces.
- **He can push back on the plan.** Eric planned this without Brendan in the room. If a task doesn't make sense to him, that's signal — either the plan needs clarifying (Claude's job) or the plan is wrong (revisit with @architect). Make space for the question.

**The bar for the project:** by the end of Phase 0 he can run `/its-alive`, make a small code change, run `/kill-this`, and watch a PR land. By the end of Phase 2 he can deploy and hand it to Spink. He won't be a developer at the end — but he'll have shipped a real tool, and he'll know what each command actually did.

## Stack

- **Frontend:** Static HTML + vanilla JS (no framework, no build step)
- **Backend:** Single Netlify Function (Node 18+, native `fetch`)
- **Hosting:** Netlify (standalone site)
- **Secrets:** Netlify env vars (`XOLA_API_KEY`, `XOLA_SELLER_ID`, `XOLA_API_BASE`) — never exposed to the browser
- **Database:** Not used — the tool is stateless; every run pulls fresh from Xola
- **Payments / Notifications / Auth:** Not used
- **Testing:** Node test runner (`node --test`) for the tip-split function, one Playwright smoke test for the page against a mocked function response

## Key Docs
| File | Purpose |
|------|---------|
| `docs/SPEC.md` | What we're building — scope, V1 vs V2 |
| `docs/DECISIONS.md` | Why we made each architectural choice (DEC-001..004 + DEC-TBD for open API questions) |
| `docs/USER_STORIES.md` | Stories for the single Payroll Operator role |
| `docs/PROJECT_PLAN.md` | 3 phases, ~29 pts. Read at planning, written at retro. Current-phase tasks live in GitHub Issues. |
| `docs/RETROSPECTIVES.md` | Phase-end retros written by `/retro` |
| `docs/AGENTS.md` | Agent and skill specs |
| `docs/BRAND.md` | Voice, visual direction (minimal — it's a payroll tool, not a product) |
| `docs/API_NOTES.md` | Created in Phase 0 — actual Xola response shapes captured against the sandbox |
| `sessions/*.md` (on orphan `sessions` branch via `.sessions-worktree/`) | Per-session files — `YYYY-MM-DD-HHMM-<dev>-<slug>.md`. Atomic after `/its-dead` closes (DEC-013); lives on the orphan `sessions` branch decoupled from any code branch (DEC-014). |
| `.claude/seeds-version` | Schema version this project was last installed at. Used by `/pull-seeds` to gate template syncs. |
| `.claude/project-type` | `tool` — `@sync-config` skips webapp-only files (`ui-reviewer.md`, Supabase scripts). |

## Core Data Model

No local database. The conceptual model lives in Xola:

```
Order (Xola)
  ├── addOns[]      → tip-bearing line items (matched by addon-name list)
  ├── gratuity?     → separate tip field if Phase 0 confirms it exists
  ├── event         → join key
  └── arrival       → date for pay-period bucketing

Event (Xola)
  └── guides[]      → assigned crew (id + name)

Guide (Xola)
  └── id, name      → reference cache; mapped to Gusto-name on output
```

Tip split rule (DEC-002): each event's total tip pool divides evenly among the guides assigned to that event. A 3-guide trip with $90 in tips → $30 per guide.

## Micro Workflow (per task, DEC-013 + DEC-014)

1. **Spec it** — poker estimate, acceptance criteria
2. **Plan it** — summarize files to create/edit + approach. Wait for explicit approval before writing code or running commands.
3. **Cut the task branch** — once the plan is approved: `git checkout -b task/X.Y-short-description` (from the session anchor or main). One branch per task.
4. **Build it** — implement the feature
5. **Write the test** — Node test for any pure logic (the tip-split function especially). Playwright smoke test if it touches the page.
6. **Run targeted tests** — `node --test tests/<file>.test.js` for unit; `npx playwright test tests/<file>.spec.ts --project=desktop` for smoke. Do NOT run the full suite — that's the user's call.
7. **Manual smoke (if UI touched)** — `netlify dev`, hit `http://localhost:8888`, exercise the change.
8. **Ship the task** — `/kill-this`. Commits code on the task branch, opens PR, appends a `## Task <N>` block to the session file (on the orphan `sessions` branch via `.sessions-worktree/`).
9. **Pick up another task** — start from step 1 again, with a new branch. `/its-alive` doesn't run again — same session.
10. **Close the Claude window** — `/its-dead` once, at the end. Stamps `ended:`, displays wall_clock to screen for gut-check, closes the session file. No time math, no version bump (those run at `/retro`).
11. **Merge PRs whenever** — order doesn't matter. PRs may merge mid-session, after `/its-dead`, or days later. Each merge deletes its task branch.
12. **Phase boundary** — `/retro` reads each session's `started`/`ended`/transcript/PR-timestamps to compute per-session time. Aggregates phase velocity. Patches version per merged PR, minor-bumps at close.

**No test, no push.**

**Full Playwright suite is never run automatically.** At the end of the session summary, ask: "Did you run the full Playwright suite yet?" and let the user decide.

## Migration Protocol

Not used — no database.

## Production Write Protection

Not used — no Supabase or other CLI-driven destructive ops. The Netlify API is browser-revertible (redeploy a prior commit).

The only production-affecting action is `netlify deploy --prod`, which the user runs manually; Claude must never run it without explicit instruction. Phase 2 task 2.3 covers this.

## Commands

```bash
# Local development
npm install                    # one-time
netlify dev                    # boots functions + static site at http://localhost:8888

# Testing
node --test                    # all unit tests under tests/
node --test tests/split.test.js  # single file
npx playwright test --project=desktop          # smoke test
npx playwright test tests/ui.spec.ts --project=desktop  # single file

# Deploy
netlify deploy                 # preview deploy (user runs this)
netlify deploy --prod          # production deploy (user runs this)
```

## Conventions

### JavaScript
- Plain ES modules. No TypeScript (it's a few files; the build-step cost outweighs the win).
- No `any`-equivalent laziness — use JSDoc `@param`/`@returns` on exported functions in `netlify/functions/`.
- Node 18+ built-in `fetch`. No `node-fetch`, no `axios`.

### Files
- `public/index.html` — the single page. Inline `<style>` and `<script>` are fine for V1; we extract only if the file passes ~300 lines.
- `netlify/functions/<name>.js` — one file per function. We expect exactly one function (`tips.js`).
- `netlify/functions/lib/` — pure helpers (split logic, date math, CSV generation). These are the only files with unit tests.
- `tests/*.test.js` — Node test runner.
- `tests/*.spec.ts` — Playwright (smoke only).

### Naming
- Files: `kebab-case.js`, `kebab-case.html`
- Functions: `camelCase`
- Env vars: `SCREAMING_SNAKE_CASE`, all prefixed `XOLA_` for the API client

### Secrets
- API key + seller ID live in Netlify env vars. **Never** in `index.html` or any client-shipped code.
- `.env.example` documents required vars; `.env.local` (gitignored) holds local dev values.

### Error Handling
- Function handler: catch upstream Xola errors, return `{ ok: false, error: <message> }` with HTTP 200 (so the browser can render the message). Reserve non-200 only for "request was malformed before we even tried Xola."
- Frontend: any `{ ok: false }` renders an inline error banner above the results area. Don't `alert()`.

### Testing
- The tip-split function is the only place tests are mandatory — it's the heart of the math.
- One Playwright smoke test: load the page, mock the function response, confirm the table renders and CSV downloads.
- Real-data validation (Phase 2 task 2.2) is a manual check against the known-good 2025 crew tips report. That comparison is the V1 success gate, not the automated tests.

## Session Skills

| Skill | When | What |
|-------|------|------|
| `/its-alive` | Session start | Stamp time, ensure `.sessions-worktree/` exists, open per-session file on orphan `sessions` branch, capture transcript, read context, recommend task |
| `/pause-this` | Mid-session break | Build check, commit WIP on the task branch, note pause in session file (on sessions branch) |
| `/restart-this` | Resume from pause | Reload context, continue same session |
| `/kill-this` | **Per task** (DEC-013) | Build check, commit code on task branch, open PR, append `## Task <N>` block to session file. Multiple runs per session — one per task. No time math. |
| `/its-dead` | Session end (once per window) | Stamp `ended:`, tally points, display wall_clock to screen, close session file. No time math, no version bump (those moved to `/retro`). |
| `/start-phase` | Phase boundary (start) | Materialize phase tasks from PROJECT_PLAN.md into Issues with phase:N + points:X labels |
| `/retro` | Phase boundary (end) | Compute per-session wall/dev/review. Aggregate phase velocity. Mark `[x]`, reconcile drift, append to RETROSPECTIVES.md, patch-bump per merged PR + minor-bump at close. |
| `/bump-major` | Breaking change | Manually bump major version. CHANGELOG.md entry + tag. |
| `/promote-staging` | Ship staging to prod | Only applies if `origin/staging` exists. Not used unless we adopt staging-flow later. |
| `/push-seeds` | After workflow improvements | Backport project-side improvements to the seeds templates via @sync-config |
| `/pull-seeds` | After seeds gets new improvements | Pull template changes into this project — schema-version-gated |
| `/read-the-tape` | After a session worth learning from | Audit JSONL transcript, find anti-patterns, propose skill improvements |
| `/doc-consistency-check` | Mid-project, before phase boundaries, or after a session that touched multiple docs | Cross-reference factual claims across `docs/*.md` + root `CLAUDE.md`; flag mismatches + unfilled placeholders. Report-only via @doc-consistency |

**Dev identity:** `~/.claude/devname` (one-line file with your handle). Set once per machine.

**Task model:** PROJECT_PLAN.md is read at planning, written at retro. Untouched mid-phase. Current-phase tasks live as GitHub Issues. The phase ends when its issues close.

## Agent Workflow

| Agent | Model | When | Purpose |
|-------|-------|------|---------|
| @architect | Opus | Before design decisions | Keep architecture coherent |
| @code-review | Sonnet | After every commit (wired into `/kill-this`) | Catch issues early |
| @pm | Sonnet | Start/end of sessions (via skills) | Track progress, flag risks |
| @sync-config | Sonnet | `/push-seeds`, `/pull-seeds` | Template sync |
| @tape-reader | Sonnet | `/read-the-tape` | Transcript audit |
| @doc-consistency | Sonnet | Via `/doc-consistency-check` skill, or ad-hoc | Cross-reference factual claims across project docs; flag mismatches + unfilled placeholders. Report-only |
| @ui-reviewer | — | Not used — type-gated to `webapp` projects only |

## Model Selection

- **Main CC session:** Sonnet by default. Switch to Opus manually when you're stuck on something hard.
- **Agents:** model is set in each agent's frontmatter. Don't override unless the task warrants it.
- **New agents:** default to Sonnet. Add `model: opus` frontmatter only for architecture-level agents.

## PR Workflow (DEC-013 + DEC-014)

- Each **task** gets a branch (`git checkout -b task/X.Y-short-description`). Multiple tasks per session is normal.
- `/kill-this` runs **per task** — opens a PR for the current task branch and appends a `## Task <N>` block to the session file.
- `/its-dead` runs **once per Claude window**, at the end.
- Merge each PR whenever convenient — order doesn't matter; `/retro` reads merge timestamps from GitHub.
- Keep no more than 3 open PRs at once. Prefer 1.
- **PRs to `main` require one approving review (DEC-007).** Eric is the reviewer. Brendan does not self-approve and does not click merge on his own PRs. `/kill-this` opens the PR and stops — it does **not** auto-merge. After Eric approves, either Eric or Brendan can click merge; pushing additional commits after approval dismisses the review and re-requires it.

### Staging vs no-staging (DEC-008)

No staging branch initially. PRs go to `main`. If we adopt staging later, cut `staging` from `main` (`git checkout -b staging main && git push -u origin staging`) — skills auto-detect.

## Versioning (DEC-007)

`package.json` carries the SemVer version, mirrored to a git tag (`vX.Y.Z`) on `main`.

- **Patch:** `/retro` — one bump + CHANGELOG entry per PR merged during the phase window.
- **Minor:** `/retro` — at phase close after all patches land.
- **Major:** `/bump-major` manual.

### `<VersionTag />` component

Not used — no React. If we ever want a visible version in the UI footer, the function response can include `process.env.npm_package_version` and the page can render it.

### CHANGELOG.md

Auto-maintained by the version-bump skills. Don't edit by hand mid-flow.

## Workflow Notes
- **Diagnostic commands** (build, lint, type check, test): run directly.
- **Environment-changing commands** (`npm install`, `netlify deploy`, `git push`): output for the user to run.
- **Never rebase a task branch that already has commits on origin.** Use GitHub's "Update branch" button at merge time.
- **Before starting `netlify dev`:** check whether it's already running on port 8888 (`curl -s -o /dev/null -w "%{http_code}" http://localhost:8888/`). If it returns 200, skip.
- **JSON parsing in Bash:** Prefer `gh ... --jq '...'` (built-in jq via `gh`) or `jq` over `python3 -c "import json,sys; ..."` one-liners. The python invocations trigger per-pattern permission prompts (each unique argument list is a new allowlist entry), while `gh --jq` runs under the existing `Bash(gh ...)` allowance. For non-`gh` JSON, install/use `jq` directly. Reserve python for cases where the data shape genuinely needs control flow.
- **Bug reports:** Create a GitHub issue (`gh issue create`), tag `bug`, add to current or next phase.

## Approval Before Action (all tasks)

For every task — not just bugs — explain the plan and wait for approval before doing anything:
1. State what files you'll create or modify and why
2. Wait for "go", "do it", or equivalent
3. Do not write code, create files, run tests, or execute any commands until approved

## Bug Reports & Questions

When a bug is reported or a question is asked:
1. Explain the cause and your proposed fix
2. Wait for approval before making any changes
3. Do not edit files, run commands, or implement fixes until given the go-ahead

## Scope Discipline

Check `docs/SPEC.md` section "Not V1" before adding anything.

If a task starts feeling bigger than its estimate:
1. Stop and re-estimate
2. Update PROJECT_PLAN.md
3. If it's now a 13, break it down
4. If it's scope creep, flag it and move on

## Tone

Occasional dry humor and sarcasm are welcome. Don't overdo it — one good line beats three forced ones.

## Verbosity

End-of-turn summaries: one or two sentences. What changed, what's next. Stop there.

Do not recap work just watched. Do not restate the task. Do not explain why an obvious step was obvious. The summary exists so the next session can re-enter context — not to demonstrate effort.

Mid-session updates: one sentence per state change. "Found X." "Switching to Y." "Build green." Not a paragraph.

## Cost and Waste

Never minimize cost. Banned phrasings: "essentially zero", "negligible", "only a few cents", "just X dollars", "a rounding error", "not a big deal", "don't worry about it". Any synonym counts.

Treat every cost as real, including small ones. Same rule for compute, API calls, third-party services, dependencies — anything that consumes resources.

Waste of any kind — hours lost, a bad batch, a bricked deploy — is a fact, not a problem to console anyone about. Acknowledge it and move on.
