# Grey Eye — technical design

Canonical pins, packages, state, network, and config. [DECISIONS.md](DECISIONS.md) records why. [GAMEPLAY_MECHANICS.md](GAMEPLAY_MECHANICS.md) restates player-visible rules and must not fork these keys.

Checked 2026-11-06 against [FabricMC/fabric-example-mod branch 26.3](https://github.com/FabricMC/fabric-example-mod/tree/26.3): `gradle.properties`, `build.gradle`, `settings.gradle`, and `gradle/wrapper/gradle-wrapper.properties`.

## Toolchain

| Piece | Pin |
| --- | --- |
| Minecraft | 26.3 |
| Java | 25 |
| Gradle | 9.8.2 |
| Fabric Loader | 0.20.1 |
| Loom | 1.19-SNAPSHOT |
| Loom plugin id | `net.fabricmc.fabric-loom` |
| Fabric API | 0.165.0+26.3 |
| Mod id | `greyeye` |
| Maven group | `dev.greyeye` |
| Gradle project name | `greyeye` |

26.1 and newer are unobfuscated. Dependencies use `implementation`. Do not add Yarn, `modImplementation`, or the remap-only plugin id used on 1.21.11 and older.

Loom is the SNAPSHOT the official example declares. The first scaffold task resolves it and records the resolved build in [DECISIONS.md](DECISIONS.md). If it does not resolve, stop. Do not pick another Loom.

Re-check [fabricmc.net/develop](https://fabricmc.net/develop/) before generating the Gradle project. Update this table and a decision record together, or leave the pins alone.

## Loader choice

Fabric, one loader. Client and dedicated server share the common source set. Client rendering is a split source set (`splitEnvironmentSourceSets()`). NeoForge is out of scope unless a later decision says a required API exists only there. None does today.

LambDynamicLights 4.13.0+26.3 is a **client optional** integration for the flashlight cone. It is not a hard dependency, because the mod is client-only and a dedicated server must boot without it. Do not depend on Illuminated. Grey Eye owns the flashlight item and the role check.

## Source layout

```text
src/main/java/dev/greyeye/           common, server-safe
src/main/java/dev/greyeye/logic/     no Minecraft imports
src/client/java/dev/greyeye/client/  HUD, flashlight, entity renderers
src/main/resources/                  fabric.mod.json, lang, data
src/test/java/                       JUnit for logic only
```

Packages under common: `network`, `world`, `director`, `role`, `registry`. One common initializer and one client initializer.

`fabric.mod.json` depends on Minecraft `26.3` and Fabric Loader `>=0.20.1`. Entry points: `main` and `client`.

Verify Fabric API class names against the resolved 26.3 sources before coding. Do not paste 1.21 networking samples and hope they compile.

## Authority and persistence

The logical server is the only writer of game state. One vanilla `SavedData` on the overworld stores:

- phase, day index, scene id
- five generators (position, fuel, repair, power, channel progress, sabotage cooldown)
- gate activated flag and gate position
- markers (id to block pos and yaw)
- players by UUID: role, wound, stun end tick, immunity end tick, flare rounds, flashlight on, disconnect deadline, excused flag
- named scene snapshots, keyed by scene id (phase, generators, gate, roles, wounds, stun, immunity, ammo, time, weather)
- id of the scene currently loaded

Write on change and on world save. Load on server start and apply to tracking clients with one full snapshot. Do not also store the same fields in a player attachment.

## Logic types

These types live in `dev.greyeye.logic` and are the only implementation of the rules:

```text
Fuel: EMPTY | FUELED
Repair: BROKEN | REPAIRED
Power: OFF | RUNNING
Wound: HEALTHY | DOWNED | DEAD
Phase: PROLOGUE | DAY_1 | DAY_2 | DAY_3 | HUNT | ESCAPE | CUT
Role: UNASSIGNED | JOHN | FRITZ | FELIX | JAMES | KILLER
```

Generator invariants, enforced by the type, not by comments:

- `RUNNING` only if `FUELED` and `REPAIRED`.
- Fuel insert and repair finish commute. Either order is valid.
- Start is legal only when fueled, repaired, and off.
- Sabotage from `RUNNING` sets power `OFF` and fuel `EMPTY`, leaves repair as it is, and starts the sabotage cooldown.
- An interrupted channel resets that channel's progress only.

Win and lose, pure functions of the snapshot:

- Team win: phase is `ESCAPE`, all five generators are `RUNNING`, gate activation has completed, and every player with a survivor role and wound other than `DEAD` is inside the escape volume.
- Killer win: all four survivor roles are `DEAD`.
- A downed survivor inside the volume still counts as not dead and satisfies the volume. A downed survivor outside blocks the team win.
- Losing power clears gate activation.

Disconnect: a `DOWNED` player with no return within `disconnect.downed_timeout_seconds` becomes `DEAD`. A `HEALTHY` disconnected player remains in the snapshot and still blocks the team win until they return or the director runs `player excuse`.

## Channels

Progress is server-side. Tick rate while a channel is active: 4 Hz to trackers. Idle: no generator packets.

| Channel | Who | Base seconds | Speed |
| --- | --- | --- | --- |
| Repair | any living survivor in range | `generator.repair_seconds` | Felix divides by `felix.repair_speed_multiplier` |
| Fuel insert | any living survivor | instant | none |
| Start | any living survivor | `generator.start_seconds` | none |
| Sabotage | killer | `generator.sabotage_seconds` | none |
| Gate | any living survivor | `gate.activation_seconds` | none |
| Revive | living survivor on a downed ally | `survivor.revive_seconds` | none |
| Killer attack windup | killer | `killer.attack_windup_seconds` | stun cancels it |

Interrupt when the actor moves out of range, releases the action, takes a wound, or the phase locks the action.

John's multiplier applies only to blocks in tag `greyeye:john_fast_break` (logs and wood: `#minecraft:logs` plus stripped log and wood variants). It does not apply to planks, stone, or generators. Break completion is server-side. The client may only predict the crack animation with the same multiplier.

## Combat

Flare: server entity. Look vector from the client is a hint and is rejected when it diverges from the server look. Ammo drops when the shot is accepted, including a miss. A hit on the killer applies stun only when both the stun timer and the immunity timer are clear. The hit does not refresh either timer. When stun ends, immunity runs for `james.stun_immunity_seconds`. While stunned, the killer cannot attack or sabotage, and movement uses a server-set slow.

Killer attack: after the windup, if the target is still in `killer.attack_range`, the target's wound advances one step. The speed multiplier applies only while attacks are armed. Survivors do not advance wounds on each other. Cancel vanilla player damage between participants so a normal melee hit cannot add a second health system on top of wounds. Killer sprint asks clients to play a footstep. No survivor markers on the killer HUD and no outlines on survivors.

Flare fire is rejected unless the phase is `HUNT` or `ESCAPE`. A director `killer stun` can still apply a scripted stun in an earlier phase. The flashlight toggle is allowed in every phase so the car scene can use it as a prop.

Environmental damage is ignored while a director phase is active unless `rules.environmental_damage` is true.

## Network

Confirm the payload registration type against Fabric API 0.165.0+26.3 at scaffold time. Intended messages:

Client to server: `FlashlightToggle`, `RepairStart`, `RepairCancel`, `FuelInsert`, `GeneratorStart`, `GeneratorCancel`, `GateStart`, `GateCancel`, `FlareFire`, `KillerAttack`, `ReviveStart`, `ReviveCancel`, `SabotageStart`, `SabotageCancel`.

Server to client: `FullSnapshot`, `GeneratorDelta`, `RoleSync`, `WoundSync`, `StunSync`, `PhaseSync`, `Subtitle`, `FlashlightSync`.

`FullSnapshot` is what join, scene load, and rejected-request repair use. Deltas are for one change. Do not emit a full snapshot every tick.

## Director commands

Permission level 2. Root `/greyeye`.

| Command | Effect |
| --- | --- |
| `phase <id>` | Sets phase and applies its locks |
| `day <1\|2\|3>` | Story day label |
| `time <ticks\|day\|night\|sunset>` | Server time |
| `weather <clear\|rain\|thunder>` | Server weather |
| `killer <arm\|disarm\|stun\|release>` | `arm` and `disarm` only override `HUNT` and `ESCAPE`. `stun` and `release` can be scripted in any phase |
| `event play <id>` | Runs a data-driven beat |
| `subtitle <targets> <key>` | Server subtitle from a translation key |
| `marker set <id>` | Saves the executor's position |
| `tp marker <id> <targets>` | Teleport for a setup |
| `scene save <id>` / `scene load <id>` | Snapshot phase, generators, gate, roles, wounds, stun, immunity, ammo, time, weather. Does not store positions |
| `take <sceneId>` | Load that scene for another recording |
| `reset scenario [--keep-roles]` | Defaults, optionally keep cast |
| `gate <open\|close>` | Forced gate, still logged as director override |
| `generator <index> status\|set ...` | Inspect or preload a generator |
| `role set <player> <role>` | Unique role assign |
| `player <eliminate\|excuse> <player>` | On-camera death, or remove a missing actor from the win check |

Events are JSON under `data/greyeye/director/events/`, not one class per beat. Dialogue keys are `greyeye.dialogue.<scene>.<nn>` in `ru_ru.json` (recording language) and `en_us.json` (development mirror).

Phase locks: `PROLOGUE`, `DAY_1`, and `DAY_2` disarm the killer and lock generators. `DAY_3` keeps attacks off and generators visible but not startable. `HUNT` arms the killer and enables generator channels. `ESCAPE` allows the win check while all five run. `CUT` freezes channels and disarms attacks.

## Config

Server file `config/greyeye-balance.json`. Synced to clients on join so predicted crack animations and channel UI match. No Cloth Config and no extra library. Missing keys use the defaults below. Unknown keys are ignored and logged.

Initial defaults, not playtested:

```text
john.wood_break_multiplier=2.0
felix.repair_speed_multiplier=1.75
james.stun_seconds=10
james.stun_immunity_seconds=8
james.flare_rounds=3
james.shot_cooldown_seconds=1.5
james.flare_speed_blocks_per_second=24
james.flare_hit_radius=0.3
james.flare_max_range=48
killer.speed_multiplier=1.10
killer.attack_windup_seconds=0.6
killer.attack_cooldown_seconds=1.4
killer.attack_range=3.0
killer.display_name=Маньяк
survivor.downed_bleed_seconds=45
survivor.revive_seconds=6
generator.count=5
generator.repair_seconds=12
generator.start_seconds=3
generator.sabotage_seconds=4
generator.sabotage_cooldown_seconds=45
generator.fuel_cells=1
gate.activation_seconds=5
disconnect.downed_timeout_seconds=60
rules.environmental_damage=false
channel.range=4
killer.stun_speed_factor=0.2
```

Felix repair duration is `generator.repair_seconds / felix.repair_speed_multiplier`. John break time is vanilla time divided by `john.wood_break_multiplier`. Fuel cells per generator: `generator.fuel_cells` of item `greyeye:fuel_cell`.

`channel.range` is the max distance in blocks for repair, start, sabotage, gate, and revive. `killer.stun_speed_factor` scales the killer's movement while stunned.

## World reaction

Each generator has a marker group `greyeye:powered_by_<index>` for camp practicals the map author places. Starting a generator plays one server sound and enables that group. The fifth start plays a single camp horn. Sounds may fall back to vanilla events until custom audio exists. A fallback is an audio stand-in, not a fake generator.

The camp mesh, cottages, and the car are world content, not this mod.

## Idle cost

No per-tick player scan and no per-tick gameplay broadcast when no channel, windup, or bleed is active. Bleed is one deadline timestamp, checked for downed players only.

## Scaffold acceptance

The first code milestone generates this toolchain, runs `./gradlew build`, and stops. It does not add generators or roles yet. See [DEVELOPMENT_PLAN.md](DEVELOPMENT_PLAN.md). Copying the example mod's build script must drop `CC0-1.0` and the `LICENSE` jar entry. See ADR-010. The dedicated server runs the `main` source set. Do not add a third server source set. `splitEnvironmentSourceSets()` only adds `client`.
