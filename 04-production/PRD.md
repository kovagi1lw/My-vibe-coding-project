# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem
 
The default analytics landing for trial workspaces shows twelve charts and no next step. The people who should make decisions from it leave without acting. Baseline behavior measured on that screen: 60% bounce rate, 6 clicks to reach the key metric, 1.3 weekly sessions per active user, and 12 charts on the default view.
 
For the purposes of this PRD the hypothesis is treated as verified: a guided first screen that leads with one headline metric, one plain-language takeaway, and one recommended action causes decision-makers to act instead of bounce.
 
## Hypothesis
 
A first screen leading with one headline metric, a plain-language takeaway, and one recommended action will cause users to act on the dashboard instead of bouncing, for the people who should make decisions from it.
 
### Success criteria
 
* Bounce rate on the guided screen falls below 45%.
* Recommended-action clicks exceed 20% of sessions.
* Minimum sample before declaring success or failure: 30 completed sessions.
 
## Users & jobs
 
**Primary user**: The person accountable for trial activation, typically a growth lead. They own the activation target and must decide what to ship this week.
 
**Decision-making audience quoted in the readout**:
 
* Marketing manager
* Growth analyst
* Product lead
 
**Job to be done**: On opening the dashboard, know in seconds whether activation is on track and what single action to take this week.


## Scope

### In scope
 
* Guided overview with one headline metric and its trend.
* Insight-to-action block directly beneath the headline metric.
* Recommendation screen with reason, impact, steps, and a create-follow-up action.
* Follow-up screen to review a planned follow-up's details.
* Experiment readout with intro, loading, empty, error, and ready states.
* Anonymous activity measurement for sessions and events.
* Compact left navigation rail plus top navigation strip.
 
### Out of scope (explicitly excluded)
 
* Multi-metric analytics beyond the single headline metric.
* Accounts, permissions, or role-based access.
* A true A/B split against the old twelve-chart screen.
* Notifications, email, or push alerts.
* Editing, assigning, or scheduling follow-ups after creation.
* Analytics, Experiments, Audiences, and Settings menu destinations — kept in the rail as disabled placeholders only.
* Any integration with external analytics, CRM, or ad platforms.
* Persisted follow-up records beyond the current browser session.

## Requirements
 
| # | Requirement | Priority | Acceptance Criteria |
|---|---|---|---|
| 1 | Single headline metric above the fold | P0 | One dominant metric (7-day trial activation) visible without scrolling on 1280 and 390 viewports; 13-week trend line beneath/beside metric; no competing charts |
| 2 | Plain-language takeaway with one primary action | P0 | Takeaway explains metric in one sentence; exactly one filled primary action button ("Review recommended action"); no secondary CTAs |
| 3 | Recommendation screen limited to reason, steps, and one CTA | P0 | Opens at `/recommendation` with unique title/description in head(); displays headline, impact, confidence, reason, 2–3 steps, "Create follow-up" button; no overview chart or baseline stats |
| 4 | Follow-up creation and detail screen | P0 | Creating follow-up navigates to `/follow-up` and records event; screen shows title, owner, due date, recommendation link, status, metric, target, checklist; empty state with recommendation link when none exists |
| 5 | Persistent follow-up entry point on the overview | P0 | Overview displays "Planned follow-up" card with title, owner, due date, status, "Review" link when one exists; shows "No follow-up planned" with recommendation link otherwise; survives navigation within session |
| 6 | Live experiment readout with thresholds and automatic decision state | P0 | Reads real anonymous sessions; computes bounce rate, action-click rate, average duration; renders: "Insufficient data" (< 30 sessions), "Continue" (bounce < 45% AND clicks > 20%), or "Pivot headline metric" |
| 7 | Intro state before first load | P0 | Opens with intro panel explaining measurements, thresholds, and minimum sample; "Load live activity" action triggers first read; numbers and decision panel hidden until result returns |
| 8 | Empty state for zero sessions | P0 | When load succeeds with no sessions, shows single "No data available" panel; explains measurements appear after first session; no em dash, 0%, or decision wording |
| 9 | Three distinct endpoint failure states with retry and last-known figures | P0 | Distinguishes network, service error, and timeout with plain-language messages and retry; failed reads preserve previous numbers marked "last known"; 8-second timeout prevents hanging |
| 10 | Disabled menu items for pages that do not exist | P0 | Analytics, Experiments, Audiences, Settings in left rail are non-interactive; muted styling, no hover, not focusable, marked `aria-disabled`; tooltip reads "Not available in this prototype" |
| 11 | 75ms hover and interaction feedback | P0 | All main-area buttons, tabs, links, cards transition ~75ms on hover/focus; keyboard focus visible and distinct; reduced-motion sets transitions to 0ms |
| 12 | Mobile 390 and desktop 1280 without overflow | P0 | Overview, recommendation, follow-up screens render without horizontal overflow at 1280×1800 and 390×844; compact rail collapses to icons on mobile; top nav usable on both sizes |
| 13 | Chart week selection storing state | P1 | Clicking week in trend line selects it and reveals detail panel; selection stored in component state during session; state resets on hard refresh |
| 14 | Range and segment filters wired to real data | P1 | Range (Last 13 weeks / Last 4 weeks) and segment (All trials / Paid search) selectors exist; send values to activity tracking; headline metric still uses mocked 13-week series |
| 15 | Richer error telemetry | P1 | Recording failures logged to console with context (status, count, paused); client reports distinct error kinds (network, service, timeout); no PII in user-facing messages |
| 16 | A control group for the readout | P1 | Readout can tag sessions as guided vs. default-landing; aggregation query supports filtering by group; UI copy acknowledges control comparison status |
| 17 | Notifications when a follow-up is due | P1 | Due-soon follow-up surfaces subtle indicator on rail icon; computed from follow-up due date in session state; no email or push notifications |
| 18 | Exporting the readout | P2 | Future release: add button to export readout as image or PDF |
| 19 | Saved views | P2 | Future release: persist selected range/segment per user |
| 20 | User accounts and editable recommendations | P2 | Future release: allow signed-in users to edit recommendations and persist follow-ups per user |

