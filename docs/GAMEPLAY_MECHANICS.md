# Grey Eye — gameplay mechanics

Player-visible rules. Types, packets, and config keys are canonical in [TECHNICAL_DESIGN.md](TECHNICAL_DESIGN.md). Abilities per person are in [CHARACTER_ABILITIES.md](CHARACTER_ABILITIES.md).

## Roles and phase

The director sets five unique roles. `UNASSIGNED` cannot use Grey Eye items. Phase decides what is even legal:

| Phase | Attacks | Generator channels | Team win |
| --- | --- | --- | --- |
| `PROLOGUE`, `DAY_1`, `DAY_2` | off | off | no |
| `DAY_3` | off | off (blocks may be visible) | no |
| `HUNT` | on | on | no |
| `ESCAPE` | on | on | yes, if the gate is live |
| `CUT` | off | frozen | no |

`CUT` remembers progress and does not tick channels. `HUNT` and `ESCAPE` tick them.

## Generators

Five generators. Each has fuel, repair, and power. Fuel and repair do not depend on each other.

Legal examples, both valid:

- broken and empty, then fuel, then repair, then start
- broken and empty, then repair, then fuel, then start

Illegal:

- start while fuel is empty or repair is broken
- power `RUNNING` after sabotage (sabotage forces `OFF` and `EMPTY`)
- a client announcing that repair finished

Sabotage is the killer's channel (default 4 seconds) on a running generator, then a 45 second cooldown on that generator. Repair stays `REPAIRED`. Survivors refuel and start again. They do not repeat the repair unless something else sets `BROKEN`, and v1 sabotage does not.

Each running generator enables its `greyeye:powered_by_<index>` group and plays a start sound. The fifth plays the camp horn once per transition into "all running", not every tick.

## Gate and winners

Gate activation is a 5 second survivor channel and completes only if all five generators stay `RUNNING` the whole time. If any generator leaves `RUNNING`, activation clears and the gate is shut.

Team win, checked on the server when a player's position changes or the gate changes: phase `ESCAPE`, gate activated, all five running, every survivor who is not `DEAD` standing in the escape volume. A downed survivor in the volume counts. A downed or healthy survivor outside it blocks the win.

Killer win: John, Fritz, Felix, and James are all `DEAD`.

There is no third ending in v1. The director can cut away with `phase CUT` without awarding a win. That cut is not a victory.

## Wounds

| From | Cause | To |
| --- | --- | --- |
| `HEALTHY` | killer attack lands | `DOWNED` |
| `DOWNED` | killer attack lands, or bleed deadline | `DEAD` |
| `DOWNED` | revive channel completes | `HEALTHY` |
| any | survivor hitting survivor | unchanged |
| any | fall, fire, mob, while director phases and `rules.environmental_damage` is false | unchanged |

Bleed deadline is 45 seconds from the down, refreshed only by a new down, not by spam. Revive is 6 seconds, range 4 blocks, and resets if interrupted. Downed players cannot toggle the flashlight, shoot, repair, or attack.

A disconnected downed player becomes dead after 60 seconds. A disconnected healthy player stays in the win check until they return or `/greyeye player excuse` removes them for this take.

## Flare and stun

Flare fire is legal only in `HUNT` and `ESCAPE`. Earlier phases reject the shot and do not spend a round. Accepted shot spends one round and spawns the server flare. Hit on the killer:

- if stun and immunity are both inactive: stun 10 seconds, then immunity 8 seconds
- otherwise: impact feedback only, timers unchanged

While stunned, attack and sabotage requests are rejected and speed uses `killer.stun_speed_factor`. Immunity exists so three rounds cannot lock the killer for the whole hunt. A scene load restores the ammo and timers stored in the snapshot.

## Flashlight

Server boolean. Fritz, healthy, holding the item, phase does not matter for the toggle itself, including prologue, because the car scene uses it as a prop. The beam does not reveal the killer through geometry beyond what the client light already does. No wallhack entity list is sent with the light.

## Director recovery

`scene save` / `scene load` / `take` restore phase, generators, gate, wounds, stun, immunity, flare rounds, roles, time, and weather. Marker teleports move bodies and do not change those fields. `reset scenario` returns defaults. `reset scenario --keep-roles` keeps the cast.

Preloading a generator for a close-up is `generator <index> set`, which is a director write through the same logic type, so it still cannot store `RUNNING` with empty fuel.

## Multiplayer

Requests that fail validation do not mutate state. The actor receives a snapshot if their client predicted something else. Another player sees generator and stun changes through deltas. A player who joins mid-take receives `FullSnapshot` before they are allowed to send requests.

## Idle

Bleed and stun are deadlines. Channels tick only while someone is holding one. Nothing in this document requires a per-tick scan of all entities in the dimension.
