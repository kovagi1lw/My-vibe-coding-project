# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

Trial workspaces land on a default analytics screen with 12 charts. Baseline usage of that screen is poor: 60% bounce, 6 clicks per session, 1.3 weekly sessions. The research quotes (marketing manager, growth analyst, product lead) say the same thing: the screen shows everything and tells no one what to do.

Hypothesis under test: a first screen that leads with one headline metric, a one-sentence takeaway and one recommended action will make users act instead of bounce. Success means bounce below 45% and recommended-action click rate above 20%, measured over at least 30 completed sessions.

Round 2 pivot: round 1 used "7-day trial activation rate" as the headline metric. Round 2 replaces it with an urgency metric, "Trials expiring in 72 hours with no activation" (23, +5 vs. last week). The recommended action is "Triage the 8 paid-search trials closest to expiry".

Where the evidence stands (honest status): collection started Sep 24, 2026. There are 27 completed sessions so far, 3 short of the 30-session minimum, so the readout says Insufficient data. Interim values meet both targets: bounce is 37.0% (10 of 27) and the action click rate is 22.2% (6 of 27). The hypothesis is therefore not yet validated. It is trending toward validation. Even after 30 sessions, the result is a threshold check against a historical baseline, not a controlled comparison. There is no concurrent control group and the traffic is not verified production users, so no causal lift can be claimed.

## Users & jobs

- **Primary user:** a growth or product decision-maker responsible for trial-to-paid conversion.
- **Supporting roles:**
  - **Marketing manager:** wants to know which acquisition channel's trials are at risk. Paid search is the flagged segment.
  - **Growth analyst:** needs the trend and a segment breakdown to trust the headline.
  - **Product lead:** needs a trackable follow-up that turns the insight into owned work.
- **Job to be done:** "When I open the dashboard, tell me what's at risk right now and the one thing to do about it this week, so I can act without reading 12 charts."

## Scope

- **In:**
  - **Guided overview:** headline metric (23, red "+5 vs. last week"), "What this means" insight card, one recommended action with "Estimated save: 3–4 activations this week", and "Review recommended action" button.
  - **Clickable headline:** the label and the value smooth-scroll to the metric drill-down.
  - **13-week count trend chart:** Y-axis 0–30, dashed "Target: Below 10" line, value labels on first/last/hovered points, clicking a point selects that week in the drill-down.
  - **Metric drill-down:** segments (All trials / Paid search / Organic), range control, per-week detail, target "< 10 per week".
  - **Recommended-action detail screen:** segment, estimated save, three evidence lines, one "Create follow-up" button with inline confirmation.
  - **Follow-ups list:** title, segment, date, status (newest first). Empty state: "No follow-ups yet — review a recommended action to create one." Always reachable from the left rail.
  - **Post-follow-up behavior:** overview button reads "Follow-up created" and opens /follow-ups instead of the detail screen.
  - **Left navigation:** Overview, Experiments, and Follow-ups are active. Audiences and Settings are muted (40% opacity) with "Coming soon" tooltip and no click action.
  - **Experiment readout:** live aggregates, automatic decision state, three-state target cards, baseline comparison, before-state quotes, current session card.
  - **Tracking-failure notice:** dismissible toast if an activity event fails to send. No retry.

