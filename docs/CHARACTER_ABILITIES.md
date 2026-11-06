# Grey Eye — character abilities

Behavior for the five roles. Defaults are copied from [TECHNICAL_DESIGN.md](TECHNICAL_DESIGN.md). If a number here drifts, the technical design wins.

Roles are server data on the player UUID. The wrong role gets a rejection and a snapshot, not a partial effect.

## John (`john`)

Chops wood at `john.wood_break_multiplier` (default **2.0**). The multiplier is vanilla break time divided by that value. It applies only to tag `greyeye:john_fast_break`: logs and wood blocks, including stripped variants. Planks, crafting tables, generators, and stone are unchanged.

The server completes the break. The client may show the crack at the same rate. John does not repair, fuel, or stun faster than anyone else. v1 has no barricade object tied to this speed.

## Fritz (`fritz`)

The only role that can toggle `greyeye:flashlight`. The server stores on or off. While Fritz is downed or dead, it is off. The item does not drop on death in v1, so a corpse still holds the prop. v1 does not add a special inventory rule. Anyone can hold the item as a prop. The toggle is rejected unless the sender is Fritz and healthy.

The cone is client-side. Preferred path: LambDynamicLights dynamic light behavior when that mod is present on the client (4.13.0+26.3). Absent the mod, the client uses a short fallback light and the gameplay flag still works. Dedicated servers do not load the renderer.

Other players use camp practicals and vanilla lanterns. They do not get a beam.

## Felix (`felix`)

Repair channel duration is `generator.repair_seconds / felix.repair_speed_multiplier`. Defaults **12 / 1.75** seconds. Other survivors take the full 12. Fuel insert and generator start are not faster for Felix. Repair shows progress: a server-driven bar at 4 Hz for the duration of the channel. Releasing, leaving 4 blocks, or taking a wound clears progress and does not clear a finished repair.

## James (`james`)

Flare gun `greyeye:flare_gun`. Starting rounds: `james.flare_rounds` (default **3**). Cooldown between accepted shots: `james.shot_cooldown_seconds` (default **1.5**). A round is consumed when the server accepts the shot, including a miss. Rejected shots (empty, cooldown, wrong role, phase lock) consume nothing.

The projectile is a server entity. Speed, radius, and range use the `james.flare_*` defaults in the technical design. Those speeds are unplayed starting values.

A hit stuns the killer for `james.stun_seconds` (default **10**) only if the killer is not already stunned and not in immunity. Stun then immunity (`james.stun_immunity_seconds`, default **8**) do not refresh if James hits again. During stun the killer cannot attack or sabotage and moves at `killer.stun_speed_factor` (default **0.2**). The server tells clients to play the shot and the stun reaction. Refills are a scene load, a director command that sets rounds, or a later map pickup. v1 does not spawn pickups by itself.

## Killer (`killer`)

A player. Configurable display name, default `Маньяк`. Speed multiplier `killer.speed_multiplier` (default **1.10**) while attacks are armed. `killer disarm` removes both the attacks and that multiplier. Attack windup **0.6** seconds, then a hit if the survivor is still within **3** blocks, then cooldown **1.4** seconds. Each landed hit advances healthy to downed, or downed to dead. Sprint plays a positional footstep for other clients.

The killer does not use survivor items, does not see stealth outlines, and does not get a flashlight cone. Sabotage is the generator channel in the gameplay doc. `killer arm` and `killer disarm` only override `HUNT` and `ESCAPE`. Earlier phases keep attacks off, which is how the actor can already be in the world for the car and the first days.

## Shared survivor limits

Living survivors may fuel, repair, start, activate the gate, and revive, subject to phase locks. Downed survivors cannot. Dead survivors are out. No survivor damages another survivor's wound track. Bleed-out is `survivor.downed_bleed_seconds` (default **45**). Revive channel is **6** seconds and is interruptible the same way as repair.

## Quality checks

- John does not break stone faster in the logic tests.
- Fritz-only toggle rejects James even if James holds the item.
- Felix's duration matches the division formula, and a non-Felix repair does not.
- Two flare hits inside one stun-plus-immunity window leave a single stun interval.
- Killer windup cancelled by stun does not advance a wound.
