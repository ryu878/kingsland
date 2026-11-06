# Grey Eye: The Lake Camp — game design

Player-facing copy is Russian. This document is English and describes behavior. Numbers and API pins live in [TECHNICAL_DESIGN.md](TECHNICAL_DESIGN.md). Story beats live in [STORY.md](STORY.md). If a default disagrees with the technical design, the technical design wins until [DECISIONS.md](DECISIONS.md) changes both.

## Fantasy

Five friends-and-stranger players shoot a slasher at a lakeside camp. Four are the vacationers. The fifth is the killer, played by a person. The survivors restore five generators and open the gate. The modification exists so a director can repeat those scenes without repairing the world by hand.

Tone in the prologue is a cheap, funny trip. The camp turns. By day 3 the road is closed and people in the other cottages are gone. The hunt is the last act, not the opening joke.

## Players

| Seat | Role id | Fantasy in one line |
| --- | --- | --- |
| 1 | `john` | Cuts wood twice as fast |
| 2 | `fritz` | Only person with the flashlight |
| 3 | `felix` | Repairs generators faster |
| 4 | `james` | Flare gun that stuns the killer |
| 5 | `killer` | Human hunter, not a mob |

Roles are unique. The director assigns them before a take. Random shuffle is out of scope until a later milestone. Display name of the killer defaults to `Маньяк` and is a config string.

## Session shape

1. Director loads a scene or sets a phase.
2. Prologue and the first two days play as scenes. Killer attacks and generator completion stay locked.
3. Day 3 shows the camp is cut off. The director then arms the killer and enters `HUNT`.
4. Survivors fuel, repair, and start five generators. Order of fuel and repair does not matter.
5. When all five are running, a survivor can activate the gate. Phase `ESCAPE` allows the win volume.
6. The killer may sabotage a running generator. The gate closes until that generator is refueled and started. Repair progress on it remains.
7. Survivors win when every survivor who is not dead is inside the escape volume while the gate is powered and activated. The killer wins when all four survivors are dead.

John's speed does not repair generators. It is character identity for filmed work and for any later wood use. v1 does not add barricades.

## Wounds

Healthy, then downed, then dead. A finished killer attack moves one step. Downed survivors bleed out. Another survivor revives them with a channel. Downed players cannot use items, so Fritz's flashlight turns off. There is no in-hunt respawn.

Friendly fire does not change wounds. During director-controlled phases, falling and other environmental damage do not either, so a missed jump does not destroy a take. A config flag turns environmental damage on for a non-film session.

## Light

Fritz's flashlight is a cone and the only portable beam. Everyone else uses short-range placed or carried lights (vanilla lanterns and the camp's practicals). The cone renderer prefers LambDynamicLights on the client. Without that mod, the flashlight still toggles in gameplay and falls back to a weaker local light. The server never loads that renderer.

## Fairness

The killer is slightly faster, has a readable attack windup, and makes a sprint footstep other clients hear. The killer does not see survivors through walls and does not teleport unless the director moves them. James cannot refresh a stun. Sabotage is a channel with a cooldown, and it does not undo Felix's repair. The escape is tight and still completable.

## Filming over balance

Balance numbers are hypotheses until a playtest milestone. The director can preload generator progress, reset ammo, move people to markers, and excuse a disconnected actor so a missing player does not block a shot. Those tools are part of the game, not cheats.

## Out of scope for the foundation

No camp geometry, no custom models, no recorded voice, no mod jar. The map is a world plus named markers. The mod stores marker ids and teleports to them.

## Acceptance for this design

A reader can name the winner, the five generator steps, what each role may do, what the killer cannot do, and how a retake restores a scene, without inventing a rule.