- **Out (explicitly):**
  - Real product data for the headline metric, the trend, or the segments.
  - User accounts, permissions, or teams.
  - Editing follow-ups, assigning owners, or changing their status.
  - A/B assignment or a control group.
  - Audiences and Settings screens.
  - Notifications and integrations (CRM, email).
  - Multiple recommendations or ranking of recommendations.

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| R1 | Headline, takeaway, and action appear in the first viewport | Must | At 1333×937 and 390×844, the metric value, first insight sentence, and action button are visible without scrolling |
| R2 | Headline links to the drill-down | Should | Clicking the label or the value scrolls smoothly to the drill-down. Pointer cursor and "Explore ↓" arrow are visible |
| R3 | Trend chart gives context | Should | Title "Expiring-without-activation count — 13-week trend". Y-axis 0–30 labelled every 5. Dashed target line at 10 labelled "Target: Below 10". Labels on first, last, and hovered points only |
| R4 | Chart points drill down | Should | Clicking a point, or pressing Enter/Space on it, selects that week in the drill-down and scrolls to it |
| R5 | Recommended-action detail | Must | Shows the triage title, "Paid-search trials" segment, "3–4 activations this week", and three evidence lines |
| R6 | Create follow-up | Must | One click stores one follow-up titled "Triage the 8 paid-search trials closest to expiry" with inline confirmation. Repeat clicks create no duplicates |
| R7 | A follow-up counts as an action click | Must | Client sends `recommended_action_clicked` before `follow_up_created`. Server sets the clicked flag whenever `follow_up_created` arrives. No session can have a follow-up without a click |
| R8 | No repeat creation from the overview | Should | Once a follow-up exists this session, the overview button opens /follow-ups, not the detail screen |
| R9 | Follow-ups list | Must | Lists title, segment, date, and status (newest first). When empty, shows exactly "No follow-ups yet — review a recommended action to create one." |
| R10 | Readout loading state | Must | First load shows placeholder bars, never 0 or NaN. Background refreshes every 15s keep current numbers visible with no placeholder or flicker |
| R11 | Zero-session state | Must | With 0 completed sessions, shows exactly "No sessions recorded yet — share the dashboard link to start collecting data." Decision and checkpoint rows are hidden |
| R12 | Decision states | Must | Under 30 sessions: "Insufficient data". At 30+ with bounce < 45% and clicks > 20%: green "Continue — guided screen is working". At 30+ otherwise: red "Pivot the headline metric" |
| R13 | Target cards (interim progress) | Should | 0 sessions: gray "No data". 1–29 sessions: amber "[value] · n of 30 sessions". 30+: green "Passing" or red "Failed · [value]". Applies to both bounce and action click cards |
| R14 | Realistic session length | Must | Duration counted only while page is visible. Each session capped at 1,800s when stored and when averaged |
| R15 | Bounce definition | Must | A bounce is a completed session under 15 seconds with no meaningful interaction |
| R16 | Tracking-failure notice | Should | A failed event POST shows "Activity tracking interrupted — your interactions may not be recorded." Dismissible. No retry |
| R17 | Unavailable navigation is clear | Could | Audiences and Settings show at 40% opacity with "Coming soon" tooltip. No click action |
| R18 | Privacy | Must | No names, emails, IP addresses, or free text are stored. Public summary returns aggregate numbers only |

## Data & events

_What gets stored, what gets tracked._

**Real, live, and stored:**

- **Anonymous sessions:** a random ID per browser tab, kept in session storage with version `_v2` for round 2.
- **Events:** sent to `POST /api/public/dashboard-activity`. Each event is validated. The endpoint accepts same-origin requests only and at most 200 events per session. Duplicate event IDs are ignored.
- **Allowed events:** `session_start`, `heartbeat`, `session_end`, `view_changed`, `metric_selected`, `chart_point_selected`, `range_changed`, `segment_changed`, `recommended_action_clicked`, `follow_up_created`.
- **Meaningful interaction:** any event except `heartbeat`, `session_end`, and `session_start`.
- **Completed session:** has a `session_end` event, or no activity for 20 seconds or more. All aggregates use completed sessions only.
- **Formulas:**
  - Bounce rate = bounced ÷ completed
  - Action click rate = sessions with a click ÷ completed
  - Avg. session = mean of min(duration, 1800s)
  - Engaged = completed sessions with a meaningful interaction
- **Follow-up records:** stored durably with title, owner "Growth team", status "Planned", due label "Next experiment cycle", and segment. All reads and writes happen on the server. The public cannot access the table directly.

**Current round (Sep 24, 2026):**

| Measure | Value |
|---|---|
| Completed sessions | 27 |
| Engaged | 15 |
| Bounced | 10 (37.0%) |
| Action-click sessions | 6 (22.2%) |
| Follow-up sessions | 7 |
| Avg. session | ≈363s |
| Follow-up records (all rounds) | 29 |

Note: one session was recorded with a follow-up before R7 was fixed, so it has a follow-up but no click. That is why follow-up sessions (7) exceed click sessions (6).

**Mocked or static:** the headline value 23 and "+5 vs. last week" change, the 13-week counts (12→23), segment and range values in the drill-down, the "8 paid-search trials" and "3–4 activations" estimated save and evidence lines, the baseline figures (60% bounce, 6 clicks, 1.3 weekly sessions, 12 charts), the three research quotes (rewritten for the before-state), and the current session card (reads from browser state, not the server).

## Open questions

1. **Control group:** should the next round randomly show the old 12-chart screen to some visitors, so a causal lift can be measured?
2. **Who counts:** traffic so far is mostly reviewers of the prototype. Who are the target participants, and how will they be recruited or verified?
3. **Source of the headline metric:** which system of record would supply "trials expiring in 72 hours with no activation", and how often does it refresh?
4. **Round comparison:** should round 1 (activation-rate headline) be kept and compared with round 2? Round 1 data was cleared at the reset.
5. **The 30-session minimum:** is it enough? What confidence level is needed before choosing Continue or Pivot?
6. **Follow-up ownership:** who owns each follow-up, how does its status progress, and should duplicates across sessions be merged?
7. **Recommendations:** how is the one recommendation chosen, and who approves the estimated-impact figures?
8. **Data retention:** how long should anonymous session and event data be kept?
9. **Security:** several security findings were left outside earlier fixes. Who triages them before any real rollout?
10. **Pre-fix session:** should the session recorded before the R7 fix be backfilled as a click, or left as is?
