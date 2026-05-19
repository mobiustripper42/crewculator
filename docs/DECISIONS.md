# Crewculator — Architectural Decisions

Decisions are numbered DEC-NNN. "DEC-TBD" means the decision is flagged but unresolved — consult @architect (or the relevant Phase 0 sandbox exploration task) before building anything that depends on it.

---

## DEC-001: Static HTML + Netlify Function (no Next.js/React/Supabase)

**Decision:** One static `index.html` served from a Netlify site, plus one Netlify Function for the Xola API call. No framework, no build step, no database.

**Why:** The tool fires twice a month, has one page, has no state to persist, has no auth, and has one user (with a backup). Adopting Next.js + Supabase + shadcn would add ~90% of the build overhead for ~0% of the user-visible benefit. Static HTML + vanilla JS is the smallest correct shape. The Netlify Function is the only thing that needs to exist server-side — to hold the Xola API key out of the browser.

**Tradeoff:** No reusable component library, no type checking, no SSR. None of those matter for one page that renders a table. If the page ever grows past ~300 lines or develops real interaction state, revisit — but if it does, that's also a sign scope has crept.

**Revisit if:** The page grows past ~300 lines, or we add a second page, or we add auth.

---

## DEC-002: Even split per guide on each event

**Decision:** For each event in the pay period, the event's total tip pool divides evenly among the guides assigned to that event. No weighting by role, hours, or lead-guide status.

**Why:** Matches how Spink does the math by hand today. Every other split rule (lead-guide bonus, hours-weighted, role-weighted) would require a separate data source the tool does not currently have. Adding any of them in V1 would require a calibration step against historical data the operator does not want to do during a payroll run.

**Tradeoff:** If a lead guide on a multi-guide trip "should" get more, V1 doesn't model it. If that becomes a real issue, V2 can add a `guide_role` field and a weight table. The current rule is also the rule the operator will sanity-check against on-screen, so any divergence is immediately visible.

**Revisit if:** Operator flags a real split that the even rule got wrong in a way the manual process would have caught.

---

## DEC-003: Tip add-on name list lives in function code, updated seasonally

**Decision:** The list of add-on product names that count as tips is a JavaScript constant in `netlify/functions/tips.js` (or a sibling config file the function imports). Not an env var, not a database table.

**Why:** Add-on names change between seasons (Xola product catalog gets refreshed). When that happens, the operator edits the list, commits, and Netlify auto-deploys — the change is reviewable in a PR and visible in git history. An env var would hide the list in the Netlify dashboard with no version control; a database table would require a database. The list will have ~3–10 entries; a code constant is the right size.

**Tradeoff:** Non-engineers can't edit it without a code change. That's fine — Spink doesn't need to edit it; the engineer doing the seasonal refresh does.

**Revisit if:** The list grows past ~20 entries, or it needs to change more often than seasonally.

---

## DEC-004: Biweekly pay period anchored on a configurable start date

**Decision:** The UI defaults the pay period to "the current biweekly window," computed from a configurable anchor start date (`PAY_PERIOD_ANCHOR` env var, ISO 8601). The operator can override the dates if needed.

**Why:** Biweekly periods drift relative to month-end; without an anchor, "the current period" is ambiguous. The anchor is set once, then the math is deterministic. Operator override exists for the inevitable case where the operator wants to re-run a prior period to compare against the manual report.

**Tradeoff:** A wrong anchor produces wrong defaults silently. Phase 2 task 2.2 (real-data validation) catches this on the first run.

**Revisit if:** Payroll cadence ever changes (weekly, semi-monthly, monthly).

---

## DEC-005: Standalone Netlify site (not on existing reports site)

**Decision:** Crewculator deploys to its own Netlify site, not as a sub-route of the existing reports site.

**Why:** Payroll tooling has a different blast radius than reporting. If a reports-site deploy breaks the build, payroll day cannot be affected. Separate env vars, separate deploy history, separate rollback button. Netlify free tier supports many sites; there is no cost to splitting.

**Tradeoff:** One more Netlify dashboard to remember. Mitigated by bookmarking and by the fact that the tool is touched twice a month.

**Revisit if:** We ever build a third Xola-API-touching tool — at that point a single "ops" site with multiple routes may be worth the consolidation.

---

## DEC-006: No automated tests beyond the split function + one smoke test

**Decision:** The tip-split function gets a Node-test-runner unit test (the math is the risky part). The page gets one Playwright smoke test against a mocked function response. Nothing else.

**Why:** Real-data validation against the known-good 2025 crew tips report (Phase 2 task 2.2) is the V1 success gate, not the automated test suite. For a twice-monthly internal tool with one page, automated regression coverage past the split-function math is friction without payoff. The operator catches drift the same way they would have caught it manually — by reviewing numbers on screen before downloading.

