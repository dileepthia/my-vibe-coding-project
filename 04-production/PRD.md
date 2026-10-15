# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

60% bounce rate. Sessions under 15s with no interaction. (This is a baseline from the brief, not measured by the prototype.)
12 charts on the default landing view compete for attention.
1.3 avg. weekly sessions per active user. People rarely come back.

The hypothesis this PRD bets on:

A first screen that leads with one headline metric, a plain-language takeaway and one recommended action, followed by priority-ranked metrics with their own actions, will let Executives and Sales get what they need within seconds instead of bouncing.

## Users & jobs

- **Primary user:** Executives and Sales leaders. Decision makers who open the dashboard briefly, often from a link or a weekly habit, to decide something. They don't analyze.
- **Job to be done:** When I open the dashboard, I want to see the one number that matters, what it means and what to do next, so I can make a decision in seconds without exporting to a spreadsheet or asking an analyst.

## Scope

- **In:** Guided first screen: headline metric, definition, takeaway, one primary recommended action, and an evidence quote.
Priority-ranked secondary metrics, each with a takeaway and its own action (charts on the default view; weekly sessions per user).
Recommended-action flow: confirm dialog (what changes, expected impact), apply, undo, dismiss.
"Your new default view": the result after applying the primary action. Headline only, with a collapsed "See all 12 charts" link.
"Why bounce is 60%" drill-down: breakdown by role and by entry page, takeaway, action. Includes loading, empty and error states.
A/B experiment of the guided view against the current 12-chart view, with per-group bounce, time to first interaction and confirmed click rate.
Kill-switch rule evaluated and shown to internal stakeholders (Ship / Watch / Pivot).
Pivot path: choose a different headline metric and restart the test.
- **Out (explicitly):** Building the weekly digest. Only the action and click tracking are in scope. Scheduling and sending the email are not.
Headlines per user or role. Everyone in the test group sees the same headline.
Cutting the default view to 3 charts. This action exists in the prototype but has no result screen. Track the click only.
Filters, custom date ranges, settings, permissions UI.
Mobile layouts. The prototype is desktop-only, around 1280px.
Showing the experiment panel to end users. It's for internal and stakeholder use only. (The prototype shows it to everyone for demo purposes.)
Generating takeaway text automatically. Takeaways are written by people in v1.

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | Guided first screen leads with one headline metric | Must | Headline value, label and definition are visible without scrolling at 1280×800. The value is the largest text on screen. |
| 2 | Plain-language takeaway under the headline | Must | One takeaway of 25 words or fewer, shown in a "What this means" panel directly under the headline. |
| 3 | 	One primary recommended action	| Must	| Exactly one primary button on the first screen. It opens a confirmation showing what changes and the expected impact.
| 4 | 	Confirm / dismiss / undo for actions	| Must	|	Apply logs a confirmed click and shows a toast with Undo. "Not now" or closing logs a dismissal. Undo reverses the change and the click count. Opening the dialog alone is not counted.
| 5 | 	Priority-ranked secondary metrics with actions	| Must	|	Ranks 2 and 3 are shown in order, labelled "ranked by impact on bounce". Each has a value, a 7-week sparkline, a takeaway, a quote and one secondary action.
| 6 | 	"Your new default view" after applying the primary action	| Should |	It appears straight after Apply, with the "This is now your default view · Undo" banner. Only the headline card is shown. "See all 12 charts" expands the full grid.
| 7 | 	Headline drill-down	| Should |	Clicking the headline value or "See why →" opens "Why bounce is 60%", showing bounce by role and the top 5 entry pages with sessions, bounce and trend. The back link returns with the banner state kept.
| 8 | 	Drill-down loading state	| Should |	Placeholder rows (3 role, 5 page) appear for up to about 1.5s and match the final 44px row height, so the layout doesn't move. The header and back link render immediately. The takeaway and action stay hidden until data loads.
| 9 | 	Drill-down empty state	| Should |	Shown when any role has fewer than 500 sessions. Exact copy: "Not enough sessions yet to break this down. We need at least 500 sessions per role. Check back on day 7." Grey styling, no red. A secondary back button replaces the action.
| 10 | 	Drill-down error state	| Should |	Exact copy: "We couldn't load this breakdown. The headline number above is still accurate." Has "Try again" (reloads) and "Back to your default view". A failed load never counts as a click.
| 11 | 	A/B split of guided vs. current view	| Must	|	Execs and Sales are randomly assigned 50/50 and stay in the same group. Control sees the current 12-chart view unchanged.
| 12 | 	Kill-switch evaluation	| Must	|	Day 14 verdict: Pivot if guided bounce is within 3 pts of control. Ship if guided bounce is 51% or lower and confirmed clicks are 10% or higher. Otherwise Watch. The rule and reason are shown on the internal experiment view.
| 13 | 	Comparison summary for stakeholders	| Should |	A table comparing control and guided on bounce, time to first interaction, action clicks and weekly sessions, plus a one-slide verdict summary.
| 14 | 	Pivot path	| Could |	When the verdict is Pivot, the primary action becomes "Choose a different headline metric", with 4 candidates. Picking one restarts the test from Day 0.
| 15 | Visual consistency	| Should |	Same design language on every screen: white cards on #F7F8FB, indigo #4F46E5 as the only accent, 11px uppercase labels, red only for things getting worse or a pivot.





## Data & events

_What gets stored, what gets tracked._

Data	Source (assumed)	Used for	Prototype
Session start/end, first interaction time	Product analytics	Bounce rate, time to first interaction	Hard-coded (18,420 sessions; 60%; 14s)
User role (Exec / Sales / Analyst)	User directory or CRM	Test eligibility, role breakdown	Hard-coded (71 / 68 / 38%)
Entry page	Router or referrer	Entry-page breakdown	Hard-coded (5 pages)
Weekly sessions per active user	Product analytics	Secondary metric	Hard-coded (1.3)
Experiment group	Experiment platform	Control vs. guided split	Simulated by the Before/After tabs
Daily results	Experiment platform	Kill switch, verdict	Simulated by the Day 1 / 7 / 14 / Fail tabs

## Open questions

Is the baseline real? Where do 60% / 12 / 1.3 come from, and over what period? Is bounce already being measured for Execs and Sales specifically?
What counts as an "interaction"? Does scrolling count? Hovering a chart? This changes the bounce rate a lot.
Is bounce the right headline? It measures the dashboard's health, but execs may care about business numbers (pipeline, win rate). The pivot candidates hint at this. Should we test a business metric as the headline from day one?
Sample size and test length. Is 14 days enough to detect a 9-point drop given Exec/Sales traffic? The "500 sessions per role" threshold also needs a power calculation.
Who writes the takeaway and action each week? If people write them, who owns that? If they're generated, what's the quality bar?
Who can apply actions? Execs may not have edit rights. We need the error and "request access" path, which the prototype doesn't have.
Is "Replace the default view" for one user or everyone? Does applying it change the view only for that user, or the whole workspace?
Who sees the experiment and kill-switch view? It's currently visible to everyone for the demo. In production it should be internal only.
Does the digest count toward the hypothesis? If the digest reduces visits, is lower bounce still the right measure?
Is undo within the session enough? Or do users need a persistent "restore previous view" setting?
Where do the user quotes come from? How many interviews and which roles? Are three quotes representative?
Usability check before launch: run a 5-second test with 5–8 Execs and Sales leaders on the guided screen and the drill-down. Pass: they can say who bounces most and what to do.