## Data & events
 
### Real and persisted
 
Anonymous session rows and event rows are stored in Lovable Cloud.
 
#### Session fields
 
* `session_id` (uuid, primary key)
* `started_at` (timestamptz)
* `last_seen_at` (timestamptz)
* `ended_at` (timestamptz, nullable)
* `duration_seconds` (integer)
* `meaningful_interaction` (boolean)
* `recommended_action_clicked` (boolean)
* `follow_up_created` (boolean)
 
#### Event fields
 
* `event_id` (uuid, primary key)
* `session_id` (uuid, foreign key)
* `event_name` (text, constrained to allowed values)
* `event_value` (text, nullable, max 80 chars)
* `occurred_at` (timestamptz)
 
#### Events recorded
 
* `session_start`
* `heartbeat` (every 10 seconds while active)
* `session_end` (on `visibilitychange` hidden or `pagehide`)
* `view_changed` (tab switches)
* `metric_selected`
* `chart_point_selected`
* `range_changed`
* `segment_changed`
* `recommended_action_clicked`
* `follow_up_created`
 
#### Ingestion behavior
 
* Validates an allowed event list.
* Rejects cross-origin calls with 403.
* Caps events per session at 200; returns 429 when the cap is reached.
* De-duplicates inserts by ignoring PostgreSQL `23505` unique-violation errors.
* Idempotently upserts the session row on every event.
 
#### Aggregates read back by the experiment readout
 
* Total sessions, completed sessions, engaged sessions, bounced sessions, action-click sessions, follow-up sessions.
* Bounce rate (`bounced / completed`).
* Action click rate (`clicked / total sessions`).
* Average duration among completed sessions.
* Collection start date.
 
### Mocked or hand-written
 
* The 46.2% activation figure and its 13-week series.
* All supporting-metric series and targets in the baseline block.
* The recommendation content: headline, impact (+6.8 pts), affected trials (1,284), evidence rows, steps, owner, due date.
* Range and segment selectors: presentation only; selecting them does not change the displayed headline metric or trend.
* The follow-up record: currently stored in browser `sessionStorage`, not the database.
* The company identity, avatar, and the three quotes in the readout.
 
## Open questions
 
* Thresholds and minimum sample are not yet agreed with stakeholders.
* There is no control group, so how the kill switch is judged remains unresolved.
* Whether activation is the right headline metric for every decision-maker role.
* Follow-up persistence and ownership beyond the current browser session.
* Retention period for anonymous activity data.
* Whether the recommendation should be generated from data rather than authored.
* What the four disabled menu areas (Analytics, Experiments, Audiences, Settings) should become if the prototype advances.
