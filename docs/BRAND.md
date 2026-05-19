# Crewculator — Brand Direction

## Name

**Crewculator** (internal repo + project name).
**Xola Tip Extractor** (external/user-facing label on the page title and the README intro).

Spink and Brendan refer to it by either name; "Xola Tip Extractor" is what shows up in a browser tab. "Crewculator" is what shows up in commits and PRs.

## Tagline

None needed. It's an internal payroll tool used twice a month. A tagline would be costume jewelry.

## Philosophy

The tool exists to remove 30 minutes of manual spreadsheet work from a payroll day that already has enough friction. Every design choice should serve the operator's ability to do one specific thing — pick a pay period, see correct numbers, download a Gusto-shaped CSV — and do it in under two minutes.

The operator must trust the numbers before they trust the CSV. So the on-screen review is not garnish; it is the product. Everything else is plumbing.

In practice this means:
- The page is one screen. No nav. No tabs. No login.
- The results table shows per-guide totals *and* per-trip detail on the same view — so if a number looks off, the operator can see why without re-running anything.
- 3+ crew trips get a visual flag because that's the case the operator is most likely to want to eyeball.
- Errors render inline above the table, not as `alert()` or as a console-log buried where no one looks.

## Voice

Practical. No marketing tone. No exclamation points. No emojis. Labels are nouns; buttons are verbs.

Examples of the voice:
- Button: **Generate** (not "Calculate my tips!" or "Run report")
- Empty state: "Pick a pay period and hit Generate." (not "Welcome to Crewculator!")
- Error: "Xola request failed: 503. Try again in a minute, or check Netlify function logs." (not "Oops! Something went wrong 😬")

What it should NOT sound like: a SaaS landing page, a chat assistant, a customer-facing product. There is no customer here. There is one operator who already knows what the tool does — give them the information and get out of the way.

## Visual Direction

- **Style:** Minimal. Single column on desktop. White background, near-black text, one accent color for the Generate button and the 3+ crew flag.
- **Default mode:** Light only. No dark-mode toggle (would be costume jewelry; the operator runs it in daylight on a desktop).
- **Font:** System font stack (`system-ui, -apple-system, Segoe UI, sans-serif`). No web font load — the page should render before the network finishes thinking about it.
- **Border radius:** `4px` everywhere. One radius value.
- **Shadows:** None. Borders only where they help readability of the table.
- **Color approach:** Hand-picked CSS variables in the `<style>` block. No Tailwind, no shadcn. A single accent color for the Generate button, a single warning color for the 3+ crew flag and error banner.
- **Tables:** Striped rows, right-aligned numeric columns, totals row in bold at the bottom. Per-trip detail collapsed under each guide by default — click to expand if a number needs investigating.

## Anti-patterns

What to explicitly avoid:
- **No marketing language.** No "powerful", "seamless", "intuitive", or any other adjective that would appear on a landing page.
- **No animations.** No fade-ins, no spinners except a basic "Loading…" text label during the Generate request.
- **No hero, no header art, no logo lockup.** The page title is a plain `<h1>`.
- **No icons for icons' sake.** A "download" icon next to a button labeled "Download CSV" is redundant. Skip it.
- **No success toasts.** When the CSV downloads, the operator sees the browser's download notification — that's the confirmation. We don't need to stack our own on top.
- **No gamification.** No counters, no badges, no "you saved N hours this year." It's payroll.

## Priority

Function over form, every time. Polish is not a phase here — there is no polish phase. If something looks unpolished but works, that's the final state.
