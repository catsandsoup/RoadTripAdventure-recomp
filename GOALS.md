# Road Trip Recomp fork: goals and implementation plan

Updated: 11 October 2026

## Purpose

Build on [silentsudin/RoadTripAdventure-recomp](https://github.com/silentsudin/RoadTripAdventure-recomp) to make Road Trip a customisable Adventure and racing sandbox, with an integrated mod workshop, split-screen free roam and online free roam with race invitations.

Keep upstream's build architecture, design language, input conventions and regression workflow. Ship useful additions in this order:

1. Cheats and save editing.
2. Body-shop and paint-shop introduction, randomiser and race customisation.
3. Split-screen free roam.
4. Online free roam and race joining.
5. Integrated mod workshop and content tools.

Phase 0 establishes the baseline needed for those additions. Follow-on research covers town streaming and further platforms.

This document records agreed goals and proposed implementation steps. All fork features below start as **planned**, with unchecked acceptance criteria. Earlier work in the owner's PAL recomp does not establish that a feature works in this USA recomp. Do not mark a feature complete without evidence from this fork.

## Clarification gate before each implementation part

The owner requires questions before implementation to eliminate confusion, assumptions and unsupported claims. Apply this gate to each feature or independently meaningful part, not just once per phase.

1. Inspect relevant code, upstream instructions and existing tests first. Separate verified behaviour, inference and unknowns.
2. Present the intended flow and ask the owner all unresolved product questions needed for that part, in focused batches. Explain constraints found in the code and offer concrete alternatives.
3. Wait for answers before coding behaviour that depends on them. Silence does not approve a default. Continue independent read-only research while answers are pending.
4. Record answers and acceptance examples in `docs/DECISIONS.md`, with the affected feature IDs. Flag proposed details in this plan that the owner has not confirmed; do not turn a proposed implementation into a user decision.
5. Confirm that scope, defaults, persistence, cancellation, edge cases and completion checks are clear. If new ambiguity appears during work, ask before implementing that dependent behaviour.
6. Implement only the clarified part, show the result and its evidence, and repeat for the next part.

Starting question areas, to refine after inspecting the code:

| Part | Questions to resolve before coding |
|---|---|
| Cheats/save editing | One-time cash versus unlimited semantics; supported amounts; naming characters/lengths; whether sandbox edits affect normal saves; exact reset/retention rules; cheat defaults and menu placement. |
| Shop introduction | Which bodies/paints are eligible, including locked originals; who speaks and the dialogue tone; how the player arrives/leaves each shop; body/paint skip behaviour; random-both entry; backtracking; pricing; default enablement; where reset/New Game Plus enters the flow. The ordered name → currency → body shop → paint shop → president → Q's Factory route is already confirmed. |
| Randomiser/races | Which pools change; seed persistence; progression guarantees; target car counts; AI/team rules; rewards and story effects; teammate meaning; replay/ghost compatibility. |
| Split-screen | Shared/personal progress, player-two saves, separate-town travel, concurrent dialogue/shops, pause ownership, split orientation, player collisions and join/leave rules. |
| Online | Target players/platforms, authority, host departure/reconnect, personal/shared progress, race invitation/late-join rules, cheats/collisions, mod compatibility and internet hosting expectations. |
| Workshop | Import/export formats, replacement versus additive content, creator workflow, trust/licensing, package conflicts, reload/removal behaviour and online integration. The workshop is last. |

Read-only setup and planning may proceed before those answers. This gate does not authorise implementation of the entire roadmap.

## Upstream rules and baseline

Read the current upstream files before each implementation phase, including applicable nested instructions:

- [CLAUDE.md](https://github.com/silentsudin/RoadTripAdventure-recomp/blob/main/CLAUDE.md): architecture, build, hooks, UI and runtime conventions.
- [docs/DEVELOPING.md](https://github.com/silentsudin/RoadTripAdventure-recomp/blob/main/docs/DEVELOPING.md): builds, regression tests and repository layout.
- [choroq-style-critic](https://github.com/silentsudin/RoadTripAdventure-recomp/blob/main/.claude/agents/choroq-style-critic.md): visual review requirements.
- [jobs-ux-critic](https://github.com/silentsudin/RoadTripAdventure-recomp/blob/main/.claude/agents/jobs-ux-critic.md): interaction and flow review requirements.
- [README.md](https://github.com/silentsudin/RoadTripAdventure-recomp/blob/main/README.md): player-facing features and installation.

Fork: [catsandsoup/RoadTripAdventure-recomp](https://github.com/catsandsoup/RoadTripAdventure-recomp). Setup baseline: upstream commit `0a4e8d3afa933b5df031e9b6afa3f95ece7ea0e1`. The upstream snapshot has no root AGENTS.md; this fork adds one to make its rules discoverable by Codex. Preserve the upstream CLAUDE.md and agent briefs. Re-read current instructions after upstream updates; links above follow the moving default branch. See `docs/UPSTREAM_INSTRUCTIONS.md` for the source map and fork-specific adaptations.

### Architecture and build requirements

- Target the upstream USA disc, SLUS-20398. Research USA addresses and layouts; do not copy PAL hooks or frame-rate assumptions from the old GOALS.md.
- Preserve first-launch extraction, recompilation and compilation on the player's device. The distributed app contains no game code or disc data. In particular, discard the old goal of compiling disc-derived game code into the distributed app.
- Keep generated code, disc images, ELFs, extracted assets, captures, saves and other disc-derived outputs out of commits and releases. Follow upstream's hash-only golden-frame convention.
- Use the existing pipeline, presets, SDK bundling, platform abstractions and pinned submodules. Edit `config/roadtrip.base.toml`; regenerate `config/roadtrip.toml` through analysis.
- Add game hooks through `src/game/overrides.cpp` and the existing registration mechanisms. Extend existing settings and UI services rather than introducing another menu framework.
- Use `src/platform/Host.h` and upstream input abstractions for platform access. Preserve runtime/game-library ABI and build-ID invalidation when changing shared interfaces.
- Preserve upstream renderer, audio, input and save-state behaviour with the new features disabled. Keep graphics changes independent of simulation changes.
- Follow upstream runtime-submodule and release-review procedures. Historical permission recorded in upstream notes to push its author's branches does not grant permission to push this fork or create upstream PRs.
- Preserve the existing upstream features: graphics options, texture packs, controller mappings, rumble, persistent settings, save states, second-screen support and first-run setup. Measure them rather than rebuilding them as fork milestones.

### Design and UI/UX requirements

- Build player-facing pages within the existing UI and `src/ui/Theme.cpp` conventions. Match Road Trip's blue bevelled menu panels, bright selection bar, gold horn cursor, rounded forms, chunky type and legible outlined text. Use the warm-orange/olive dialogue treatment where appropriate.
- Reuse theme colours, spacing, radii, button glyphs and navigation components. Keep the panel size and position stable as focus moves. Avoid stock developer-tool styling and dense player-facing tables.
- Support gamepad-only completion of each flow, then keyboard and mouse, and touch where upstream supports it. Honour each controller family's confirm/back mapping. Include controller-driven text entry for player and currency names.
- Keep text readable on a TV and a six-inch handheld: approximately 22 px or larger at 1080p, scaled with window height; text contrast at least 4.5:1 and non-text controls at least 3:1. Test 640×448, 1080p, 4K and ultrawide without stretching artwork or clipping labels.
- Use plain labels, sensible defaults, short hints and immediate feedback. State restart requirements at the point of change. Place advanced controls on focused pages so ordinary play needs no extra setup.
- Follow current CLAUDE.md capability filtering: hide features that a device cannot support or that have not been implemented. Explain temporary blockers, such as an incompatible save or unavailable multiplayer action, with a controller-accessible reason. Reconcile any changed upstream guidance before coding.
- Preserve existing menu shortcuts and Title/Pause entry points. In solo play, opening a menu pauses the game and blocks gameplay input; closing restores the prior context. Multiplayer requires the explicit local-menu behaviour in Phases 3 and 4.
- Capture local screenshots of new pages beside reference game menus. Run the upstream `choroq-style-critic` and `jobs-ux-critic` reviews after UI changes, address their must-fix items, and record the outcome. Do not commit reference images from the disc.

### Save and feature behaviour

- Make gameplay changes opt-in. An Original profile uses upstream defaults with fork gameplay modifiers disabled; it does not replace upstream presentation defaults with those of the old PAL project.
- Preserve the original memory-card format for original fields. Store added content identifiers, mod state and fork-only metadata in a versioned sidecar associated with the correct save.
- Back up before destructive edits, reset or migration. Write changes atomically and provide restore. Distinguish memory-card saves from save states; detect or invalidate stale states that conflict with edited progress or mods.
- Turning off a modifier stops future effects. It does not undo earned rewards or prior save edits. Offer backup restore or an explicit reset for those changes.
- Store user-facing preferences through upstream settings, with migration and recovery for damaged configuration. Keep development saves separate from normal Adventures.

## Phase 0: establish the fork baseline

### Implementation

Record the fork URL and upstream revision. Build through upstream's pipeline on macOS first, then check supported Windows and Android builds as changes touch those platforms. Run the existing unit and regression suites. Map the USA progress fields, car catalogues, money/name fields, race entrants and safe mutation points using upstream hooks, `tools/save_parser.py`, `config/game_state.toml` and reference runs.

### Checks

- [ ] Record the upstream revision, build environment, supported platforms and known baseline failures.
- [ ] Run existing regression routes for Adventure, towns, shops, conversations, Quick Race, two-player races, options, save states and the ending; distinguish passes, failures and skipped routes.
- [ ] Confirm a clean app build needs no disc; first launch builds from a supported player-owned disc.
- [ ] Check the packaged executable for generated `sub_00` symbols using upstream's `nm`/`llvm-nm` check: zero game-function symbols in the distributed executable.
- [ ] Establish three-repeat deterministic state/frame comparisons with upstream test settings, plus real-time performance measurements on named hardware.
- [ ] Write a field/hook map with evidence and a save-backup/restore prototype before applying edits.

Exit gate: the team can reproduce the baseline, restore a save and identify where each Phase 1 edit belongs.

## Phase 1: cheats, save editing and sandbox controls

Implement small, verified mutations first. Add a Cheats/Sandbox page using upstream UI components, validated setters at safe game-thread points, backups and profile persistence. Distinguish one-time actions from continuously enabled cheats.

| ID | Feature | Completion checks |
|---|---|---|
| C1 | Money: 99,999, custom amount and unlimited | Validate against researched storage and shop limits; reject overflow and invalid input. Verify cash display, buying, selling, rewards, save/load and relaunch. Unlimited money must survive spending; switching it off resumes ordinary accounting. |
| C2 | Rename player and currency | Provide controller text entry and supported character/length limits. Verify dialogue, menus, notebook and save/load; test empty, maximum-length and unsupported input without corrupting adjacent fields. |
| C3 | Change current body and paint | Preview the choice; apply at a safe point. Verify town, shop, race, cutscene and saved appearance; reject missing bodies or invalid paints. |
| C4 | Unlock bodies, parts and locations | Support individual categories and unlock-all. Map inventory, shop, visited-area, road/gate, licence and event flags separately. Verify access and purchases in each town; preserve story consistency or state the sandbox-only limitation before applying. |
| C5 | Progress reset | Offer complete reset and selective story reset with explicit retention choices for car, parts and money. Back up first; cancel changes nothing. Verify intro, licences, quests, stamps, race records, teammates and location flags follow the selected policy; reload and restore the backup. |
| C6 | New Game Plus | Start a fresh story with selected retained unlocks using a documented retention matrix. Verify early quests and late-game event gates, saving and restoring; keep the source Adventure intact. |
| C7 | Vehicle performance | Separate top speed, acceleration, grip, braking and steering. Publish researched safe ranges; verify each control on a fixed course, high-speed roads, slopes, collisions and water/air parts where applicable. Defaults reproduce baseline physics; timers, dialogue and music keep their normal rate. |
| C8 | Profiles and presets | Ship Original, Sandbox and Randomised Adventure profiles as their features become available; support named custom presets. Round-trip settings through restart and update, show active modifiers and handle missing mods without silent substitution. |
| C9 | Garage presets | Save and restore body, paint and fitted parts. Check ownership/availability, preview changes, and retain presets across saves and relaunch. Report missing add-on parts or bodies. |
| C10 | Recovery tools | Restore a backup, recover a stuck car and return to a safe town spawn. Verify on roads, interiors and race contexts; reject unsafe actions with a reason, and preserve quest state and valid saves. |

Phase gate:

- [ ] Complete each C-item above with a focused regression and manual controller check.
- [ ] Open/close menus without leaking input, moving focus unexpectedly or changing unrelated save fields.
- [ ] Demonstrate defaults-off state equivalence and successful recovery from interrupted writes and damaged sidecars.
- [ ] Pass both upstream UI reviews and relevant build/regression checks.

## Phase 2: body-shop and paint-shop introduction, randomiser and race customisation

### Implementation

Build the new Adventure introduction using the game's body shop and paint shop, then a versioned seeded randomiser, then configurable race rules. Selecting existing bodies and paints provides customisation before the workshop exists; this phase must not depend on an asset importer or mod manager. Research and remove fixed-size assumptions before raising entrant counts. Carry each entrant's identity through the entire event, including teammates and results.

### New Adventure sequence

**New Game → enter player name → enter currency name → body shop → paint shop → president scene → Q's Factory.**

Transport the player into the actual body-shop area to choose their starter body, then transition to the actual paint-shop area to paint it. Add original interaction dialogue that introduces each choice and guides the player onward, presented through the game's dialogue system and shop UI. Reuse the shop interiors, previews, camera behaviour and interaction patterns; this is a playable introduction within the game, not a standalone host-menu chooser. Research the USA shop tasks and initialization dependencies before entering them ahead of the original president scene.

Make body and paint choices independently skippable and randomisable. At each stage, let the player choose manually, randomise from an eligible pool, or skip and accept the current/default selection. Provide a random-both route for repeat or roguelike-style starts. A paint skip retains valid paint for the chosen body; a body change must not leave an incompatible colour. Explicit seeded randomisation follows R2/R3; skipping does not silently reroll a confirmed choice. When the new introduction is disabled, preserve the original new-game route.

Treat these visits as starter customisation rather than normal purchases: they must not deduct starting money or grant accidental story progress. Pass the final body and paint into the president scene and Q's Factory using the game's valid save fields and initialization path.

### Feature checks

Randomisation scope starts with explicitly selectable pools for starter bodies/paint, opponents and race configurations. Extend to parts, shops, rewards and progression only after documenting dependencies and preventing impossible Adventures. The precise supported pools are a research decision; do not imply an existing randomiser can already do this.

| ID | Feature | Completion checks |
|---|---|---|
| R1 | Body-shop and paint-shop introduction | Follow the specified name → currency → body shop → paint shop → president → Q's Factory sequence. Use actual shop areas, shop selection/preview UI, original interaction dialogue and area transitions. Complete manual/manual, manual/random, random/manual, random/random and each skip combination using only a controller. Verify final appearance in the president scene, Q's Factory, free roam and save/relaunch; preserve starting cash and unrelated progress. Disabled introduction reproduces upstream's original route and assignment for the same inputs. |
| R2 | Controlled randomiser | Expose seed, included categories, eligible pools and exclusions. Same algorithm version, seed and settings produce the same assignments across three runs; distinct seeds produce variation. Save the rules/version, prevent empty pools and validate progression dependencies. |
| R3 | Shareable seeds | Export/import a small seed-and-rules package without game assets. Verify reproduction and show incompatibilities for changed algorithm versions, disc regions or missing mods; never substitute silently. |
| R4 | Smaller and larger grids | Research limits across car storage, AI, collisions, cameras, rendering, HUD/minimap, dialogue, replays, results and saves. Offer only tested counts, including fewer than baseline and more than baseline. At the minimum, original count and tested maximum, finish races and return to Adventure without invalid identities, memory corruption or broken results. |
| R5 | More teammates | Map teammate roster, recruitment, dialogue, team scoring and World Grand Prix assumptions. Validate expanded rosters through selection, event entry, results, conversations, saving and loading; disabling the feature must handle an expanded saved roster explicitly. |
| R6 | Custom race builder | Choose course, lap count, opponents, grid size, AI difficulty, allowed parts and team rules. Verify enforcement, countdown, finish order, ties, AI completion, rewards and return flow. Treat custom events separately from story progression unless an explicit policy enables it. |
| R7 | Ghosts and shared challenges | Record/replay a vehicle ghost with course, physics, mod and rules metadata. Verify timing and trajectory on repeat playback; ghosts do not collide or alter results. Import/export challenges without disc data and label incompatible runs. |

Phase gate:

- [ ] Capture each new-intro area, dialogue, choice and transition for both upstream UI reviews; verify game styling, button glyphs, focus, sound and loading/fade behaviour.
- [ ] Test back/cancel, randomise again, confirm, skip, reset/New Game Plus entry and interrupted/resumed setup; choices must not duplicate visits, charge cash, reroll on reload or skip the president scene. Verify no setup-car placeholder leaks into cutscenes.
- [ ] Run three-repeat seeded intro tests, all body/paint compatibility cases and defaults-off original intro regressions.
- [ ] Run start → race → results → replay → retry → Adventure return → conversation → save/load at each supported grid boundary.
- [ ] Test Quick Race, Adventure races and team events; document context-specific limits instead of claiming one global maximum.
- [ ] Publish entrant-count/performance results on named hardware and reject settings above verified limits.
- [ ] Keep baseline races unchanged with modifiers off; pass both UI reviews.

## Phase 3: split-screen free roam

### Implementation

Research upstream's existing two-player race path as a starting point. Prototype two independently controlled cars and cameras in one town before expanding to interiors, transitions and the full Adventure. Research whether the game can maintain players in separate areas or needs separate world contexts; record the architecture and limits before promising unrestricted separation.

Design decisions to resolve in the prototype: personal versus shared progress, player-two profile ownership, simultaneous conversations/shops, split orientation, audio listener mixing and joining/leaving during transitions. These remain design choices, not completed decisions.

| ID | Feature | Completion checks |
|---|---|---|
| L1 | Two-player free roam | Give each player independent car, input, camera and viewport. Drive apart, collide and regroup without cross-control or camera ownership errors. Support horizontal/vertical layouts where readable. |
| L2 | Join/leave and controllers | Join through a controller-friendly flow, select a profile/car, handle unplug/reconnect and leave without ending player one's Adventure. Verify controller assignment and per-player glyphs. |
| L3 | World travel and interactions | Resolve separate town/interior loading and independent NPC/shop interactions. Test one player talking or buying while the other drives, both choosing doors, and different-area travel. State any prototype restriction; unrestricted free roam remains pending until verified. |
| L4 | Menus, HUD and audio | Keep each HUD readable and focus scoped to its owner. Define which actions pause everyone and which open a local overlay; block input only where intended. Test dialogue, maps, radio and competing sound sources. |
| L5 | Shared races | Both players enter a race from free roam, with selected AI/team/grid rules, finish, see correct results and return to their world positions/profiles. Test cancel, retry, one player leaving and different finish times. |
| L6 | Progress and persistence | Implement the documented ownership policy for money, quests, unlocks and rewards. Save/reload both profiles and sidecars, restore a backup and prove unrelated single-player saves remain intact. |

Phase gate:

- [ ] Complete a 30-minute two-player session covering town travel, shops, conversations, races and save/reload without hangs, input leaks or corruption.
- [ ] Target normal USA logic speed and 60 real game frames/s on declared supported hardware at the documented split-screen settings. Report frame times and low-percentile rates; generated frames do not count as simulation performance.
- [ ] Pass solo and existing two-player race regressions, plus both UI reviews for join, play and leave flows.

## Phase 4: online free roam and race joining

### Implementation

Prove synchronisation with two peers on a LAN, then add internet joining. Write an architecture decision for authority, simulation replication, prediction/reconciliation, world instances and race transitions. A host-authoritative design is a candidate; verify the game-state hooks before selecting it.

Start with two players and publish a verified player limit after measurement. Keep relay/services, hosting costs and account requirements explicit; select infrastructure after the LAN prototype. Preserve local play when online services are unavailable.

| ID | Feature | Completion checks |
|---|---|---|
| N1 | Sessions and joining | Create a private session, invite/join, show progress/failures and leave. Support late joining in a supported free-roam state; reject incompatible app/protocol/disc versions before entering. Verify reconnect and host departure with a documented policy. |
| N2 | Synchronous free roam | Synchronise cars, appearance, motion, collisions and relevant world state. Exercise towns, roads, interiors and separate-area travel. Test correction after divergence, connection loss and scene loading; no stale cars or duplicated entities. |
| N3 | Free-roam race invitations | Invite nearby/session players, accept/decline, assemble the grid and fill empty slots with AI according to host rules. Late session joiners can join the next race; mid-race participation needs separate validation. Synchronise countdown, finish order, laps, results/rewards and return to free roam. |
| N4 | Host rules | Host controls cheats, performance limits, eligible parts, collisions, team/grid settings and personal/shared progress mode. Show rules before joining; clients cannot apply prohibited mutations. Test changes between events and lock rules during races. |
| N5 | Rules and content compatibility | Before joining, compare app/protocol versions, built-in physics settings, randomiser versions, race rules and supported existing content overrides. Reject incompatible gameplay settings and explain what to change; allow only verified cosmetic differences. Build this handshake without depending on the future workshop. Extend it with M12 package manifests in Phase 5. Do not send disc-derived assets to other players. |
| N6 | Progress and saves | Attribute purchases, money, quests and rewards according to session policy. Test simultaneous actions, disconnect during reward/save and reconnect without duplication or loss. Keep solo Adventures and backups separate from shared-session state. |
| N7 | Online menu behaviour | Options and local overlays do not pause the remote world. Apply this rule to workshop and photo mode when Phase 5 adds them. Block local gameplay input where needed, show relevant ongoing-session status and defer unsafe changes. Test menu exit while the player moves or changes area remotely. |
| N8 | Network resilience and trust | Validate messages, identifiers, sizes and requested actions at the authority. Test at 0/50/100/200 ms RTT, jitter and 1%/5% packet loss; publish the supported envelope. Run multi-peer soak, repeated joins/leaves and malformed-message tests without crashes, invalid saves or runaway memory. |
| N9 | Platform compatibility | Verify same-version cross-platform joining between each supported pair. Until a pair passes synchronisation, rules, races and persistence checks, advertise only the combinations tested. |

Phase gate:

- [ ] Complete a 60-minute online session including late joins, travel, shops, races, disconnect/reconnect and persistence; compare authoritative outcomes across peers.
- [ ] Publish supported player counts, network conditions, hardware, frame-time results and known limits.
- [ ] Pass upstream solo/local multiplayer regressions and both UI reviews for hosting, joining, invitations and errors.
- [ ] A player can join a friend, drive together, complete a race and return to free roam through the UI with no developer commands.

## Phase 5: integrated native mod workshop

### Implementation

Integrate a maintained fork of [mholeys/roadtrip-choroq-tools](https://github.com/mholeys/roadtrip-choroq-tools) into the application's mod workflow. Resolve permission/licensing for code reuse and distribution first, pin the source revision and preserve notices. Port or adapt the useful format/import/export code to bundled native components with the existing UI; the player should not need Python, terminal commands or an external extractor to install and use mods. External creation tools such as Blender may remain optional.

The current tools document model/texture extraction and car modding. They describe incomplete course/field understanding and collision-property export limitations. A complete track pipeline needs additional work; model extraction alone does not establish that a playable track can be imported.

Implement the workshop after split-screen and online free roam. Existing-body/paint customisation and built-in gameplay controls must remain usable without it. Build read-only browsing/export first, then validated replacement imports, package management, hooks and additive content. Reuse upstream texture hashes, texture-pack handling and audio exporters where they already solve the problem.

| ID | Feature | Completion checks |
|---|---|---|
| M1 | Asset browser and previews | Browse local cars, textures, models and supported sounds with provenance and format support shown. Compare previews with in-game assets; unsupported files report a useful error. Cancel/background long work without freezing play or menus. |
| M2 | Local exporters | Export supported car/world/course geometry and textures, and decoded audio to WAV with manifests. Record source, format, omissions and failures; explain missing collision properties. Verify sample exports in an external viewer and compare sound with playback. Keep disc-derived exports local. |
| M3 | Replacement import and fast iteration | Validate replacement cars, textures, models, sounds and text where supported. Preview, apply and restore originals without patching the player's disc. Hot-reload only at verified safe points; state when a reload is needed. Reject malformed inputs without damaging saves. |
| M4 | Mod packages and manager | Provide install/remove, per-mod enable/disable, load order, versioned manifests, dependencies and conflict reports. Resolve failures in the UI. Test archive path traversal, missing dependencies and incompatible versions; restrict data mods to their managed paths. |
| M5 | Function hooks and extensibility | Document versioned pre/post/replace hooks using named functions and verified USA addresses; ship an original sample mod such as a speed display. Handle ABI/version mismatch. Native code mods require an explicit trust model; do not promise folder confinement for arbitrary native code. |
| M6 | Additive cars | Register a new vehicle alongside original entries, with body, paint, preview and selection. Verify free roam, races, collisions, save/reload and disabling/removing the mod with a defined fallback. |
| M7 | Additive tracks | Document geometry, collision properties, spawns, AI routes, checkpoints, camera, minimap and catalogue requirements. Import one new track and finish a race with humans and AI; validate wrong-way detection, laps, results, replay and unloaded-mod handling. |
| M8 | New upgrades and parts | Add engine, tyre and other supported part types to shops, catalogue, text and stat calculation. Verify purchase/equip, money, UI and measured handling changes. Keep added identifiers in sidecars; original saves load unchanged. |
| M9 | Visual overrides | Per-environment FOV and draw-distance/culling controls plus texture replacements. Compare checkpoints across town, field, race and interior; changing presentation must leave simulation state unchanged. |
| M10 | Debug controls and developer menu | Opt-in telemetry, coordinates, verified speed and render-only free camera; include parts unlocks and cash controls. Compare readings with game state, and prove free camera does not alter simulation. Route developer mutations to separate development saves. |
| M11 | Photo mode | Hide HUD, move the camera, pause solo play and save a local image. Restore camera/input on exit; verify resolution/aspect handling and no simulation mutation. Multiplayer photo mode must not pause other players. |
| M12 | Mod compatibility metadata | Compute a versioned manifest of gameplay content/rules, separating cosmetic changes. Detect different physics, parts, tracks, hooks and randomiser versions. Test missing/conflicting packages locally and extend the Phase 4 compatibility handshake with this manifest. |

Phase gate:

- [ ] From a clean installed app, install, enable, preview, use, disable and remove sample mods through controller-accessible flows.
- [ ] Prove one new car, track and part work beyond previews; document any narrower format support.
- [ ] Restore the unmodified experience and handle saves whose add-ons are unavailable.
- [ ] Complete licensing, packaging, malformed-input and upstream UI reviews; distribute no extracted game assets.

## Follow-on goals and research

| ID | Goal | Completion checks |
|---|---|---|
| F1 | Fewer loading boundaries | Map town/field transition triggers, files, memory, timing and world assumptions. Produce a feasibility report for preloading/streaming, including multiplayer. Only claim a seamless-world feature after a separate prototype proves transitions, NPCs, collisions and saves. |
| F2 | Remaining quality-of-life ideas | Audit upstream support for fast loading, skip logos, language selection, memory-card import/export and automatic backups. Implement verified gaps; language requires supported region/text assets, not an assumed PAL language switch in the USA game. |
| F3 | Further platforms | Maintain upstream macOS/Windows/Android support as additions land. Research Linux/Steam Deck and browser feasibility as later ports, with build, input, storage, mod and multiplayer checks for each. |

## Evidence and completion policy

A checkbox means implementation and its checks passed in this fork. Research reports complete research tasks only; prototypes do not complete player-facing features.

For each feature, record:

- Feature ID, fork/upstream revisions, platform, hardware and settings.
- Hook/layout evidence, decisions and known limits.
- Tests run, results and links to original-code changes or reports; keep disc-derived artifacts local.
- Manual controller-flow and screenshot-review outcomes when UI changes.
- Defaults-off comparison, save/reload/restore result and relevant performance measurements.

Use upstream's existing checks as applicable:

```sh
# App build; no ROM required to build the application.
python3 scripts/pipeline.py bootstrap build

# Unit and disc-dependent checks; set RT_ROM to your own supported disc.
ctest --preset macos-release

# Existing gameplay regressions, then focused fork regressions.
python3 scripts/regress.py
python3 scripts/regress.py --perf
```

Check current command usage in upstream docs/scripts before running. Use its platform-specific pipeline/presets for Windows and Android. Do not update goldens to conceal a regression; explain intended changes and review the affected frames. Run release-guard before commits/pushes and preserve bundled licence notices.

Release acceptance requires a clean installation, player-owned disc setup, valid save migration, accessible menu flows, relevant upstream regressions, feature-specific checks and a manual play session. Record failures and unsupported cases explicitly. Do not claim cross-platform, maximum-grid or multiplayer support beyond measured evidence.

## First implementation milestone

After Phase 0, deliver the themed Cheats page with **99,999/custom/unlimited money**, **player/currency rename**, and **backup/restore**. Verify these through new Adventure, Continue, shopping, dialogue and save/relaunch. Then expand through the remaining Phase 1 items before the Phase 2 body-shop → paint-shop introduction and randomiser. The mod workshop follows split-screen and online play as Phase 5.
