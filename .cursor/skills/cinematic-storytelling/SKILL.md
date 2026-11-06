---
name: cinematic-storytelling
description: >-
  Operates Grey Eye Director Mode for filmed Minecraft scenes, including
  phases, retakes, subtitles, weather, teleport markers, and scripted events.
  Use when adding director commands, dialogue, scene state, or when a mechanic
  must be enabled or suppressed for a specific story beat.
---

# Cinematic storytelling

Grey Eye is built so a small crew can shoot scenes without hand-editing the world between takes. Behavior: [STORY.md](../../../docs/STORY.md). Commands: [TECHNICAL_DESIGN.md](../../../docs/TECHNICAL_DESIGN.md).

## Phase locks

| Phase | Killer attacks | Generators | Escape |
| --- | --- | --- | --- |
| PROLOGUE, DAY_1, DAY_2 | no | locked | no |
| DAY_3 | no | visible, not startable | no |
| HUNT | yes | active | no |
| ESCAPE | yes | must stay running | yes, while all five run |
| CUT | no | frozen | no |

Attacks turn on when the phase becomes `HUNT`. `killer arm` and `killer disarm` only override `HUNT` and `ESCAPE`. They do not arm `DAY_3`. A scripted `killer stun` can still play in an earlier phase. Loading a scene restores phase, roles if the snapshot has them, generators, gate, time, and weather. It does not restore positions. Use `tp marker` for marks.

## Commands the director actually uses

`/greyeye phase`, `day`, `time`, `weather`, `killer arm|disarm|stun|release`, `event play`, `subtitle`, `marker set`, `tp marker`, `scene save`, `scene load`, `reset scenario`, `take`, `gate`, `generator`, `role set`, `player eliminate|excuse`.

`take <sceneId>` loads that scene and resets cooldowns and flare ammo to the snapshot. `player excuse` removes a disconnected body from the escape requirement so a missing actor does not soft-lock a shot. `player eliminate` is the on-camera death.

## Dialogue

Player-facing lines are Russian, stored under translation keys (`greyeye.dialogue.<scene>.<nn>`), not inside Java. Subtitles are a server message to named players or `@a`. The server does not assume a client-only title manager.

## Events

Scripted events are data (sound id, marker, subtitle key, phase). A Java class per story beat is the wrong split. The director can replay `event play <id>`.

## Done check

After `scene save` and a deliberate mess (generator started, killer stunned, player moved), `scene load` or `take` restores phase, generator flags, and stun. Positions stay where the players are until `tp marker`. If phase, generators, or stun survive the load, the retake is not done.
