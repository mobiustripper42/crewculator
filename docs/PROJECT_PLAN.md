# Crewculator — Project Plan

**Start date:** 2026-05-18
**V1 target:** First production payroll run done via Crewculator instead of the manual export/cross-reference process.
**Critical path:** On-screen numbers match the validated 2025 crew tips report for a known biweekly window. Without that, the CSV is untrusted; without the CSV, the tool has no value.

**Planner:** Eric (2026-05-18 planning session).
**Implementing dev:** Brendan. All Phase 0–2 tasks are his to execute, re-estimate, or push back on. He inherits the standing disagreements table empty — anything he disagrees with during the build goes there.

---

## Estimation Method

Fibonacci scale (2, 3, 5, 8, 13). See `VELOCITY_AND_POKER_GUIDE.md` for definitions.
All estimates from planning poker between Eric and Claude during the 2026-05-18 planning session.
Disagreements logged in the Standing Disagreements table at the bottom.
Tests are baked into every task estimate — no separate testing tasks.

**Velocity baseline:** Not yet established. Will update after first 5 sessions.

---

## Phase 0 — Foundation & Sandbox Confirmation

Everything needed to build safely + resolve the open API questions from the handoff before committing the MVP to a shape. No user-facing surface yet.

| # | Task | Effort | Notes |
|---|------|--------|-------|
| 0.1 | Repo skeleton: `package.json`, `.gitignore`, `netlify.toml`, `.env.example`, `public/index.html` stub, `netlify/functions/` dir; Netlify site init + linking; `netlify dev` boots both function + static site locally | 3 | The doc files (CLAUDE.md, SPEC.md, etc.) are already in place from the 2026-05-18 planning session — they don't count against this task |
| 0.2 | Sandbox API exploration — hit `/orders`, `/events`, `/guides` against `sandbox.xola.com`; capture real response JSON for one tipped trip, one 3+ crew trip, one cancelled trip; document actual shapes in `docs/API_NOTES.md`; promote the four DEC-TBD entries to numbered DEC entries | 5 | Resolves DEC-TBD: addon shape, date-range query, event→guides, gratuity field |

**Phase 0 total: 8 pts**

**Ejection point:** `netlify dev` runs locally, env vars resolve, `docs/API_NOTES.md` documents exactly what the three endpoints return for our cases, and `DECISIONS.md` has four new numbered DEC entries replacing the four DEC-TBDs.

**Demo:** `netlify dev` → curl the function locally with a dummy date range → see a JSON response with real sandbox data. Open `docs/API_NOTES.md` and walk through the captured shapes.

---

## Phase 1 — MVP

End-to-end against sandbox: pick dates, hit Generate, see per-guide totals, download a Gusto-shaped CSV. Not yet production-validated.

| # | Task | Effort | Notes |
|---|------|--------|-------|
| 1.1a | Xola API client (`netlify/functions/lib/xola.js`) — fetch `/orders`, `/events`, and `/guides` (if Phase 0 confirms a lookup is needed) for a given date range. Pagination, date-range query per Phase 0 findings, env-var auth (`XOLA_API_KEY`, `XOLA_SELLER_ID`, `XOLA_API_BASE`), normalized return shape. Tests with mocked `fetch` (`tests/xola.test.js`). Pure I/O — no tip logic. | 3 | Split out of original 1.1. Lands on its own PR; 1.1b imports it. |
| 1.1b | Tip extraction + split + handler — pure split function (`netlify/functions/lib/split.js`, per-event even split per DEC-002), per-person aggregation, per-trip detail, 3+ crew flag computation. Netlify handler (`netlify/functions/tips.js`) that wires client → split → JSON response. Unit test for the split function (`tests/split.test.js`) — the math is the mandatory test per DEC-006. | 5 | Split out of original 1.1. Imports the 1.1a client. |
| 1.2 | `public/index.html` — biweekly date picker (default current period via DEC-004 anchor), Generate button, results table with per-guide totals + collapsible per-trip detail + 3+ crew flag, client-side CSV download in Gusto format (`Employee Name,Tips`). Inline error banner above table. One Playwright smoke test against a mocked function response. | 5 | Vanilla JS, no framework. CSV is built client-side from the JSON response, no library. |

**Phase 1 total: 13 pts**

