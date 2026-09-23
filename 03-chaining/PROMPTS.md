# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: Guided-dashboard hardening chain

### Step 1: Build missing screens in strict sequence, anchored to the existing visual style.
```
Build the next phase of this app in a strict sequence:

1. Build {{Recommended-action detail screen}}: a separate focused view that opens when the user clicks "Review recommended action." 
It must show the affected segment (e.g. "Paid-search trials"), estimated activation impact (+6.8 pts),   two to three lines of supporting evidence, and a single "Create follow-up" button. 
When the follow-up is created, show inline confirmation ("Follow-up  created — view in Follow-ups") and record a `follow_up_created` event.
2. Build {{Follow-up overview panel}}: accessible from the main guided overview   via a "Follow-ups" link in the left navigation. It lists all previously created follow-ups (action title, segment, date created, status). If no follow-ups  exist yet, show the empty state "No follow-ups yet — review  a recommended  action to create one."
3. Navigation: {{Recommended-action detail}} links back to the guided overview  and forward to {{Follow-up overview}} after a follow-up is created. The {{Follow-up overview}} link should also be accessible from the left navigation rail at all times.
Anchor each screen to the attached screenshot so the visual style (charcoal rail, white workspace, #E11D2E accent, Sora/Manrope/IBM Plex Mono typography, thin borders, compact controls) matches the existing guided overview exactly.

Do not change anything else in the project or touch the experiment readout.
```

### Step 2: Hard-code loading, empty, and error states with exact copy.
```
Apply the following logic constraints to the flow:

1. Experiment readout — loading state: while session data is being fetched from  the server, show a skeleton placeholder (gray pulsing bars) in place of every    metric card. Do not show 0 or NaN values during load. 
2. Experiment readout — zero-session empty state: when the total session count  is 0, replace the entire metrics section with a centered message:   "No sessions recorded yet — share the dashboard link to start collecting data."
  Hide the baseline comparison and decision-state sections until at least 1  session exists.
3. Experiment readout — decision states: implement all three automatic states    from the README:
   - "Insufficient data" when finalized sessions < 30 (already exists). 
   - "Continue — guided screen is working" when sessions >= 30, bounce < 45%,  and action clicks > 20%. Use a green accent.
   - "Pivot the headline metric" when sessions >= 30 but engagement misses those thresholds. Use the red accent (#E11D2E) and show the copy: "The guided layout engaged users, but the selected metric did not drive action. Consider testing a different headline metric."
4. Activity endpoint — error feedback: if the dashboard-activity fails (network error, non-2xx response),  show a small, dismissible toast at the bottom of the screen:  "Activity tracking interrupted — your interactions may not be recorded."  Do not retry automatically.
5. Left navigation rail: remove click handlers from nav items that do not have  a built page (Audiences, Settings). Show them as visually muted (opacity 0.4)    with a "Coming soon" tooltip on hover. Keep Overview, Analytics, Experiments, and Follow-ups active.
Maintain the same design language throughout and tether all behavior strictly to these rules. Do not add any new screens or change layout structure.
```

### Step 3: One surgical visual polish on the most critical UI element.
```
The "What this means" text block on the Guided overview needs a visual-hierarchy
polish so the insight-to-action chain scans in under 3 seconds.

1. Start by comparing the current "What this means" block to the metric card directly above it — list the 3 biggest gaps in contrast, container treatment, and type weight.
2. Once you've identified those, make these changes:
   - Wrap the entire block in a light-background card (#F8F8F9) with a 3px left border in #E11D2E.
   - Set the first sentence ("New teams are stalling before they invite a teammate.") to semibold at one step larger than the current body size.
   - Restyle "Review recommended action" as a filled button (background #E11D2E, white text, same border-radius as other controls in the app).

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- Naming the exact element stops the AI touching the rest.
- Passing the README as context keeps the chain consistent.
- Attaching a screenshot as visual anchor gets closer to the right layout than describing it in words
- Hardcoding the exact string copy for empty/error states ("No sessions recorded yet…") prevents from inventing an own placeholder text
- Ending every prompt with "Don't change anything else" is cheap i — without it the AI helpfully refactors things you didn't ask about
- Auditing before prompting matters — I found 4 flow gaps I wouldn't have spotted if I'd just started building

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

I originally tried to get all three screens plus the loading/empty/error states in a single prompt. Lovable built the recommended-action detail screen but skipped the follow-up overview entirely and only added a loading skeleton on the experiment readout — it ran out of attention halfway through. Splitting into screens only then Behavior meant each prompt had one job. Both were  complete on the first try.
The Refine step almost broke the metric card above it — the AI interpreted "wrap in a card" as "restyle the whole section." Adding "compare the block to the metric card above it first" forced it to look at what already existed before making changes, so it only touched the insight block.


