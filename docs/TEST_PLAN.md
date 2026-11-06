# Grey Eye — test plan

Evidence rules: [game-testing skill](../.cursor/skills/game-testing/SKILL.md). Mechanics under test: [GAMEPLAY_MECHANICS.md](GAMEPLAY_MECHANICS.md). Pins: [TECHNICAL_DESIGN.md](TECHNICAL_DESIGN.md).

A row marked **Unit** is mandatory once `dev.greyeye.logic` exists, and it does not require a client. A row marked **Manual** stays `NOT RUN` until someone records a launched client or dedicated server. A row marked **Static** is a file and spec check.

Milestone 0 (this foundation) executes only the static rows. It does not fail because Gradle is absent.

## Defect block

```text
ID: QA-<task>-<nn>
Severity: BLOCKER | CRITICAL | MAJOR | MINOR
Description:
Steps to reproduce:
Expected result:
Actual result:
Related files:
Verification status: OPEN | CLOSED
```

## Product criteria

These are the game-quality targets. They are not all milestone 0.

| Id | Criterion | Layer | When |
| --- | --- | --- | --- |
| C1 | Five human players can hold the five roles | Manual | after roles exist on a dedicated server |
| C2 | Abilities match John, Fritz, Felix, James | Unit, then manual | logic milestone, then in-world |
| C3 | Server owns phase, wounds, generators, win | Unit and code review | logic and network milestones |
| C4 | Generator changes reach another client | Manual | network milestone |
| C5 | Killer is a player, not a mob | Code review and manual | killer milestone |
| C6 | Team win and killer win match the pure functions | Unit | logic milestone |
| C7 | Director commands change phase, time, weather, subtitles | Manual | director milestone |
| C8 | `take` restores a saved scene | Unit for the snapshot type, manual for teleport | director milestone |
| C9 | Balance file overrides a default without a recompile | Unit | config milestone |
| C10 | `./gradlew build` succeeds | Build | scaffold milestone onward |
| C11 | Rules live in one logic package the rest can call | Code review | every code milestone |

## Static rows (milestone 0)

| Id | Check | Expected |
| --- | --- | --- |
| S1 | Agent files exist and declare `name`, `description`, `model: inherit` | Five files under `.cursor/agents/` |
| S2 | Critic and Judge set `readonly: true` | They cannot be the code writer |
| S3 | Skills exist with `name` and `description` | Six `SKILL.md` files |
| S4 | Workflow rule is always applied and states the 7-iteration cap | `.cursor/rules/team-workflow.mdc` |
| S5 | Pins match across the modding skill and the technical design | 26.3, Java 25, Gradle 9.8.2, Loader 0.20.1, Loom 1.19-SNAPSHOT, API 0.165.0+26.3 |
| S6 | Fuel and repair are not a single forced order | Both orders described as legal |
| S7 | Stun does not refresh, flare locked until hunt | Same rule in technical design, abilities, and gameplay |
| S8 | Links between these docs and the agent instructions resolve | No dangling relative link |

## Unit rows (when logic code exists)

| Id | Behavior | Expected |
| --- | --- | --- |
| U1 | Fuel then repair, and repair then fuel | Both can start |
| U2 | Start while empty or broken | Rejected, state unchanged |
| U3 | Sabotage a running generator | Power off, fuel empty, repair unchanged, cooldown set |
| U4 | Interrupt a repair channel | Progress 0, committed repair unchanged |
| U5 | `RUNNING` with empty fuel | Type refuses to construct or clears power |
| U6 | Flare hit on a clear killer | Stun 10s, then immunity 8s |
| U7 | Second hit during stun or immunity | Timers unchanged, round already spent only if the shot was accepted |
| U8 | Shot in `PROLOGUE` | Rejected, rounds unchanged |
| U9 | John multiplier on a log and on stone | Log uses 2.0, stone uses 1 |
| U10 | Felix repair duration | `12 / 1.75`, other role uses 12 |
| U11 | Fritz toggle by James | Rejected |
| U12 | Four dead survivors | Killer win, even if the gate is open |
| U13 | One healthy survivor outside the volume | No team win |
| U14 | Downed survivor inside, others inside, gate live, phase `ESCAPE` | Team win |
| U15 | One generator drops while the gate is active | Gate clears |
| U16 | Snapshot round-trip | Load equals save for phase, generators, wounds, ammo |
| U17 | Duplicate role | Second assignment rejected |
| U18 | Config override of stun length | Pure function reads 10 only when the file does |

## Manual rows

Do not mark these passed from a unit result.

| Id | Setup | Expected |
| --- | --- | --- |
| M1 | Dedicated server, five clients | Roles stick after rejoin |
| M2 | Two clients watch one repair | Progress and final flags match |
| M3 | James stuns, shoots again inside 10s | One stun, killer can act after immunity |
| M4 | Kill all four | Killer win announcement once |
| M5 | Five generators and gate | Team win only inside the volume |
| M6 | `scene save`, sabotage, `take` | Generator returns to the snapshot |
| M7 | Server without LambDynamicLights | Boot succeeds, flashlight flag still toggles |
| M8 | Restart the dedicated server | SavedData restores generators and phase |

## Performance rows

| Id | Check | Expected |
| --- | --- | --- |
| P1 | Idle server, no active channel | Mod sends no repeating gameplay snapshot |
| P2 | One active repair | Updates at 4 Hz to trackers, not to the whole server list every tick |

## Milestone 0 execution note

Until a Gradle wrapper exists, build and unit commands are `NOT RUN` and that is the correct status for this task. QA must say so. Judge may still accept milestone 0 on static evidence.
