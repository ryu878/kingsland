---
name: horror-game-design
description: >-
  Shapes Grey Eye as a five-player asymmetric slasher filmed in Minecraft.
  Use when changing survivor or killer abilities, generator pressure, lighting,
  audio telegraphy, win conditions, or the fairness of the hunt.
---

# Horror game design

The film comes first, and the hunt still has to be finishable. Dread comes from a human killer, a dark camp, and missing neighbors. It does not come from a hidden mob or from rules that make the generators impossible.

Read [GAME_DESIGN.md](../../../docs/GAME_DESIGN.md) and [STORY.md](../../../docs/STORY.md). Numbers stay in [TECHNICAL_DESIGN.md](../../../docs/TECHNICAL_DESIGN.md).

## Cast

- John chops wood faster. That identity does not gate the escape in v1. Do not remove it, and do not pretend it repairs generators.
- Fritz is the only player who can toggle the flashlight. Other light is placed or handheld and short-range. The cone is the camp's only beam.
- Felix repairs faster. Fueling and starting are the same speed for everyone.
- James stuns the killer with the flare gun and cannot chain that stun. Immunity after the stun is part of the fantasy, not a bug.
- The killer is played by the fifth person. No wallhacks, no free teleport, a short attack windup, and a sprint footstep other clients can hear.

## Pressure

Five generators, each fueled and repaired in either order, then started. The killer can stop a running generator by spending a sabotage channel. That clears fuel and keeps the repair, then starts a cooldown. The gate stays open only while all five are running. One sabotage closes it. That is the comeback, not a lockout.

Wounds are healthy, downed, dead. Allies revive a downed friend. Bleed-out kills. Environmental damage is off during director phases so a bad jump does not ruin a take. `rules.environmental_damage` can enable it for a non-film session.

## What not to add

- A zombie or villager that stands in for the killer.
- Random ambient damage that the director did not start.
- A survivor HUD marker for the killer, or a killer outline of survivors.
- Infinite stun, instant unavoidable down, or a sabotage that deletes repair progress.
- Prologue combat. The early days stay unarmed until the director enters the hunt.

## Done check

A new mechanic states who can do it, which phase locks it, what the other role can do about it, and how a take recovers after a mistake. If any of those four is missing, the mechanic is not ready.
