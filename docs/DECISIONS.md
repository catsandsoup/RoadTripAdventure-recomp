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

The C1 clarification gate is in progress. GOALS.md's detailed behaviours are proposals unless recorded above or subsequently approved here. Setup does not mark gameplay features complete.

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

## C1 money cheats: first answer batch, 11 October 2026

Owner answers:

- Add a dedicated Cheats entry to upstream's existing custom Escape/controller menu; do not create a separate menu system.
- Unlimited Money makes purchases free. Rewards may still change the displayed balance.
- Money edits affect the current Adventure when the player saves, with a backup before the first edit.
- Persist the saved balance and Unlimited Money setting per Adventure across relaunch.

Pending research/clarification before dependent implementation:

- Identify purchase checks and debits, supported money limits and safe mutation points from USA code/probes.
- Resolve the scope of free spending (shops, services, bets and other costs) and affordability handling at low balance.
- Resolve save identity/slot policy for cheat settings and automatic backup/restore, plus save-state interactions.
- Confirm custom-amount input and cancellation flow after inspecting existing controller UI components.

No gameplay implementation has begun.

UI reference supplied by owner, 11 October 2026: the existing Escape menu has Resume, Options, Controllers and Quit, with a cyan panel, orange border/title tab, blue focused row, gold horn cursor, chunky outlined text, player-status band and bottom input hints. Add Cheats to this existing root menu and reuse its components. The screenshot stays local and is not committed.

## C1 money cheats: second answer batch, 11 October 2026

- Unlimited Money covers every payment, including betting stakes. Rewards still change the balance.
- Require an ordinary in-game save before enabling or applying money cheats, so a recoverable backup exists.
- Loading an older save state restores the Unlimited Money setting from that saved moment.

Implementation consequences: intercept affordability checks as well as debits; pair save-state snapshots with cheat metadata; scope persisted settings to the loaded Adventure. Static research establishes a normal transaction cap of 999,999, so 99,999 is a preset rather than the game maximum. Runtime validation remains pending.

## Lifecycle and adversarial verification, 11 October 2026

Owner requires this rule throughout the fork: inspect how affected functions work across their lifecycle, including adjacent, prerequisite and subsequent functions; recurse through relevant dependencies and test with a failure-seeking approach. Start from the hypothesis that a change will break, and try to demonstrate that break rather than follow only a happy path.

Apply to money transactions and all later features. Document examined dependencies, break hypotheses, evidence, failures, repairs, retests and unknowns. This requires substantive adversarial tests; it does not justify claiming mathematical/exhaustive proof from passing samples.

## C1 money cheats: bank and betting decisions, 11 October 2026

- Bank deposits and withdrawals remain normal transfers. Unlimited Money must not create bank funds by making deposits free.
- A winning bet pays the full original payout even though the stake was free. Correct the preview to show the actual gain.
- Owner enabled Computer access for visual verification; use an isolated test instance for gameplay validation.

C1's core behaviour is now clarified. Code-level hook safety, save identity and full lifecycle coverage still require evidence; ask again if research exposes a new product choice.

## C1 money cheats: prices and player trades, 11 October 2026

- Keep original shop prices visible and add a free-payment indication while Unlimited Money is enabled.
- For player-to-player part trades, waive the buyer's payment while crediting the seller normally. This differs from bank transfers, which remain normal.

Acceptance checks: a completed trade adds the part to the buyer, removes it from the seller, leaves the enabled buyer's wallet unchanged and credits the seller exactly once at the original trade price. Cancellation and failed inventory checks must transfer neither money nor parts. Disabling Unlimited Money restores the original payment behaviour. Save/reload and save-state restoration must preserve the applicable Adventure setting without replaying a completed transfer.

These presentation and trade choices are resolved. Save identity, safe hook/resume behaviour and lifecycle coverage still require investigation; record further questions if that evidence exposes a product choice.

## C1 money cheats: copies, legacy states and backup scope, 11 October 2026

- A copied or imported Adventure starts with Unlimited Money off on its destination card, including copies made by this game.
- A save state created before cheats existed remains loadable and restores its original balance with Unlimited Money off.
- Require a fresh ordinary save of current progress before the first money-cheat edit; an earlier completed save is insufficient.
- Back up the whole selected memory card and its cheat settings before that edit.

Acceptance checks: unrelated existing saves and cancelled/failed saves do not unlock edits; a new Adventure cannot inherit an old Adventure's eligibility. Verify the backup contents before enabling mutation. Copy/import must leave the source's setting intact while the destination starts off. Legacy state loading must not fail solely because cheat metadata is absent. If backup creation or verification fails, refuse the edit and explain recovery; do not mutate gameplay first.

The requested persistence policies are resolved. Safe completion hooks, identity reconciliation and interruption recovery remain engineering work requiring runtime evidence.
