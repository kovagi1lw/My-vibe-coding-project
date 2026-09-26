# Iteration: Analytics, Sprint, Final Recommendation

> Module 6 · Evals & Iteration. Read the analytics, run an iteration sprint, present with evidence.

## What the evidence says

_What real usage showed: numbers if your tool has analytics, counted behaviour if it does not. Put the signal that matters on screen._

- **Primary signal:** all pages were visited, mainly from desktop
- **What moved:** With login feature the visits dropped, prior that had direct visits.
- **What didn't:** bounce rate decreased with login.

_Analytics snapshot: visitors 21; page views 166; views per visit 7.9; duration 6m24s; bounce 29%._

_Observed behaviour: reach 1; core action 22.7%; stall point Follow-up; explained away none; own first run 2._

_____

## The recommendation

**Decision:** ☐ Go  ☑ Iterate  ☐ Kill

_The evidence that justifies the call:_

The guided screen works — both engagement thresholds pass, the urgency metric pivot was validated, and the decision state correctly fired "Continue." But the product isn't ready to ship to real users yet. The auth-wall blank screen on first login is a P0 (prompt 1 is now fixed, but prompts 2 and 3 are pending). The sample is 66 sessions from ~11 visitors, mostly reviewers — not production trial-workspace users. And there's no concurrent control, so we can claim "the experience engages people" but not "it outperforms the old 12-chart screen." The next iteration should: finish prompts 2–3, recruit a wider test audience, and decide whether to add a control group for causal measurement.

## Final showcase

- **Demo link:** https://insight-to-do-v2.lovable.app/
- **The one-sentence story:** A guided dashboard that replaced 12 charts with one urgent metric and one action proved — in two experiment rounds — that the metric matters more than the layout: the urgency framing ("trials expiring in 72 hours") passed both engagement thresholds where the abstract framing ("7-day activation rate") failed with the same design.
- **Where it landed on the Confidence Line (M2 → now):** M2 was a hypothesis with a clickable mockup. Now it's a working product with live anonymous tracking, auth, RLS, edge-case hardening, and 66 real sessions proving the guided approach works — with an honest acknowledgment that the sample is small, the audience is reviewers not production users, and there's no causal control. Confidence moved from "I think this could work" to "I have evidence it engages people, and I know exactly what to test next.
