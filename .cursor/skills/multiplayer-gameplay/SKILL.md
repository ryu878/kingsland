---
name: multiplayer-gameplay
description: >-
  Designs and checks server-authoritative multiplayer for Grey Eye's five
  players, roles, generators, stun, and win state. Use when changing
  networking, player roles, synchronization, dedicated-server behavior, or
  when a client could disagree with the server about game state.
---

# Multiplayer gameplay

Five human players. Four survivors (John, Fritz, Felix, James) and one killer. The killer is a player role, not a mob. Roles are unique. Director commands assign them; a survivor item used by the wrong role does nothing.

## Authority

The server owns phase, roles, wounds, stun, flashlight on/off, generator flags, gate, and win/lose. A client packet is a request. Reject it when the phase lock, distance, cooldown, or role check fails, and send the current snapshot back if the client was wrong.

Join and scene load send one full snapshot. Later changes send a delta. Do not sync these on a per-tick broadcast.

## What clients may do

- Predict the local mining crack for John's wood speed, using the same multiplier. The server decides when the block breaks.
- Draw the flashlight cone. If LambDynamicLights is missing, draw the fallback and still obey the server's on/off flag.
- Play sounds and subtitles the server ordered.

Clients may not set `REPAIRED`, apply stun, open the gate, or declare a win.

## Payload split

Client to server: flashlight toggle, repair start/cancel, fuel insert, generator start/cancel, gate start/cancel, flare fire (look vector is a hint), killer attack.

Server to client: full snapshot, generator delta, role, wound, stun, subtitle, phase, flashlight visibility for other players.

The flare is a server entity. The look vector is accepted only when it is close to the server's view of the player's look. Ammo decrements when the server accepts the shot.

## Persistence

One world `SavedData` record holds generators, gate, phase, scene id, markers, and player records keyed by UUID. Write it when those change and on save. Do not split the same fact across an attachment and the world data.

## Exploit checks

- A second flare during stun or immunity does not refresh stun.
- Fuel and repair commute. Neither order is rejected.
- A running generator is impossible unless fueled and repaired.
- Sabotage stops power and clears fuel; it does not clear repair.
- Duplicate role assignment is rejected.
- Survivor-versus-survivor damage does not change wounds.

## Done check

A second test player, or a unit test of the pure rules plus a code path that only mutates state on the server, must show the same generator and stun state after a snapshot. Document `NOT RUN` when no second client was started.
