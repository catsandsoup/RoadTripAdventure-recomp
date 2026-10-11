# Money cheats: USA research

Date: 11 October 2026. Fork code baseline: `ed4725008029a783cf024394cbba1989121b6262`; upstream code baseline: `0a4e8d3afa933b5df031e9b6afa3f95ece7ea0e1`.

This is static research, not implemented cheat behaviour or a runtime test pass. Derived code and probe artifacts stay in ignored local directories.

## Confirmed product decisions

See `DECISIONS.md`: dedicated Cheats entry in the existing Escape/controller menu; all spending free while Unlimited Money is enabled, including bets, with bank transfers remaining normal; rewards still affect balance; require an ordinary save before cheating and back it up before the first edit; persist balance/toggle per Adventure; save states restore the toggle from that moment.

Keep original shop prices and add a free-payment indication. In player-to-player part trades, waive the enabled buyer's payment but credit the seller normally. Preserve item eligibility and cancellation; a bank transfer remains excluded.

## Verified static findings

| Item | Source/evidence |
|---|---|
| Money field | `config/game_state.toml`: confirmed u32 at progress offset `0x654`, RAM `0x0177FDB4`. |
| Initial money | USA ELF initialization at `0x00229FD8–0x0022A048`, store at `0x0022A004`: player 0 starts with 1000. |
| Shared money adjustment | `0x0023DC58–0x0023DCBC`: signed debit in `a0`, player index in `a1`; progress stride `0x3448`. Insufficient balance returns -1; success returns 0. Negative debit grants money. |
| Native transaction cap | Adjustment routine clamps to `0xF423F` (999,999). Preview calculations at `0x0022E7F0–0x0022E8B8`, `0x0022E8C0–0x0022E95C` and `0x002693A8–0x002693CC` also clamp there. |
| Transaction sound | Nonzero debit calls `0x00258E50` with sound ID `0x305`. Preserve appropriate feedback rather than deleting transaction effects. |
| Credit callers | `0x002366EC–0x002366F4`, `0x00236C7C–0x00236C88`, `0x0023A218–0x0023A224` negate rewards before adjustment. |
| Earlier affordability checks | Examples at `0x002699DC–0x002699E8`, `0x00247318–0x00247330`, `0x002567F4–0x00256804`. Disabling the final debit alone does not permit every purchase at zero balance. |
| Item ordering | Callers near `0x002697D0` and `0x00269AE8` perform item operations before the adjustment; preserve their success/failure ordering. |
| Display wrappers | `0x0022E780`, `0x0022E7C0` read balance/currency for shared drawing function `0x00202D10`. No separate display cap has been established. |
| Current host menu | `src/ui/PauseMenu.cpp`: root rows include Resume, optional state slots, Options, Controllers, Quit. Add Cheats through this page system and Theme components. |
| Existing persistence test | `tests/regression/test_memcard.py` writes 4321, saves and reloads. This describes test intent, not a current pass. |

99,999 is the owner's preset, not the game maximum. Constrain editable normal balances to the verified transaction range unless further research establishes another safe range. Do not infer safe values from unsigned storage size.

## Additional lifecycle findings and decisions

Static call tracing found a synchronous sound path from the money-adjustment routine through `0x00258E50`, `0x00258308` and `sceSifCallRpc` at `0x00287EA8`. Preserving the transaction includes scheduler continuation at `0x0023DCB0`; replacing the debit argument with zero would suppress its normal sound. Runtime verification must confirm this path.

Betting includes a wager ceiling based on the balance and a low-balance refusal before the final debit. Its winning payout uses wager × odds, while its preview assumes a deducted stake. The owner confirmed full payout for a free stake and an updated preview showing the actual gain.

The paired debit/credit workflow near `0x0023A460` transfers value between wallet and the field at progress offset `0x330C`. Deposit/withdrawal dialogue strings corroborate bank use, but runtime entry/dispatch validation remains pending. The owner confirmed ordinary bank transfers remain charged/credited normally; they are excluded from free spending. A no-debit hook applied to deposits would violate this decision and allow deposit/withdrawal money creation.

## Save lifecycle: static evidence

The original game stores one Adventure progress file per memory card. Card selection and player progress-block selection are separate task arguments. No immutable Adventure identifier has been established; names, currency and content hashes cannot serve as stable identity across edits and saves.

- Normal save task `0x00228708` writes auxiliary files, then progress. `0x00228A34` writes `0x3448` progress bytes. Completion branches near `0x00228AB0–0x00228AF0` distinguish success from failure; registered resume `0x00228AC4` follows the success sound. File existence alone does not prove this Adventure completed a normal save.
- Normal load task `0x00228510` reads the selected card into the selected player's block, then transfers to completion task `0x002283E0`, which checks the read size and close result. Attach host metadata only after verified completion.
- Runtime `MemoryCard.cpp` writes and flushes the destination directly; ordinary saving is not atomic. Snapshot card bookkeeping restores open-file positions, not card-file contents. A snapshot load therefore does not rewind an ordinary save.
- `EeSnapshotProbe.cpp` blocks snapshot capture while a card file is open. `StateSlots.cpp` marks cards changed after loading a snapshot so the game rereads present contents.
- `SaveState.cpp` requires every registered custom section. A new cheat section would reject old snapshots lacking it. The existing `GAME` section in `src/game/overrides.cpp` can instead use the archive's additive versioning. The owner confirmed old states should load with cheats off; implementation and runtime checks remain pending.

Implementation must pair a host identity with verified load/save completion, preserve original progress bytes, verify the first-edit backup before mutating, and recover from interruption between the original save and sidecar writes. Atomic sidecar rename alone cannot guarantee a consistent pair. These are design constraints, not implemented protection.

The owner requires a fresh save of current progress and a backup of the whole selected card plus its cheat settings. Copies/imports start with Unlimited Money off on the destination. Eligibility and metadata must therefore distinguish completed saves of this active Adventure from unrelated, failed or stale saves.

Additional break hypotheses: an unrelated existing card makes an unsaved New Game eligible; a short/failed save establishes eligibility; renaming loses the toggle; imports inherit another Adventure's toggle; restoring Adventure A's snapshot overwrites Adventure B's metadata; a legacy snapshot becomes unloadable; an interrupted paired write loses identity or backup provenance. Probe these before accepting persistence.

## Runtime verification still required

- Trace debits, credits and affordability checks through parts/body/paint shops, services and roulette/bets. Confirm which paths bypass the central routine and how they display prices and previews.
- In disposable test Adventures, verify balance 0, 99,999, 999,998 and 999,999 across spending, rewards, save/load and title/new-game transitions.
- Prove free payment at zero balance grants the purchased item/service, retains the balance, and preserves rewards and game feedback. Exercise cancel and failed transactions.
- Verify safe hook entry/resume points across scheduler suspension. Do not temporarily inflate a live wallet across frames or save checkpoints to simulate free purchases.
- Bind sidecar persistence to the loaded Adventure/save and verify changing slots, starting a different Adventure, copying/importing a save and restoring a backup.
- Snapshot cheat state through the existing save-state registration mechanisms; verify old states without fork cheat data, cross-Adventure loads, and rollback of both balance and toggle.
- Verify no edits outside the intended field and sidecar, no cheats during title/demo/stale progress contexts, and baseline equivalence with cheats off.

Record actual probe results before selecting the final hook design. Ask the owner if evidence exposes a new product choice.
