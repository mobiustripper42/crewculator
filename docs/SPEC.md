# Crewculator — Product Specification

## Overview

Crewculator (external name: **Xola Tip Extractor**) is a single-page web tool that converts a manual ~30-minute payroll-day chore into a 30-second one. The operator picks a biweekly pay period, hits Generate, reviews per-crew tip totals on screen, and downloads a Gusto-ready CSV. Tip data is pulled live from the Xola API; guide assignments come from Xola events; the math (even split per guide per event) is done server-side in a Netlify Function.

The tool is internal and twice-monthly. There is no marketing surface, no auth, no user accounts. Spink is the primary operator; Brendan is the backup.

This is also the first project to exercise the Xola API integration end-to-end. The Crew Scheduler project (a much larger initiative) will reuse the same API client patterns — so Phase 0 sandbox exploration here pays forward.

## Philosophy

The tool should feel like a single, sharp instrument. One page. One button. One file out. No nav, no settings page, no marketing copy, no animations. If anything on the page does not exist to get a CSV downloaded faster, it should be deleted.

If something looks off in the numbers, the operator must be able to see *why* on the same screen — per-trip detail, flagged 3+ crew trips, per-person totals — before committing the CSV to Gusto. Trust comes from visibility, not from polish.

## Target Launch

- **V1 target:** First production payroll run using Crewculator instead of the manual process.
- **V1 critical path:** Numbers on screen match the validated 2025 crew tips report for a known pay period. Without that, nothing else matters.

## Stack

- **Frontend:** Static HTML + vanilla JS. No framework. No build step.
- **Backend:** Single Netlify Function (Node 18+, native `fetch`).
- **Hosting:** Standalone Netlify site. Separate from the existing reports site — payroll tool cannot be broken by an unrelated reports deploy.
- **Secrets:** Netlify env vars (`XOLA_API_KEY`, `XOLA_SELLER_ID`, `XOLA_API_BASE`).
- **Database:** Not used — stateless tool, every run pulls fresh.
- **Payments:** Not used.
- **Notifications:** Not used.
- **Auth:** Not used — obscurity-via-bookmark for V1. Net effect is the same as the existing reports site, which has no public link surface either.
- **Testing:** Node test runner for the tip-split function. One Playwright smoke test for the page against a mocked function response. No pgTAP (no DB), no axe-core (no public surface).

## Roles

- **Payroll Operator** — Spink primary, Brendan backup. Runs the tool on payroll day. Selects pay period, reviews per-crew totals on screen, downloads the CSV, uploads it to Gusto during the payroll run. That is the full role. There are no other roles.

## Core Concepts

- **Tip add-on** — a Xola order add-on whose name matches the configured tip-addon list. The price of the add-on is the tip amount for that order.
- **Gratuity** — a separate Xola feature that may also contain tip money. Phase 0 sandbox exploration will confirm whether gratuity is a distinct field on the order/transaction object that we need to sum alongside add-ons.
- **Event** — the Xola event a tipped order belongs to. The event carries the assigned guides (the crew).
- **Guide** — a crew member assigned to one or more events in the pay period. Identified by Xola guide ID; displayed by Xola name; output under Gusto name (may require a mapping if Xola and Gusto disagree on the format).
- **Pay period** — biweekly window. Anchored on a configurable start date; the UI defaults to "the current period" relative to today.
- **Split rule** — for each event, the event's tip pool divides evenly among the guides assigned to that event. Three guides on a $90-tip trip = $30 each. (DEC-002.)
- **3+ crew flag** — any event with three or more guides is flagged on screen so the operator can eyeball whether the split looks right. Doesn't affect the math, just visibility.

## V1 Scope

### Phase 0 — Foundation & Sandbox Confirmation

Repo skeleton (package.json, .gitignore, netlify.toml, .env.example, public/index.html stub, netlify/functions/ dir). Netlify site init + linking. `netlify dev` boots both function + static site locally. Sandbox exploration of `/orders`, `/events`, `/guides` against `sandbox.xola.com` — capture real response JSON for one tipped trip, one 3+ crew trip, one cancelled trip; document actual shapes in `docs/API_NOTES.md`; resolve the four open DEC-TBD entries from DECISIONS.md.

### Phase 1 — MVP

Netlify function `netlify/functions/tips.js` that accepts a date range, pulls orders + events for that window, extracts tip add-ons (and gratuity if Phase 0 confirms a separate source), joins events → guides → orders, splits tips evenly per guide on each event, and returns both per-person totals and per-trip detail. Unit test for the split function.

`public/index.html` — biweekly date picker (default current period), Generate button, results table with totals + 3+ crew flag, client-side CSV download in Gusto format (`Employee Name,Tips`). One Playwright smoke test against a mocked function response.

### Phase 2 — Config, Validation, Deploy

Tip add-on name list (in function code, easy seasonal update). Guide → Gusto name mapping (env var or JSON in repo). Biweekly anchor date. Cancelled-order filter (`items.status: 200`). Error handling for API failures. Real-data validation against a known-good 2025 biweekly window — on-screen numbers must match the validated crew tips report. Deploy to standalone Netlify prod site. Hand off to Brendan for a live test run.

## Not V1

Out of scope. Flag any of these as scope creep before adding.

- **Weighted tip splits** — V1 splits evenly per guide on the event. Lead-guide bonuses, hours-based weighting, role-based splits: all V2.
- **Direct Gusto upload** — V1 produces a CSV the operator uploads manually. No Gusto API integration.
- **Multi-org / multi-account support** — single Xola seller, single Gusto company.
- **Historical pay period reports** — the tool runs against arbitrary date ranges but doesn't store or list past runs. If the operator wants to re-run an old period, they pick the dates again.
- **Auth / login** — no accounts, no SSO. Bookmark-only.
- **Email / SMS notifications** — no notifications at all.
- **Guide self-serve view** — guides do not see their own tips through this tool. They see tips in their paycheck.
- **PDF reports, charts, dashboards** — CSV only. No visualization beyond the on-screen review table.
- **Mobile-first polish** — the tool runs on Spink's desktop. It should not break on mobile, but mobile is not the design target.
- **Public exposure** — the site is internal-use. No SEO, no public-friendly error pages, no marketing surface.
