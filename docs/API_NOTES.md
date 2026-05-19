# Crewculator — Xola API Notes

This document captures the **actual** response shapes from the Xola API for the three endpoints Crewculator uses: `/orders`, `/events`, `/guides`. The handoff doc described what the docs *say*; this file documents what the API *actually returns*.

**Status:** Empty stub. Populated by **Phase 0 task 0.2** (Sandbox API Exploration). Once filled in, the four `DEC-TBD` entries in `DECISIONS.md` get promoted to numbered `DEC-NNN` decisions.

## What goes in here (during Phase 0 task 0.2)

For each endpoint, capture:

1. **Request shape** — full URL with all params, headers used
2. **Response sample** — real JSON from the sandbox, trimmed to one representative item per list. Replace any sensitive fields (real names, emails) with realistic placeholders before committing.
3. **Field map** — which fields we actually read for tip calculation, with the JSON path (e.g., `items[].addOns[].price`)
4. **Edge cases observed** — what an order with no add-ons looks like, what a cancelled order looks like, what a multi-guide event looks like

## Endpoints to document

### `GET /orders`
- Resolves DEC-TBD: **Xola order add-on shape on returned orders**
- Resolves DEC-TBD: **Date-range query on `/orders`**
- Resolves DEC-TBD: **Gratuity feature — separate source from add-ons?**

### `GET /events`
- Resolves DEC-TBD: **Event → guides relationship**

### `GET /guides`
- Only document if the `/events` response turns out to need a separate guide lookup (i.e., events return guide IDs only, not full guide objects).

## Cases to capture

At minimum, pull and save one representative response for each of these:
- A normal tipped trip (one guide, one tip add-on)
- A 3+ crew trip (multiple guides on the same event, one tip pool)
- A cancelled trip (so we can confirm the `items.status: 200` filter behaves)
- A trip with gratuity but no tip add-on (if it exists — confirms the gratuity field path)