**Tradeoff:** A page-rendering regression could ship if the smoke test doesn't catch it. Mitigated by `/kill-this` running a quick `netlify dev` + manual smoke before merge, and by the operator's review step.

**Revisit if:** The page grows interactive surface area (multiple buttons, edits, undo, etc.) past what one smoke test can cover.

---

## DEC-007: Repo is public; PRs to `main` require Eric's review

**Decision:** The GitHub repo is **public**, and a GitHub Ruleset on `refs/heads/main` requires one approving review before any PR can merge. Brendan is the dev; Eric is the reviewer. Brendan does not self-approve and does not click merge on his own PRs. `/kill-this` opens the PR and stops there.

**Why:**
1. **Brendan is new to dev work** (see "Working with Brendan" in `CLAUDE.md`). Eric owns payroll-day correctness, so Eric reviews every change that lands.
2. **GitHub charges for branch protection on private repos** (Pro tier required). Rulesets give equivalent enforcement on public repos for free. The codebase has nothing sensitive — first names only, no business name, no secrets in code (env vars live in Netlify). The visibility tradeoff is minimal; the enforcement guarantee is real.
3. **Stale-review dismissal** is on — if Brendan pushes more commits after Eric approves, the approval is dismissed and re-review is needed. Prevents "approval drift" where an old approval blesses unreviewed code.

**Tradeoff:** Eric becomes a bottleneck. PR turnaround depends on Eric's availability. The seeds default ("self-approve unless explicitly needed") is overridden here because the dev/reviewer split is the whole point.

**Override of seeds default:** The seeds CLAUDE.md says "Self-approve unless a stakeholder review is explicitly needed." This project explicitly needs it.

**Revisit if:** Brendan becomes comfortable enough that Eric's review is rubber-stamping every PR, or if Eric's review latency becomes a real blocker. Either way, the right move is to relax the rule (drop required reviews to 0), not to bypass it.

---

## DEC-TBD: Xola order add-on shape on returned orders

**Question:** The Xola API docs describe add-on *definitions* on Experiences, but the shape of `items[].addOns[]` on a *returned order* is undocumented. We need: the field name for tip amount (`price`? `amount`? `unitPrice` * `quantity`?), whether quantity matters, and whether the add-on name is on the line item or requires a separate join.

**Resolution path:** Phase 0 task 0.2 — hit `sandbox.xola.com/api/orders` with a real tipped order, capture the JSON, and document in `docs/API_NOTES.md`. Then promote to a numbered DEC entry.

**Blocks:** Phase 1 task 1.1 (tip extraction).

---

## DEC-TBD: Date-range query on `/orders`

**Question:** The Xola `items.arrival` filter accepts a single date. Can we use `items.arrival[gte]` / `items.arrival[lte]` via the [Xola query language](https://developers.xola.com/reference/query-language), or do we need to loop day-by-day across the pay period?

**Resolution path:** Phase 0 task 0.2 — attempt a range query against the sandbox; if it works, document the exact syntax. If not, document the day-loop pagination approach (14 days × N orders/day with pagination per day).

**Blocks:** Phase 1 task 1.1 (function shape — single call vs loop).

---

## DEC-TBD: Event → guides relationship

**Question:** Does the `/events` response include guide *objects* (id + name + maybe more) inline, or only guide *IDs* that require a separate `/guides` lookup to resolve to names?

**Resolution path:** Phase 0 task 0.2 — capture a real `/events` response and inspect `guides[]`. If only IDs, add a Guide lookup-and-cache step to the function (one-time fetch per cold start; in-memory cache for the lifetime of the function invocation).

**Blocks:** Phase 1 task 1.1 (whether the function makes 2 or 3 API calls per run).

---

## DEC-TBD: Gratuity feature — separate source from add-ons?

**Question:** The handoff notes that tips come *mostly* from add-ons but also some from Xola's Gratuity feature. Is gratuity a distinct field on the order object, or on a transactions endpoint? If distinct, the function must sum both sources per order.

**Resolution path:** Phase 0 task 0.2 — inspect a known-good order that has both an add-on tip and a gratuity tip. Document the field path. If the gratuity is on `/transactions`, add a third endpoint to the function flow.

**Blocks:** Phase 1 task 1.1 (whether the function reads one source or two).

---

## DEC-TBD: Guide name normalization (Xola vs Gusto)

**Question:** Xola may store a guide as "Eric Stoffer" while Gusto has "Stoffer, Eric" (or any other variation). The CSV must use Gusto's exact format for Smart Import to auto-match.

**Resolution path:** Phase 2 task 2.1 — once we have a list of real guide names from the sandbox, compare against the Gusto employee list and decide between (a) a mapping file in the repo, (b) a normalization function (last-comma-first → first-last), or (c) requiring Xola names to be edited to match Gusto. Most likely (a).

**Blocks:** Phase 2 task 2.1 (config), not Phase 1 — the Phase 1 MVP can ship with raw Xola names and Phase 2 reformats.

---
