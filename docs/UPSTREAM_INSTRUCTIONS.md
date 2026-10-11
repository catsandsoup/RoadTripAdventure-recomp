# Upstream instruction map

Captured during fork setup on 11 October 2026 from `silentsudin/RoadTripAdventure-recomp` commit `0a4e8d3afa933b5df031e9b6afa3f95ece7ea0e1`. The inherited files remain intact. This map helps Codex discover rules stored for upstream's agent environment.

## Sources to read

| Source | Role in this fork |
|---|---|
| `CLAUDE.md` | Primary technical instructions: first-run build, no shipped game code, hooks, menu/theme, controller mapping, platform abstractions, graphics/audio/runtime constraints. |
| `docs/DEVELOPING.md` | Current pipeline and platform builds, on-device compiler/SDK, runtime structure, test socket, deterministic harness and save tooling. |
| `.claude/agents/choroq-style-critic.md` | Visual review: game art direction, colours/panels/cursor/type, consistent tokens, contrast, readable sizes and reference screenshots. |
| `.claude/agents/jobs-ux-critic.md` | Flow review: controller-first navigation, few presses to driving, plain labels, useful errors, restart disclosure and safe return to play. |
| `.claude/agents/reverse-engineer.md` | Gameplay research: known state map first, USA address evidence, static analysis/runtime probes, facts versus guesses. |
| `.claude/agents/bug-tester.md` | Regression triage: failures, skipped routes, unchanged goldens, process cleanup and factual results. |
| `.claude/agents/perf-profiler.md` | Repeatable scenes, three-run median/spread, hardware/settings, logic versus presented fps and thread hotspots. |
| `.claude/agents/release-guard.md` | Publication checks: staged content, game-free package, attribution/licences, submodule and runtime regressions. Apply fork-specific identity rules in AGENTS.md. |
| `tests/regression/COVERAGE.md` | What tests actually cover, remaining routes and shortcuts. A listed test is not a fresh pass. |
| `config/game_state.toml` | Existing researched RAM/save fields and tables. Consult before adding guesses or hard-coded lists. |
| `src/ui/Theme.cpp`, `src/ui/Theme.h`, `src/ui/PauseMenu.cpp` | Existing visual tokens, rendering and navigation to extend. |
| `docs/NEXT_STEPS.md` | Dated upstream session history. Verify any open task, branch or count against current source and tests. |

## How to apply these instructions

1. Read root AGENTS.md and the feature's GOALS.md checks.
2. Read primary sources relevant to the change and inspect the current code/tests.
3. Ask the owner unresolved product questions for that part, then record answers before dependent implementation.
4. Use the relevant upstream role briefs for research or review. Preserve their report format and provide evidence.
5. Record verification in the feature report and update GOALS.md only after its checks pass.

Upstream briefs include the author's own identity, local disc path, historical branch permissions and older platform descriptions. Preserve these as source history. Use this owner's verified identity, actual supported disc location, current checkout and explicit permissions when executing. Do not manufacture session trailers or apply an inherited permission to another account.

CLAUDE.md currently hides device-unsupported/unimplemented options; the UX critic also asks for explanations of unavailable choices. Follow current capability filtering and use visible, controller-accessible reasons for temporary/contextual blockers. Ask the owner if a new flow needs a different product rule.

Reverse-engineer asks for played-through progression tests; COVERAGE.md records existing edited-save and warped-door shortcuts. Keep shortcuts useful for focused research but label them, and require a real route for any end-to-end completion claim.

## Setup verification boundary

Fork setup adds planning documents only. It does not initialise the runtime submodule, extract a disc, build an app or run gameplay tests. Baseline build/gameplay checks remain Phase 0 tasks. Preserve the pinned submodule and existing upstream files until scoped implementation requires a change.
