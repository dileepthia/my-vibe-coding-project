# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

Pulse is a front-end-only React prototype for testing one bet: decision makers bounce less if the dashboard opens on one headline metric, a plain-language takeaway and one recommended action. It replaces today's 12-chart view. The prototype has three tabs: Before (the 12-chart control), After (the guided screen) and Compare (results side by side, with a Ship / Watch / Pivot verdict). Inside After, applying the main action leads to "Your new default view". From there, clicking the headline opens the "Why bounce is 60%" drill-down, which has loading, empty and error states. There is no backend, no API, nothing saved between page loads, and no analytics. Every number is hard-coded mock data in a data.ts file, and the experiment's progress is faked with Day 1/7/14/Fail tabs. The parts worth keeping are the screen flows, the kill-switch rule (a single tested-by-hand function), the drill-down state rules and the feature-folder structure. To make it real you'd need event tracking, assignment of users to control or test, a metrics API and a permission check before actions apply.

## Architecture (plain language)

- **Frontend:** Stack: React 19 + TypeScript + Vite, Tailwind for styling, Recharts for charts, lucide-react for icons, Sonner for toasts. Shell: src/App.tsx holds all the state (which tab and screen is open, which scenario is selected, which actions are applied, click counts, the open dialog) and passes it down to the screens. There's no router and no global state library. Features: src/features/<screen>/ has one folder per screen in the PRD. Each has a display component named after the screen and a data.ts with its mock data and copy. Shared pieces (theme, types, quotes, card, label, sparkline) live in features/shared/. Navigation: handled entirely in App state, so there are no URLs. You can't deep-link to the drill-down, and refreshing the page resets everything.
- **Backend / data:** Backend: none. Data: static TypeScript constants: experiment/data.ts: four scenarios standing in for the daily experiment results. bounce-drilldown/data.ts: bounce by role and by entry page. current-dashboard/data.ts: the 12 widgets on the control view. recommended-actions/data.ts: action titles, descriptions and expected impact. Calculations (pure functions, ready to reuse): evaluateKillSwitch(scenario, clickRate) returns Ship, Watch or Pivot with a reason. confirmedClickRate is a mock formula: seeded % + 2 per confirmed click. goalProgress and buildComparisonRows produce the progress bar and the Compare table. "Fetching": useDrilldownState fakes a 1.5-second load with setTimeout. The States control (Normal, Loading, Empty, Error) forces whichever result you choose.
- **Key flows:** Guided first screen → action: a click opens ActionConfirmationDialog. Apply counts as a confirmed click and shows a toast with Undo. "Not now", the backdrop or the X counts as a dismissal. Simply opening the dialog doesn't count.
Apply primary action: goes to NewDefaultView with the "This is now your default view · Undo" banner. Undo, from the banner or the toast, takes the click back off and returns to the guided screen.
Headline → drill-down: clicking 60% or "See why →" opens BounceDrilldown and runs the load. It then shows normal, empty or error, depending on the States control. Try again re-runs the load. The back link returns to the new default view with the banner as it was.
Experiment: the Day tabs swap the scenario, which recalculates the kill switch, the Compare table and the verdict.
Pivot (Fail case): the main action becomes "Choose a different headline metric". The user picks one of 4 candidates and clicks "Start new test", which only shows a toast.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| Screen flows and copy match the PRD (R1–R15) | solid | _____ |
| Click counting: only confirmed clicks count; dismissals are tracked separately; undo reverses | solid | _____ |

## Risks & assumptions for the team

Assumptions baked into the prototype

Baselines: 60% bounce, 12 charts and 1.3 weekly sessions are accurate and recent. None of these were checked against real analytics.
Bounce definition: "Bounce" means a session under 15s with no interaction. What counts as an interaction is undefined (click? scroll? hover?).
Role data: a user's role (Exec, Sales, Analyst) can be looked up when the page loads.
One headline: everyone in the test group sees the same headline metric.
Permissions: execs have permission to change their default view. If they don't, there's no failure path yet.
Undo: reversing within the session is enough.
Risks

Not enough data: Exec and Sales traffic may be too low for 14 days to show a 9-point drop. The "≥3 pts" and "500 sessions per role" thresholds haven't been sized with a power calculation.
Wrong measure: bounce may be the wrong headline. Execs may want business numbers. The pivot path exists, but it's untested.
Who owns the copy: takeaways and actions are written by hand. Someone has to own them every week, or they'll go stale.
Leaking the experiment UI: the experiment panel must not ship to end users.
Digest side effect: if the weekly digest works, people may visit less. Measuring bounce alone could then misread success.
Mock data mistaken for real: stakeholders may treat the made-up Day 14 "Ship" verdict as evidence. The README and PRD both say it's mocked, so keep repeating that.
What it takes to make it real

Event tracking: dashboard_viewed, first_interaction, session_ended, action_opened/confirmed/dismissed/undone/failed, drilldown_opened/state_shown/retry, all_charts_expanded and pivot_metric_selected. The full list is in the PRD.
Assignment: an experiment flag that splits users 50/50 and keeps each user in the same group.
Metrics API: bounce, time to first interaction and weekly sessions per group per day, plus breakdowns by role and entry page.
Actions: an API to save a user's default view, scheduling for the digest, and a permission check with a real error state.
Routing: proper URLs for the guided screen, the drill-down and the comparison.
Internal access: put the experiment panel and Compare tab behind an internal-only flag.

## How to run it

```
npm install
npm run dev        # starts Vite and prints the local URL
npm run build      # production build → dist/

Viewport: desktop, about 1280px wide.
Where to start: src/App.tsx, then src/features/.
To change mock data: edit the data.ts file in the relevant feature folder.
Kill-switch rule: src/features/experiment/killSwitch.ts. The thresholds are in src/features/shared/theme.ts.
If the page loses its styling: stop and restart npm run dev. This is a known Vite/Tailwind dependency-cache issue.
If you see @/ import errors: vite.config.js is missing the @ import shortcut. Either add resolve.alias: { '@': path.resolve(process.cwd(), './src') } or keep using relative imports.
Demo path (2 minutes):

After → Replace the default view… → Apply → See why.
In the experiment panel, use States to switch between Error, Empty and Normal.
Pick Day 14, open Compare, and you'll see Verdict: Ship.
Pick Fail case, open After, and you'll see the pivot path.
```
