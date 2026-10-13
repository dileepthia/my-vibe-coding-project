# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: Pulse-Dashboard-Hardening-Chain

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:

1. Add a screen "Your new default view". Match the layout and spacing of the attached
   guided-dashboard screenshot (image.png). Keep the centered ~760px main column with the
   ~220px right-hand experiment panel, 16–20px gaps between cards, 24px card padding, an
   oversized headline number, small uppercase indigo labels, a single indigo accent and
   white cards on a #F7F8FB background.
   - Show it after the user confirms "Replace the default view with one headline metric".
   - Top banner: "This is now your default view · Undo" (Undo returns to the After view
     and removes the confirmed click).
   - Content: the headline card only (60% bounce rate, definition, takeaway, one action),
     then a collapsed "See all 12 charts" link.
   - The 60% headline number is clickable and labelled "See why →".

2. Add a screen "Why bounce is 60%". Match the data density of the attached
   breakdown-table screenshot. Show more data than the guided view, but still group it
   into clearly labelled sections, never a wall of widgets.
   - Header: back link, "Why bounce is 60%", definition line.
   - Breakdown by role: Exec 71%, Sales 68%, Analyst 38%, each with a bar and a change vs last week.
   - Breakdown by entry page: top 5 pages with sessions, bounce % and a sparkline.
   - One "What this means" takeaway: "Decision makers bounce almost twice as often as
     analysts. The first screen isn't built for them."
   - One recommended action: "Send Execs & Sales a weekly one-slide digest".
   - Growth analyst quote: "The number I need is in there, but it takes six clicks and
     three filters to find it."

3. Navigation: write the logic so "Your new default view" links to "Why bounce is 60%".
   - Clicking the headline number or "See why →" opens the drill-down.
   - The back link returns to "Your new default view" and keeps the Undo banner state.
   - The experiment panel stays visible on both screens, and the drill-down's action counts
     toward confirmed clicks.

Build these in order so "Your new default view" is the anchor for "Why bounce is 60%".
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the "Why bounce is 60%" flow
(entered from "Your new default view" via the headline number or "See why →"):

- Use skeleton screens for the role breakdown and entry-page breakdown loading state.
  - Show the skeleton for 1500ms on entry: 3 rows for roles (label, bar, change) and
    5 rows for entry pages (page name, sessions, bounce %, sparkline), matching the
    final row heights so nothing shifts when the data arrives.
  - The header, definition line and back link render immediately; only the lists load.
  - The takeaway and the recommended action stay hidden until both lists have loaded.

- If no data is present, show the empty state:
  "Not enough sessions yet to break this down. We need at least 500 sessions per role. Check back on day 7."
  - Use slate/grey styling, not red. Empty is not a failure.
  - Keep the back link and the headline metric visible.
  - Replace the recommended action with a secondary button: "Back to your default view".

- On fetch failure, trigger the error state:
  "We couldn't load this breakdown. The headline number above is still accurate."
  - Show it inline in place of the lists, with a primary "Try again" button that re-runs
    the loading state, and a secondary "Back to your default view".
  - Do not count the failed load as a confirmed action click.

- Add a "States" control in the experiment panel (Normal · Loading · Empty · Error) so each
  state can be triggered on demand.

Maintain the same design language throughout (white cards on #F7F8FB, a single indigo
accent #4F46E5, small uppercase labels, slate-600 supporting text, red only for
worse/pivot) and tether all behavior strictly to these rules.
```

### Step 3: Refine, one surgical polish
```
The "Why bounce is 60%" screen needs a professional executive-dashboard polish:
calm, scannable and confident, readable in under 5 seconds by a decision maker.

1. Start by listing the 3 biggest gaps in typography and spacing compared to the attached
   guided-dashboard screenshot (image.png). For each gap, give:
   - what's off on "Why bounce is 60%" (e.g. header size, label weight, row height,
     gaps between sections, number alignment),
   - what the reference does instead (e.g. 64px bold headline, 11px uppercase indigo
     labels with wide letter spacing, 24px card padding, 16–20px gaps between cards,
     13px slate-600 supporting text),
   - one line on why it matters to an exec skimming the page.
   Report the list before making any edits.

2. Once you've identified those, resize the headers and update the role and entry-page
   breakdown rows to match:
   - Headers: page title, section labels ("By role", "By entry page") and the
     "What this means" label use the same type sizes, weights and letter spacing as the
     reference.
   - Breakdown rows: consistent row height (~44px), bounce % right-aligned in bold with
     tabular (equal-width) numbers, bars at one fixed width, change values in a consistent
     red/green, 1px slate-100 dividers, and padding that matches the reference cards.
   - Keep the skeleton, empty and error states visually consistent with the updated rows.

Don't change anything else in the project or touch the underlying logic: no changes to
copy, data, navigation, state rules, the experiment panel or any other screen.
```

## Reusable techniques learned

- _____
- _____

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

_____
