---
name: performance-optimization
description: >-
  Keeps the Grey Eye server tick and network small for five players and a
  director. Use when adding ticks, entity AI, lighting, packets, generator
  updates, or when a feature would scan all players or broadcast every tick.
---

# Performance

Five players do not need a framework. They do need the mod to stay quiet when nothing is happening.

## Budgets

- While no channel is active, the mod does not scan the player list every tick and does not send gameplay packets every tick.
- An active repair, start, sabotage, revive, or gate channel updates at 4 Hz, and only to players tracking that block or the wounded player.
- Flashlight cone math runs on the client that draws it. The server stores a boolean and the holder's id.
- The killer has no AI tick. Attack windup is one scheduled check, not a per-tick pathfind.
- Join sync is one snapshot, not a burst of per-generator packets in a loop that can grow without a cap. Five generators fit in that one snapshot.

## Patterns

Drive generator progress from the block entity or a single server ticker that iterates only the active channel set. When the set is empty, the ticker returns immediately.

Do not raycast the flare on every client. The server entity is the hit. Clients render it.

Do not use dynamic lighting on the server. LambDynamicLights stays client-side.

## What to measure later

When a client exists, sample server milliseconds for the mod with nobody interacting, then with one active repair. Record the numbers in the task report. Until then, do not claim a frame-time win.

## Done check

A new repeating task answers: who ticks it, how often it sends a packet, and what it does when idle. If the idle answer is "broadcast state", change the design before coding.
