# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: DASHBOARD HARDENING

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:
1.  Build {{screen A}}: Recommendation:  Build a separate screen for recommendation, current one is overwhelemed with data
2.Build {{screen B}} Follow-up :Add a screen for follow ups, to review the follow-up details
2. Navigation:{{screen A}} should be linked to{{screen B}}  when follow -up is created. Link to {{screen B}} should be available from main screen as well. to verview already planned follow-up.

Build these in order so {{screen A}} is the anchor for {{screen B}}.
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the flow:
- Use landing state on experiment read out
-Use empty state for zero sessions,{{No data available}}".
- Add error handling activity endpoint 
- remove left nav elements  to a non working page
Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine, surgical polish
```
Following things to improve:
- The "What this means" text block on the Guided overview needs a visual-hierarchy polish so the insight-to-action chain scans in under 3 seconds.
Start by comparing the current "What this means" block to the metric card directly above it — list the 3 biggest gaps in contrast, container treatment, and type weight.
Once you've identified those, wrap the block in a light-background card with a left border accent using 
#E11D2E, bump the first sentence ("New teams are stalling…") to a semibold subheading size, and restyle "Review recommended action" as a filled button instead of a plain text link.
- Hovers on main are appearance is extremely slow, make that faster.
- Navigation buttons to prior screen {{screen A}}, {{screen B}} should be  vsible on the top.
```

## Reusable techniques learned

- You need to provide provide as precise description as possible, planning helps, but should have describe the flow in more details, especially what should happen in loading state
- For auditing the readme AI can help, but user look is also needed

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

Navigation fixed, flow improved, showing clearly the action on recommendation, 

```
[prompt text]
```
**Expects in:** _____
**Produces out:** _____

## Reusable techniques learned

- _____
- _____

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

_____
