# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

the default analytics landing for trial workspaces shows twelve charts and no next step, so the people who should make decisions from it leave without acting (60% bounce, 6 clicks to the key number, 1.3 weekly sessions per active user, 12 charts on the default view). For the purposes of this PRD the hypothesis is treated as verified: a guided screen with one headline metric, a plain-language takeaway, and one recommended action causes decision-makers to act rather than bounce. Success criteria: bounce falls below 45% and recommended-action clicks exceed 20% of sessions, on a minimum sample of 30 completed sessions. Kill switch remains: if guided sessions still bounce past that sample, the headline metric is wrong — pivot the metric, not the layout.

## Users & jobs

- **Primary user:** The person accountable for trial activation (growth lead), with marketing manager, growth analyst and product lead as the decision-making audience
- **Job to be done:** on opening the dashboard, know in seconds whether activation is on track and what single action to take this week.

## Scope

- **In:** guided overview with one headline metric and trend, the insight-to-action block, the recommendation screen, the follow-up screen, the experiment readout with its intro/empty/error states, anonymous activity measurement, the compact rail plus top navigation.
- **Out (explicitly):** multi-metric analytics beyond the single headline; accounts, permissions or role-based access; a true A/B split against the old twelve-chart screen; notifications, email or alerts; editing, assigning or scheduling follow-ups; the Analytics, Experiments, Audiences and Settings menu destinations (kept in the rail as disabled placeholders); any integration with external analytics, CRM or ad platforms; persisted follow-up records beyond the current browser session.

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | Single headline metric above the fold | Must | Acceptance criteria: One dominant metric (7-day trial activation) is visible without scrolling on both desktop 1280 and mobile 390 viewports. A 13-week trend line appears directly beneath or beside the metric. No other chart competes with it for prominence. |
| 2 | Plain-language takeaway with one primary action | Must | Priority: Must have Acceptance criteria: The takeaway explains the metric in one sentence ("New teams are stalling before they invite a teammate"). Exactly one filled primary action button follows the takeaway ("Review recommended action"). No secondary CTAs appear in the same visual group. |

## Data & events

_What gets stored, what gets tracked._

Real and persisted: anonymous session rows and event rows (session id, start, last seen, end, duration, meaningful interaction, recommended-action click, follow-up created; event id, name, optional value, timestamp). Events recorded: session start, heartbeat, session end, view change, metric selection, chart-point selection, range change, segment change, recommended-action click, follow-up creation. Ingestion validates an allowed event list, rejects cross-origin calls, caps events per session, and de-duplicates. Aggregates read back: total and completed sessions, engaged, bounced, action-click and follow-up counts, bounce rate, click rate, average duration, collection start.
Mocked, hand-written content: the 46.2% activation figure and its 13-week series, all supporting-metric series and targets, the recommendation (invite step for paid-search trials, +6.8 pts, 1,284 trials, evidence rows), range and segment filters (presentation only, no different data), the follow-up record (browser session storage, not the database, and lost when the tab closes), the company identity and avatar, and the three quotes as authored research statements.
Also stated plainly: this measures engagement of the guided screen against thresholds; it is not an A/B comparison against the old twelve-chart screen and cannot claim causal lift.

## Open questions

thresholds and minimum sample not yet agreed with stakeholders; no control group, so how the kill switch is judged; whether activation is the right headline metric per role; follow-up persistence and ownership; retention period for activity data; whether the recommendation should be generated from data rather than authored; what the four disabled menu areas should become.
