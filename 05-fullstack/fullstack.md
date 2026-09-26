# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

_The working, shareable link that survives real users._

https://insight-to-do-v2.lovable.app/

## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| metrics | id, name, value, change_text, target_value, insight_detail | Headline metric ("Trials expiring in 72h"). target_value and insight_detail are new nullable columns replacing hardcoded copy |
| metric_time_series | id, metric_id, week_label, value, segment | 13-week count series per segment. Replaces hardcoded TRIAL_RISK_WEEKS array |
| recommendations | id, metric_id, title, segment, estimated_save, evidence, is_active | is_active (default true) lets the system deactivate a stale recommendation without deleting it |
| baseline_benchmarks | id, label, value, unit, description | The "old 12-chart screen" reference numbers (60% bounce, 6 clicks, 1.3 sessions, 12 charts) |
| research_quotes | id, quote, role, context_note | Before-state user quotes, tagged "Before the guided screen" |
| experiments | id, name, metric_id, started_at, min_sample, bounce_target, click_target, status | Replaces all hardcoded thresholds (30/45/20) and the start date. Status: active/completed/paused |
| profiles | id (= auth user id), display_name, workspace_id | Created on signup via trigger |
| workspaces | id, name | Default workspace created on first signup |
| workspace_members | workspace_id, user_id | Maps users to workspaces for RLS scoping |
| user_roles | user_id, role (app_role enum: admin/member) | Checked via has_role() security-definer function, never from the browser |
| follow_ups | id, recommendation_id, title, segment, status, due_label, created_by, workspace_id, created_at | created_by and workspace_id are new nullable columns. Old rows without them are backfilled to a legacy workspace |
| dashboard_sessions | id, experiment_id, user_id, started_at, ended_at, duration_s, is_bounced, has_click, is_finalized | experiment_id and user_id are new nullable columns linking sessions to the active experiment round |
| dashboard_events | id, session_id, event_name, metadata, created_at | Validated against an allow-list of 10 event names. Capped at 200 per session. Duplicate event IDs ignored |

## Access rules

_Who can see / do what? Where are the auth boundaries?_

**Anonymous visitors (not signed in):**
- Can view the Guided overview, trend chart, and drill-down (core content reads are public).
- Can submit anonymous activity events via `POST /api/public/dashboard-activity` (service role, same-origin check, validated, rate-capped).
- Cannot access follow-ups, the experiment readout, or any write operations beyond activity tracking.
- Hitting a protected page redirects to `/auth`.

**Signed-in members:**
- Can read all reference data: metrics, time series, recommendations, baselines, quotes, experiments.
- Can read follow-ups scoped to their workspace only.
- Can create follow-ups where `created_by` = their own user ID and `workspace_id` = their workspace.
- Cannot update or delete follow-ups unless they are the creator.
- Can read and update only their own profile. Can read only their own roles.

**Admins (has_role('admin'), checked server-side):**
- Everything a member can do, plus:
- Can read the Experiment readout (activity summary aggregation).
- Can update or deactivate recommendations.
- Can update follow-ups created by any member in their workspace.
- Can read the raw activity summary. Raw session/event rows are never exposed to the browser — only aggregates.

**Hard boundaries:**
- RLS is enabled on all tables. Policies enforce workspace scoping and ownership.
- Admin checks use `has_role()` security-definer function on the server, never in the browser.
- `dashboard_sessions` and `dashboard_events` have no browser-level SELECT/INSERT — all access is through the service role on the server.
- The public activity endpoint validates event names against an allow-list, enforces same-origin, caps at 200 events per session, and deduplicates by event ID.

## Edge cases hardened

| Case | Before (integration plan) | After (hardened) |
|---|---|---|
| **Empty / first-run: no follow-ups** | Follow-ups list shows a blank page | Shows "No follow-ups yet — review a recommended action to create one." with a CTA linking to the guided overview |
| **Empty / first-run: zero experiment sessions** | Readout shows 0s and NaN percentages | Shows "No sessions recorded yet — share the dashboard link to start collecting data." Decision and target rows are hidden until ≥ 1 session |
| **Empty / first-run: old follow-ups with no owner** | Orphan rows with null created_by and workspace_id | Backfilled to a "Legacy" workspace on migration. Visible to admins; members see only their workspace's rows |
| **Empty / first-run: short or empty time series** | Chart crashes or shows a flat line with no context | Guards for < 2 data points — shows "Not enough data to display a trend" instead of rendering the chart |
| **Bad input: recommendation slug missing or deactivated** | "Create follow-up" button still clickable, creates a broken row | Button is disabled with a message "This recommendation is no longer active." Server rejects follow-up creation if `is_active = false` |
| **Bad input: double-clicking "Create follow-up"** | Creates duplicate follow-up rows | Button disables on first click (optimistic UI); server deduplicates by recommendation_id + session within a short window |
| **Bad input: malicious activity payload** | Accepts any event name or oversized metadata | Allow-list of 10 valid event names. Metadata size-limited. 200-event cap per session. Duplicate event IDs silently ignored. 429 on excessive requests |
| **Bad input: non-admin opens experiment readout** | Readout renders with full data for anyone | Shows "Readout is admin-only" state. Server-side `has_role('admin')` check — the aggregation query never runs |
| **Failure / offline: token expires during 15s refresh** | UI hangs or shows a white screen | Catches 401 on background refresh, redirects to `/auth` without flicker. No skeleton flash — last known values stay visible |
| **Failure / offline: activity POST fails** | Silent failure, experiment data has gaps nobody notices | Dismissible toast: "Activity tracking interrupted — your interactions may not be recorded." No retry |
| **Failure / offline: core data backend unavailable** | App crashes or shows empty screens | Falls back to labeled local fixtures with a visible "Sample data" badge so the user knows it's not live |
| **Failure / offline: multiple tabs with sessionStorage** | Sessions leak across tabs or collide | Each tab gets its own random session ID in `sessionStorage`. `sendBeacon` on `pagehide` finalizes the session reliably |
| **Failure / offline: switching experiments while sessions are open** | Open sessions counted toward the wrong experiment | Sessions are stamped with `experiment_id` at creation. A round change only affects new sessions |
| **Failure / offline: clock skew or sessions > 30 minutes** | Duration could accumulate indefinitely (10,000s+ observed) | Duration counted only while `document.visibilityState === "visible"`. Capped at 1,800s on both write and aggregation |
| **Failure / offline: sendBeacon fails on page close** | Session might never finalize | Sessions without `session_end` auto-finalize after 20s inactivity in the aggregation query |

## Stress test results

_What you threw at it, and what held / broke._

_____


