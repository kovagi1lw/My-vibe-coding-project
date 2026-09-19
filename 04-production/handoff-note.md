# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

This prototype answers the question: "What if the analytics landing page told you what to do instead of drowning you in charts?" It is a guided decision screen for trial workspaces that replaces the usual 12-chart analytics overview with one headline metric (7-day trial activation), a plain-language takeaway, and one recommended action. The goal is to stop decision-makers from bouncing and get them to act.
 
The prototype includes four connected screens:
 
1. **Guided overview**: An interactive headline-metric chart and a drill-down comparing key vs. supporting metrics.
2. **Experiment readout**: Shows whether the guided screen is actually improving bounce rate and action clicks.
3. **Recommendation screen**: Turns the insight into a concrete next step.
4. **Follow-up screen**: Where the user records what they will do next.
 
Live anonymous session and event tracking powers the readout, while all product copy, baseline figures, chart data, and recommendations are mocked.

## Architecture (plain language)

### Frontend
 
* **Stack**: TanStack Start v1 + React 19, Vite 8, Tailwind v4 via `src/styles.css` (semantic design tokens only; no hardcoded colors in components).
* **Routing**: File-based routes in `src/routes/`: `index.tsx` (`/`), `recommendation.tsx` (`/recommendation`), `follow-up.tsx` (`/follow-up`), plus `api/public/dashboard-activity.ts`. Route files are thin wrappers setting head metadata and rendering a screen component.
* **Feature Structure**:
   * `src/features/guided-overview/` — `GuidedOverview.tsx`, `parts/HeadlineMetricChart.tsx`, `parts/HeadlineMetricDrilldown.tsx`
   * `src/features/experiment-readout/` — `ExperimentReadout.tsx`, `parts/` (ReadoutMetric, ReadoutCheckpoint, BaselineStat, BeforeStateQuote)
   * `src/features/recommendation/` — `RecommendationScreen.tsx`, `parts/RecommendationFigure.tsx`
   * `src/features/follow-up/` — `FollowUpScreen.tsx`, `parts/FollowUpField.tsx`, `follow-up.store.ts`
   * `src/features/navigation/` — `AppShell.tsx` (compact left rail + company bar + top nav), `parts/` (RailLink, TopNavLink, ViewTab)
   * `src/features/activity/` — tracking, server aggregation, shared types
* **Data separation**: Every screen has a `*.data.ts` module holding copy, figures, thresholds, options, quotes, and nav definitions. Components render; they do not embed content.
 
### Backend and data
 
* **Database**: Lovable Cloud (PostgreSQL). Migration applied: `drizzle/migrations/0000_create_anonymous_activity_measurement.sql` (and core prototype tables in `0001`).
   * `dashboard_sessions`: One row per anonymous browser session: `duration_seconds`, `meaningful_interaction`, `recommended_action_clicked`, `follow_up_created`, `started_at`, `last_seen_at`, `ended_at`. RLS enabled, accessible via `service_role`.
   * `dashboard_events`: Append-only event log, foreign key to sessions, event name constrained by a check list, `event_value` capped at 80 chars.
* **Ingestion**: `POST /api/public/dashboard-activity`. Rejects cross-origin callers (`sec-fetch-site` / `origin` check), validates the body strictly with Zod, idempotently upserts the session, caps a session at 200 events, and updates session flags.
* **Aggregation**: `getActivitySummary` in `src/features/activity/activity.functions.ts` (`createServerFn`), reads up to 10k sessions, calculates aggregates, and returns `{ ok: true, summary } | { ok: false, reason: "service" }` without throwing.
* **Key Definitions**:
   * Finalized: `ended_at` is set or idle 20s+.
   * Bounced: Finalized with no meaningful interaction and under 15s.
   * Engaged: Any non-heartbeat interaction occurred.
   * Bounce rate: Computed over finalized sessions.
   * Action-click rate: Computed over all sessions.
 
## Key flows
 
1. **Session tracking**: First interaction generates a UUID in `sessionStorage`, sends `session_start`, followed by periodic heartbeats. Unload sends `session_end` via `navigator.sendBeacon` (fallback to `fetch(keepalive)`).
2. **Overview interaction**: View/tab switching, metric selection, chart point selection, range, and segment adjustments are tracked as meaningful interactions.
3. **Insight to action**: The "What this means" card leads to `/recommendation`, triggering `recommended_action_clicked`.
4. **Follow-up**: Creating a follow-up saves it to storage, emits `follow_up_created`, and routes to `/follow-up`.
5. **Readout**: Starts in an intro state ("Load live activity"), followed by loading, populated figures, `No data available` (zero sessions), or typed network/service/timeout errors with retry. Decision thresholds: Insufficient data (< 30 finalized sessions), Continue (bounce < 45% and action clicks > 20%), Pivot the headline metric otherwise.

## What's solid vs. what's duct tape

### Solid
 
* Clean route and screen structure matching the PRD with components named after their respective screens.
* Strict separation of content/data from presentation in `*.data.ts`.
* Ingestion security: same-origin checks, strict Zod validation, idempotent upserts, 200-event cap per session, RLS enabled.
* Comprehensive readout state machine: intro, loading, empty, 3 typed error states with retry, last-known values.
* Honest navigation: non-existent rail links are muted, non-interactive, and flagged with `aria-disabled`.
* Server functions return typed success/error objects instead of crashing. Client pauses recording after 3 consecutive failures.
 
### Duct tape
 
* Follow-up storage: Saved in `sessionStorage` only (one per session, lost on tab close).
* No auth or identity: Visitors are anonymous per-tab sessions; no workspace, cohort, or returning user tracking.
* Fixed product data: Activation chart, comparisons, benchmarks, and recommendations are static content rather than warehouse queries.
* Static thresholds: Constants in `experiment-readout.data.ts` rather than dynamic configuration.
* No A/B control arm: Baseline 60% bounce rate is historical/asserted, not split tested against a live control group.
* No committed test suite: Relies on typechecks, production builds, and ad-hoc Playwright scripts.
 
## Risks and assumptions
 
* Small-sample risk: Below 30 finalized sessions, the readout is not decision-grade.
* Undercounting: Ad blockers, privacy modes, and hard-kills can drop events.
* Session inflation: Reloads or new tabs create separate sessions without cross-tab deduplication.
* Kill-switch discipline: If the guided headline still bounces, the rule is to pivot the metric, not the layout.
* Scale assumption: In-memory aggregation reads up to 10k session rows; high-traffic scale will require SQL aggregation views.
 
## How to run it
 
```
bun install
bun run dev                          # Starts Vite dev server at http://localhost:8080
bunx tsgo --noEmit -p tsconfig.json  # Typecheck
bun run build                        # Production build
```
 
* Backend environment variables (`VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `VITE_SUPABASE_PROJECT_ID`) are provisioned automatically.
* To smoke-test: navigate through `/`, switch tabs, click the recommendation, create a follow-up, and click "Load live activity" on the Readout tab while monitoring network requests.
 
## Suggested next steps for an engineer
 
1. Persist follow-ups in the database table with owner/status fields to replace `sessionStorage`.
2. Connect the headline metric and time-series drilldown to read from the newly provisioned database tables.
3. Add a control arm and variant flag to compare bounce rate live against the default 12-chart landing.

