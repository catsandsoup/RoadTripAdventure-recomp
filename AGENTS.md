# Fork instructions for agents

## Scope and precedence

This is `catsandsoup/RoadTripAdventure-recomp`, forked from `silentsudin/RoadTripAdventure-recomp`. The setup baseline is `0a4e8d3afa933b5df031e9b6afa3f95ece7ea0e1`.

Read `GOALS.md`, `CLAUDE.md`, `docs/DEVELOPING.md`, `docs/UPSTREAM_INSTRUCTIONS.md` and the relevant `.claude/agents/` briefs before working. Preserve upstream's architecture, build rules, visual design and gameplay verification methods. Direct owner instructions take precedence. Treat upstream session history, old pass counts, personal paths and historical push approvals as context, not current authorization or current test evidence.

## Required clarification before implementation

The owner explicitly requires all necessary questions before implementing each part. Inspect code and tests first; bring concrete choices rather than questions the repository already answers.

- State verified facts, unresolved choices and any proposed assumptions separately.
- Ask focused batches covering the feature's flow, scope, defaults, persistence, cancellation, edge cases and success checks. Wait for answers before implementing dependent behaviour. Never infer approval from silence.
- Record answers in `docs/DECISIONS.md` with feature IDs and acceptance examples. Preserve answers across turns; ask again only if scope changes or evidence reveals a new conflict.
- Continue independent read-only investigation while waiting. Setup/planning work is allowed; pending questions do not authorise gameplay changes.
- Reopen clarification if new ambiguity appears during implementation. Do not invent game mechanics, address layouts, performance claims or decisions.

## Roadmap

Follow GOALS.md: baseline, cheats/save editing, shop-based introduction/randomiser/race customisation, split-screen free roam, online free roam/races, then the native mod workshop. Selecting original bodies and paint does not require the workshop.

Confirmed new Adventure order: **New Game → player name → currency name → actual body shop → actual paint shop → president scene → Q's Factory.** Use new interaction dialogue and transitions, with independent skip/randomise options for body and paint. Clarify detailed choices before implementation; a host-overlay picker does not fulfil this goal.

## Build and runtime

- Target USA SLUS-20398. Verify hooks/layouts against this version; old PAL work is reference material.
- Preserve first-launch extraction/recompilation/compilation from the player's disc. Ship no game code or data in the app. Run upstream's packaged-symbol check for built releases.
- Commit no generated code, disc images, ELFs, extracted assets, saves, game screenshots or dumps. Use upstream hash-only golden convention and ignored local output directories.
- Follow `scripts/pipeline.py`, current presets and SDK/build-ID handling. Edit `config/roadtrip.base.toml`; regenerate `config/roadtrip.toml` with analysis.
- Put game hooks in `src/game/overrides.cpp`. Use existing settings, input, UI and `src/platform/Host.h` abstractions. Preserve DLL/dylib/runtime shared-state and ABI requirements in CLAUDE.md.
- Keep `third_party/PS2Recomp` at its pinned upstream commit for setup. If runtime edits become necessary, first establish a writable fork/branch under this owner's account, then update the superproject pointer to a published commit. Do not push to silentsudin's repositories or replace the submodule without authorization for that work.

## Design and interaction

Use `src/ui/Theme.cpp`, existing controls and the two upstream UI critic briefs. Match Road Trip's rounded, colourful, chunky presentation: blue bevelled menu panels, bright selection, gold horn cursor, warm-orange/olive dialogue panels, readable outlined type and controller-family glyphs. No stock ImGui styling or generic desktop-tool panels in player flows.

Keep panel position/size stable during focus changes. Complete each flow with a controller alone, including text entry, and preserve keyboard/mouse and supported touch interaction. Test handheld/TV readability, scaling, contrast, 4:3 and ultrawide layouts. Use plain labels, live feedback and explicit restart requirements. Hide unimplemented/device-unsupported features according to current CLAUDE.md; explain temporary blockers through reachable hints.

Use actual game interiors, dialogue and shop tasks for the introduction. Load original art/fonts from the player's disc at runtime; commit only original assets. Keep solo pause/input isolation intact. Define local-menu ownership for split-screen and online before implementing those modes.

After UI edits, compare local screenshots with game references and use `choroq-style-critic` and `jobs-ux-critic` for their specified reviews. Fix mandatory findings and record outcomes. Do not claim a review occurred without screenshots and results.

## Gameplay and saves

Read `config/game_state.toml` before researching fields. Follow `reverse-engineer`: cite addresses and evidence; label inference; use short controlled probes when static analysis cannot settle a question. Close probe processes.

Preserve upstream behaviour when fork modifiers are off. Research race-count dependencies throughout AI, collisions, cameras, dialogue, HUD, results, replays, team scoring and saves. Preserve original save fields/formats; keep added content metadata in versioned sidecars. Back up before destructive changes, write atomically, and provide restore. Clarify reset retention and stale-save-state handling before coding.

Use played-through routes to prove progression. Label edited-save fixtures, warps or other shortcuts as narrow probes; they do not prove an end-to-end Adventure. Test real-time play as well as deterministic/headless runs.

## Lifecycle and adversarial verification: owner requirement

Apply this rule to every feature, not only money cheats. Assume the proposed change breaks a connected behaviour and actively try to demonstrate that failure before accepting the design.

- Trace the full lifecycle: entry/preconditions, adjacent calls/shared data, mutation ordering, callbacks/resume points, side effects, cancellation/failure, persistence, and subsequent use/reload. Recurse through relevant callers/callees and shared-state consumers; record visited paths and unresolved boundaries.
- Build an evidence-backed dependency graph. Treat inventory, progress, UI, audio, AI, replay, save states and storage consumers as part of the change when they share affected data or invariants.
- Write concrete break hypotheses and tests that could demonstrate each failure. Include zero/min/max/out-of-range inputs, repeat operations, interrupted/cancelled work, stale state, transitions, conflicting settings, missing/corrupt files, and concurrency where relevant. Do not settle for a single successful action.
- Reproduce baseline failures separately. For each test, report what it exercises and what it cannot establish. A successful narrow probe does not prove the whole lifecycle.
- Resolve demonstrated failures and retest their connected paths. Record untested edges and residual risks; do not claim exhaustive proof, and do not update goldens or weaken assertions to conceal a break.
- Use targeted recursion with explicit boundaries and a visited-path list. Follow newly discovered dependencies until the affected invariants are understood; never substitute an unbounded test loop for a defined completion gate.

## Verification and release

Use `bug-tester` for regression triage, `perf-profiler` for repeatable measured performance, and `release-guard` before commits/pushes. Follow the briefs' evidence/report formats. Reconcile older branch names or test counts with the current checkout. User approval is required for changed goldens; explain mismatches rather than changing hashes to mask bugs.

For this fork, adapt release-guard's identity check to **catsandsoup and its verified GitHub no-reply email**. Never impersonate silentsudin or fabricate Claude-session/coauthor trailers. Preserve existing upstream attribution. A documentation-only change needs document/link/privacy checks; a full gameplay regression and packaged-app check remain required when the affected code/release warrants them. Report unrun checks as unrun.

Check staged/published files for credentials, personal machine paths and disc-derived content. Follow applicable owner instructions for commits, pushes and releases. Fork setup is not permission to implement all roadmap features or deploy anything.

## Deployment

For Vercel deployment, use its CLI or installed plugin. Preserve project link, team and protection. Commit/push permission is separate from deployment permission; do not deploy through GitHub Actions, integrations or a push unless the owner asks for that method.
