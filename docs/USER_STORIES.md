# Crewculator — User Stories

Story IDs use the prefix `OP-` for the Payroll Operator role. There is only one role; sub-sections group stories by phase of the payroll-day workflow. Cross-reference with `SPEC.md` and `PROJECT_PLAN.md`.

---

## Payroll Operator (Spink primary, Brendan backup)

### Generating the report
- **OP-1:** As the operator, I open the bookmarked page and the current biweekly pay period is already selected, so I don't have to remember today's date relative to the pay calendar.
- **OP-2:** As the operator, I can override the date range when I need to re-run an old period (e.g. to compare against the manual report), so I'm not locked into "current period only."
- **OP-3:** As the operator, I hit Generate once and see a table of per-guide tip totals within a few seconds, so I'm not waiting through a multi-minute spinner during a payroll run.

### Reviewing the numbers
- **OP-4:** As the operator, I see per-guide totals and per-trip detail on the same screen, so if a total looks off I can trace it to the trips that produced it without re-running anything.
- **OP-5:** As the operator, I see a visual flag on any event with 3+ guides, so I can eyeball whether the even-split feels right before I commit the CSV.
- **OP-6:** As the operator, when the function fails to reach Xola or returns a partial result, I see a clear inline error message above the table — not a blank page, not an `alert()`, not a "console.log it and hope" — so I know to retry or escalate.

### Producing the CSV
- **OP-7:** As the operator, I click Download CSV and get a file named for the pay period (e.g. `crewculator-2026-05-04_2026-05-17.csv`) with exactly two columns (`Employee Name`, `Tips`), so it drops straight into Gusto Smart Import with no editing.
- **OP-8:** As the operator, the CSV uses each guide's Gusto-formatted name (not their raw Xola name), so Gusto auto-matches every row without a manual mapping step during the payroll run.

### Backup workflow
- **OP-9:** As the backup operator (Brendan), I can use the same bookmarked page with no extra training when Spink is out, because the workflow is "pick dates → Generate → review → Download" with no hidden state or environment toggles.
