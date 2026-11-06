# Architecture decisions

Newest rulings stay at the top of the log of decisions. Team Lead writes this file. A change to a pin or a default updates the matching section in [TECHNICAL_DESIGN.md](TECHNICAL_DESIGN.md) in the same task.

## ADR-001 — Fabric on Minecraft 26.3

**Status:** accepted 2026-11-06.

**Decision:** One loader, Fabric. Minecraft 26.3, Java 25, Gradle 9.8.2, Fabric Loader 0.20.1, Loom 1.19-SNAPSHOT (`net.fabricmc.fabric-loom`), Fabric API 0.165.0+26.3.

**Why:** Those are the versions in the official `FabricMC/fabric-example-mod` branch `26.3` (`gradle.properties`, `build.gradle`, wrapper `gradle-9.8.2`). Minecraft 26.1+ is unobfuscated, so 1.21 plugins, Yarn, and `modImplementation` do not belong on this project. NeoForge would double the source split for no API we are missing.

**Cost:** Loom is a SNAPSHOT because the example declares it that way. Milestone 1 records the resolved build or stops. We do not "try 1.18" to get a green resolve.

**Alternatives rejected:** Stay on 1.21.1 for a larger old mod set. That fights the current template and the 2026 lighting builds that already target 26.3.

## ADR-002 — Server authority in one SavedData

**Status:** accepted 2026-11-06.

**Decision:** Phase, roles, wounds, generators, gate, markers, and named scene snapshots live in one overworld `SavedData`. Clients send requests.

**Why:** Two stores (attachments plus world data) will disagree after a crash. One record can be snapshotted for a retake.

**Cost:** The blob must stay small. Five players and five generators fit. Do not put chat logs or replay video in it.

## ADR-003 — Optional client flashlight

**Status:** accepted 2026-11-06.

**Decision:** Gameplay on/off is server state. The cone uses LambDynamicLights 4.13.0+26.3 when the client has it. There is no hard dependency and no dependency on Illuminated.

**Why:** LambDynamicLights is client-only. A required dependency would break the dedicated server. Illuminated is someone else's item and is not role-gated to Fritz.

**Cost:** Clients without the light mod get a weaker fallback. Document that in QA when a visual check is claimed. The toggle itself is still testable.

## ADR-004 — Composite generator, limited sabotage

**Status:** accepted 2026-11-06.

**Decision:** Fuel and repair are independent flags. Power runs only when both are done. Killer sabotage clears fuel and power, keeps repair, and has a cooldown.

**Why:** The user asked not to force an order. Clearing repair as well would erase Felix's contribution and make five generators a grind the film cannot finish. Keeping repair preserves pressure without a lockout.

## ADR-005 — Stun immunity and wound track

**Status:** accepted 2026-11-06.

**Decision:** Stun is 10 seconds, then 8 seconds of immunity that a second flare does not refresh. Rounds start at 3 and decrement on an accepted shot. Wounds are healthy / downed / dead, not vanilla hearts. Environmental damage is off in director phases unless configured. Flare shots work only in `HUNT` and `ESCAPE`.

**Why:** The user required a 10 second stun and protection from infinite stun. A separate wound track keeps a fall from ending a take. Vanilla damage plus wounds would double-dip; melee between participants is cancelled.

**Cost:** The camp is less lethal to clumsy movement than a pure survival server. That is intentional for shooting. Turn `rules.environmental_damage` on for a non-film session.

## ADR-006 — Director commands, not per-scene Java

**Status:** accepted 2026-11-06.

**Decision:** `/greyeye` at permission level 2, scene snapshots inside SavedData, events as data, dialogue as translation keys. Russian is the recording language.

**Why:** Retakes have to be a command. A class per story beat will drift from the script.

## ADR-007 — Balance file, hypothesis numbers

**Status:** accepted 2026-11-06.

**Decision:** `config/greyeye-balance.json` is the only place gameplay code reads the defaults from. The table in the technical design is the initial file content. Those numbers are not claimed to be fun.

**Why:** The user wants balance changes without a rewrite. Pretending an unplayed 1.75 repair rate is final would lock the next milestone to a guess.

## ADR-008 — Judge cap and a single log writer

**Status:** accepted 2026-11-06.

**Decision:** At most seven iterations per task, defined as Critic, Programmer fix, QA, Critic again, Judge. Judge is read-only and returns the log entry. Team Lead appends [AGENT_REVIEW_LOG.md](AGENT_REVIEW_LOG.md). After `STOP_LIMIT_REACHED` or a terminal `BLOCKED`, only the user reopens the task.

**Why:** Cursor's `readonly: true` flag is what stops Judge and Critic from editing code. Letting them also write the log means two writers. The parent session must launch Programmer, QA, Critic, and Judge itself, because a nested subagent cannot launch the next one.

**Cost:** The pipeline is instruction plus review, not a kernel-enforced state machine. Skipping QA is a Team Lead failure, and Judge treats missing mandatory evidence as not accepted. On 2026-11-06 the Task tool in this session accepted only built-in types (`generalPurpose`, `explore`, and the other listed built-ins) and rejected `programmer`. Until that changes, Team Lead runs the role file through `generalPurpose` and writes the fallback into the review log.

## ADR-009 — John's speed does not gate escape

**Status:** accepted 2026-11-06.

**Decision:** Wood break speed ships in v1. Barricades do not. Escape is generators and the gate.

**Why:** The user specified the wood ability and specified generators as the objective. Tying the win to wood would invent a system they did not ask for. Leaving the speed out would drop a required ability.

## ADR-010 — No license copied from the example mod

**Status:** accepted 2026-11-06.

**Decision:** Gameplay sources do not inherit the Fabric example license. The user chooses the license before any public release.

**Why:** The example license may be permissive and must not be assumed to cover this project.
