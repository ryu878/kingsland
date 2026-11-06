# Grey Eye — story

Recording language is Russian. Mechanics mentioned here are defined in [GAME_DESIGN.md](GAME_DESIGN.md) and locked in [TECHNICAL_DESIGN.md](TECHNICAL_DESIGN.md). Dialogue keys ship in `ru_ru.json`. The lines below are the script, not placeholder lorem.

Four friends buy a 90% discount trip to the camp on the lake Серый Глаз. They drive up laughing. The discount is the joke. The empty cottages are the turn. On day 3 the camp is cut off, and the fifth player hunts them until the generators open the gate.

The killer's on-screen name defaults to `Маньяк`. Who they were before the hunt is not settled in this document. Do not invent a backstory in code.

## Phase map

| Scene id | Phase | Killer attacks | Generators |
| --- | --- | --- | --- |
| `PROLOGUE_CAR` | `PROLOGUE` | no | locked |
| `DAY1_ARRIVAL` | `DAY_1` | no | locked |
| `DAY1_COTTAGE` | `DAY_1` | no | locked |
| `DAY1_ODDITY` | `DAY_1` | no | locked |
| `DAY2_LAKE` | `DAY_2` | no | locked |
| `DAY2_FOREST` | `DAY_2` | no | locked |
| `DAY2_NIGHT` | `DAY_2` | no | locked |
| `DAY3_MISSING` | `DAY_3` | no | visible, not startable |
| `DAY3_BLACKOUT` | `DAY_3` | no | visible, not startable |
| `HUNT` | `HUNT` | yes | active |
| `ESCAPE` | `ESCAPE` | yes | must stay running |
| `CUT` | `CUT` | no | frozen |

`take <sceneId>` loads that row's snapshot. Early scenes do not leak flare ammo as a puzzle and do not run generator channels.

## Prologue — the car

They are loud, young, and not in danger yet.

| Key | Speaker | Line |
| --- | --- | --- |
| `greyeye.dialogue.prologue.01` | John | Я так рад, наконец-то отпуск! |
| `greyeye.dialogue.prologue.02` | Felix | Девяносто процентов скидки! Это вообще законно? |
| `greyeye.dialogue.prologue.03` | James | Надеюсь, коттеджи хотя бы с крышей! |
| `greyeye.dialogue.prologue.04` | Fritz | Если крыши нет, я хотя бы нашёл фонарик. В бардачке. |

Fritz's line plants the flashlight as a joke. It is not a tutorial prompt. No stun, no fuel, no gate.

## Day 1 — arrival

They find the camp, pick a cottage, and notice one joke too many: a light on in an empty unit, a discount poster with the phone number scratched out, the lake too still. The director plays these as subtitles and practicals. The mod does not spawn an NPC to explain them.

## Day 2 — the lake and the woods

Swimming, a walk, other guests who are extras in the world (map builds, not mod entities). At night the director plays a sound event. The killer stays disarmed. Survivors may explore. They cannot finish generators.

## Day 3 — cut off

The neighboring cottages are empty. The car will not leave: the gate is shut and the director has locked the exit. Evidence of violence is set dressing and subtitle keys `greyeye.dialogue.day3.*`, written when those scenes are shot. Then `HUNT` arms the killer.

The survivors' job from here is the five generators and the gate, as in the gameplay doc. The story does not add a second win condition.

## Hunt and escape

The killer is the fifth player. Scripted stings (a horn, a subtitle, a light group) are director events, replayable, and never a substitute for that player. When the fifth generator starts, the camp horn is the world reaction from the technical design. The gate can then be activated. Leaving happens only in `ESCAPE`.

## Retake rules

- Dialogue is keys, so a retake does not require a recompile.
- `scene save` before a shot, `take` after a ruined one.
- A missing actor is `player excuse`, not a deleted UUID and not a silent win.
- An on-camera death is `player eliminate` or a real wound. Do not teleport a dead survivor back to healthy inside a hunt take unless the director loaded a snapshot where they were healthy.

## Consistency

The prologue must not require items that only exist after `HUNT`. Day 3's blackout does not start generator repair by itself; the director changes phase. The discount dialogue is the same offer the design doc calls a 90% trip. No scene asks the killer to be a mob.