**Ejection point:** Open the page locally (`netlify dev`), pick a date range covering known sandbox data, hit Generate, see numbers on screen, download the CSV. The math passes its unit test; the page passes its smoke test. Not yet validated against real 2025 production data.

**Demo:** Live walkthrough against sandbox. Spink eyeballs the on-screen numbers — they don't have to be production-correct yet, they just have to look reasonable.

---

## Phase 2 — Config, Validation, Deploy

Production. Real-data validation is the gate.

| # | Task | Effort | Notes |
|---|------|--------|-------|
| 2.1 | Config: tip add-on name list (in function code per DEC-003); guide → Gusto name mapping (resolves the last DEC-TBD); biweekly anchor date as env var (DEC-004); cancelled-order filter (`items.status: 200`); error handling for Xola API failures (timeout, 5xx, malformed response). | 3 | |
| 2.2 | Real-data validation — switch from sandbox to production Xola; run against a known-good 2025 biweekly window; compare on-screen numbers AND downloaded CSV row-for-row to the validated crew tips report. Fix any discrepancies before proceeding. | 3 | This is the V1 success gate per SPEC.md critical path. |
| 2.3 | Deploy to standalone Netlify prod site (DEC-005); set prod env vars (`XOLA_API_KEY`, `XOLA_SELLER_ID`, `XOLA_API_BASE`, `PAY_PERIOD_ANCHOR`); smoke test on the deployed URL; bookmark; hand to Brendan for a live test run on the next payroll day. | 2 | The user (not Claude) runs `netlify deploy --prod`. |

**Phase 2 total: 8 pts**

**Ejection point:** Spink uses Crewculator on the next real payroll day with zero help, and the resulting Gusto Smart Import matches the validated crew tips report.

---

## Total project effort

| Phase | Pts |
|-------|-----|
| Phase 0 | 8 |
| Phase 1 | 13 |
| Phase 2 | 8 |
| **Total** | **29** |

Handoff estimated 25 pts across 6 sprints. The 4-pt delta is unit + smoke tests baked into 1.1 and 1.2 per the seeds convention.

---

## Velocity Table

Updated at end of each phase by `/retro`. Used by @pm to project remaining time.

| Phase | Actual Hours | Effort Points | Hrs/Pt | Notes |
|-------|--------------|---------------|--------|-------|
| 0 | — | 8 | — | |
| 1 | — | 13 | — | |
| 2 | — | 8 | — | |

**Lifetime velocity:** — hrs/pt (not yet established)

---

## Estimation Poker — Standing Disagreements

Unresolved estimate disagreements. Revisit when the task starts.

| Task | Claude says | Eric says | Question |
|------|-------------|-----------|----------|
| _(none yet)_ | — | — | — |

---

## Phase Boundary Checklist

At the end of every phase:
1. Unit tests green (`node --test`)
2. Smoke test green (`npx playwright test --project=desktop`)
3. @pm phase retrospective — velocity check, timeline update
4. Write retrospective entry in `docs/RETROSPECTIVES.md` (velocity, scope changes, process notes, forecast update)
5. Review docs against intent before starting the next phase

---

## Cuttable Tasks (if behind)

Tasks that can be deferred to V2 without breaking core V1 functionality. Reference before any scope cut conversation.

| Task | Why it's cuttable | Defer to |
|------|-------------------|---------|
| Per-trip detail in 1.1b return shape | Per-person totals alone are enough to populate the CSV; per-trip detail is review-aid only. Drop it if 1.1b runs long | V2 |
| Per-trip detail collapse-to-expand UI in 1.2 | The flat list still works; collapse is ergonomics, not correctness | V2 |
| 3+ crew flag styling in 1.2 | The flag's job is "draw the eye." A bare row label like `[3+]` would do — fancier styling is optional | V2 |
| Guide → Gusto name mapping in 2.1 | If Gusto Smart Import can match raw Xola names well enough on the validation run, defer the mapping table | V2 |
| Error handling polish in 2.1 | A bare "Request failed" message is enough for V1 if a richer one isn't built — the operator can read function logs | V2 |
| Playwright smoke test in 1.2 | The unit test on the split function plus the manual validation in 2.2 is enough for first ship. Smoke test is regression insurance, not first-ship gate | V2 |
