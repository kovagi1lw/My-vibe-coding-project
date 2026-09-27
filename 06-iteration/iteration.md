# Iteration: Analytics, Sprint, Final Recommendation

> Module 6 · Evals & Iteration. Read the analytics, run an iteration sprint, present with evidence.

## What the evidence says

_What real usage showed: numbers if your tool has analytics, counted behaviour if it does not. Put the signal that matters on screen._

- **Primary signal:** The experiment passed both thresholds at 66 completed sessions — bounce rate 42.4% (target: below 45%) and action click rate 22.7% (target: above 20%). Decision state flipped to "Continue — guided screen is working.
- **What moved:** The round 2 urgency metric ("Trials expiring in 72 hours with no activation") drove action where the round 1 abstract metric ("7-day trial activation rate") did not. Round 1 stalled at 56% bounce and 17% clicks after 116 sessions — both failing. Same layout, different metric, completely different result. The pivot was the right call.
- **What didn't:** Auth friction is eating visits before they become sessions. Lovable analytics shows 11 visits generating 56 page views, with /auth as the most-visited page (8 hits). The 57% bounce rate in Lovable analytics (which includes auth-wall bounces) is much higher than the 42.4% in the experiment readout (which only counts completed sessions past auth). Peer feedback confirmed: first Google login rendered a blank screen.

_Observed behaviour: reach 1; core action 22.7%; stall point Follow-up; explained away none; own first run 2._

## Iteration sprint

| Change | Hypothesis | Result |
|---|---|---|
| Fix first-login blank screen — wait for profile creation trigger + core data before rendering; show "Setting up your workspace…" skeleton after 3s instead of white screen | First-time Google sign-in visitors are bouncing because nothing loads; fixing this removes the #1 drop-off point before the guided screen | Ran prompt 1. First Google login now shows a loading skeleton and resolves to the guided overview within seconds instead of a blank page |

## Peer feedback

_____

## The recommendation

**Decision:** ☐ Go  ☑ Iterate  ☐ Kill

_The evidence that justifies the call:_

The guided screen works — both engagement thresholds pass, the urgency metric pivot was validated, and the decision state correctly fired "Continue." But the product isn't ready to ship to real users yet. The auth-wall blank screen on first login is a P0 (prompt 1 is now fixed, but prompts 2 and 3 are pending). The sample is 66 sessions from ~11 visitors, mostly reviewers — not production trial-workspace users. And there's no concurrent control, so we can claim "the experience engages people" but not "it outperforms the old 12-chart screen." The next iteration should: finish prompts 2–3, recruit a wider test audience, and decide whether to add a control group for causal measurement.

## Final showcase

- **Demo link:** https://insight-to-do.lovable.app/
- **The one-sentence story:** A guided dashboard that replaced 12 charts with one urgent metric and one action proved — in two experiment rounds — that the metric matters more than the layout: the urgency framing ("trials expiring in 72 hours") passed both engagement thresholds where the abstract framing ("7-day activation rate") failed with the same design..
- **Where it landed on the Confidence Line (M2 → now):** M2 was a hypothesis with a clickable mockup. Now it's a working product with live anonymous tracking, auth, RLS, edge-case hardening, and 66 real sessions proving the guided approach works — with an honest acknowledgment that the sample is small, the audience is reviewers not production users, and there's no causal control. Confidence moved from "I think this could work" to "I have evidence it engages people, and I know exactly what to test next."
