# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

A working React/TanStack Start prototype that replaces a 12-chart analytics landing page with a guided decision flow: one urgent metric ("23 trials expiring in 72 hours with no activation"), a plain-language explanation, one recommended action ("triage the 8 paid-search trials closest to expiry"), and a follow-up creation flow. Four screens — Guided overview, Recommended-action detail, Follow-ups list, and Experiment readout — are wired end to end. The headline metric, trend, evidence, and quotes are curated scenario data stored in Lovable Cloud (not a live analytics feed). Anonymous session tracking and follow-up records are also persisted there. The Experiment readout measures whether visitors actually engage with this guided experience against two thresholds: bounce < 45% and action clicks > 20%.

## Architecture (plain language)

- **Frontend:** React 19 with TanStack Start/Router, Vite, and Tailwind CSS. Code is organized by feature (`guided-overview`, `recommended-action`, `follow-ups`, `experiment-readout`, `activity`, `dashboard`, `workspace`). Routes are thin — they delegate to named screen components. View state (range, segment, selected week) is local React state; "follow-up created this session" lives in `sessionStorage`.

- **Backend / data:** Lovable Cloud provides PostgreSQL. Content tables (`metrics`, `metric_time_series`, `recommendations`, `baseline_benchmarks`, `research_quotes`) feed the guided screens via a single `getCoreDashboardData` server function. Behavior tables (`dashboard_sessions`, `dashboard_events`) store anonymous experiment activity via a validated `POST /api/public/dashboard-activity` endpoint (same-origin, 200-event cap per session, duplicate-tolerant, duration capped at 1,800s). `follow_ups` table stores created work items. `getActivitySummary` aggregates finalized sessions for the readout. All tables are locked from direct browser access — server functions use privileged credentials.

- **Key flows:**
  1. **Read the guided dashboard:** core content loaded from backend; if the read fails, local fixtures keep screens usable.
  2. **Explore the metric:** clicking headline or chart points scrolls to drill-down. Range, segment, and week changes are tracked as anonymous events.
  3. **Create a follow-up:** "Review recommended action" records `recommended_action_clicked` → detail screen → "Create follow-up" records `follow_up_created` and persists a follow-up. Browser session updates immediately; returning to overview redirects to Follow-ups list.
  4. **Measure the experiment:** new anonymous session per tab. Heartbeats count only visible-tab time. Background refresh keeps existing values visible (no flicker). Target cards show three states: "No data" at 0 sessions, amber interim at 1–29, final pass/fail at 30+.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| Feature-based code organization | solid | Screens map directly to PRD names and flows |
| Event tracking pipeline | solid | Schema-validated, deduplicated, volume-capped, duration-capped on write and aggregation |
| Follow-up persistence | solid | Survives reloads; events (`recommended_action_clicked`, `follow_up_created`) are consistent across tracking and ingestion |
| Background refresh (no flicker) | solid | Last successful read stays visible during refetch |
| Database row-level protection | solid | Tables not exposed directly to browser clients |
| Desktop + mobile end-to-end flow | solid | Exercised manually; latest preview build is passing |
| Headline metric and trend data | rough | Curated scenario data, not calculated from a production analytics source |
| Local fallback content | rough | Protects the demo from outages but can hide backend failures and drift from stored version |
| Recommendation selection | rough | Single recommendation via hardcoded slug; some derived fields ("This week", "Planned") are still in code |
| Follow-up ownership | rough | Global records, not scoped to a user or workspace. "Already created" is a per-tab sessionStorage marker, not a durable rule |
| Authentication | rough | Server functions use privileged access with no identity, workspace, or role enforcement |
| Activity tracking integrity | rough | Anonymous, client-reported. No bot filtering, stable visitor identity, consent controls, or synthetic traffic protection |
| Same-origin checks | rough | Reduces accidental cross-site submissions but is not strong caller authentication |
| Automated tests | rough | None — confidence comes from build checks and browser walkthroughs only |

## Risks & assumptions for the team

1. **Security first:** before any production use, authenticate follow-up reads/writes and scope every operation to a workspace with server-side enforcement.
2. **Data validity:** the experiment shows engagement with the prototype, but cannot prove causal improvement without a concurrent control group or random assignment. The 30-session threshold is a product rule, not a statistical power calculation.
3. **Metric source:** the product assumes a future system can reliably define trial activation, expiry, acquisition channel, and workspace membership. Those definitions are not implemented.
4. **Sample quality:** repeat visits, internal traffic, and bots can bias results. A session is a browser tab, not necessarily a unique person.
5. **Follow-up duplicates:** possible across tabs, sessions, or visitors. Add idempotency if one follow-up per recommendation/workspace is required.
6. **Failure visibility:** core data failures fall back silently to local content. Production needs monitoring to distinguish stale/fallback data from live data.
7. **Scale:** activity aggregation reads up to 10,000 session rows in application code. Move to indexed SQL aggregation or precomputed summaries as volume grows.
8. **Privacy:** activity is anonymous today, but production rollout needs a documented retention policy, consent decision, and review of any future identifiers or event values.

## How to run it

```
# In Lovable — preview runs automatically, use left nav to exercise all four screens.

# Locally:
# Requirements: current Node.js, npm, Lovable Cloud env vars (do NOT put
# private server keys in browser-visible variables or commit them)

git clone <repository-url>
cd <repository-name>
npm install
npm run dev
# Open the local URL printed by Vite

# Useful checks:
npm run build
npm run lint

# Available scripts: dev, build, build:dev, preview, lint, format
# No automated test command exists yet.
```

