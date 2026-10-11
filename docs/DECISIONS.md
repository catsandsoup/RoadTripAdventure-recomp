# Fork decisions and clarification log

## Confirmed by the owner, 11 October 2026

- Follow the upstream build, design, UI/UX and gameplay verification rules.
- Implement cheats/save editing, then new-game customisation/randomiser/races, then split-screen free roam, then online free roam/races. Put the integrated mod workshop last.
- New Adventure flow: **New Game → name → currency → body shop → paint shop → president scene → Q's Factory.**
- Use actual shop areas, new interaction dialogue and transitions. Make body and paint choices skippable and randomisable independently.
- Include the feature ambitions and acceptance checks in GOALS.md; they remain planned.
- Before implementing each part, ask all questions needed to resolve confusion and assumptions. Inspect evidence first and wait for answers before dependent coding.
- Create the GitHub fork and capture important agent instructions in the fork's planning documents.

## Pending feature decisions

No feature clarification gate has completed yet. GOALS.md's detailed behaviours are proposals unless recorded above or subsequently approved here. Setup does not mark gameplay features complete.

Use this format for each clarified part:

```text
Feature IDs / scope:
Date:
Verified current behaviour and source evidence:
Questions asked:
Owner answers:
Approved behaviour and defaults:
Persistence / back / cancel / error cases:
Acceptance examples:
Remaining questions or exclusions:
```

Record actual answers, not inferred preferences. Link each later change and verification report to its decision entry.
