---
name: minecraft-modding
description: >-
  Implements the Grey Eye Fabric mod on Minecraft 26.3 with the pinned loader,
  Loom, Java 25, and Gradle from the official example. Use when adding or
  changing Java mod code, fabric.mod.json, Gradle files, registries, source
  sets, or when someone asks which Minecraft or Fabric version to use.
---

# Minecraft modding (Grey Eye)

Canonical detail: [TECHNICAL_DESIGN.md](../../../docs/TECHNICAL_DESIGN.md). If this skill and that document disagree, the document wins.

## Pins

Copied from `FabricMC/fabric-example-mod` branch `26.3` on 2026-11-06. Do not mix in 1.21 artifacts.

| Piece | Value |
| --- | --- |
| Minecraft | 26.3 |
| Java | 25 (`release` 25) |
| Gradle | 9.8.2 |
| Fabric Loader | 0.20.1 |
| Loom | 1.19-SNAPSHOT, plugin id `net.fabricmc.fabric-loom` |
| Fabric API | 0.165.0+26.3 |
| Mod id | `greyeye` |
| Maven group | `dev.greyeye` |
| Root project name | `greyeye` |

Dependencies use `implementation`, not the old `modImplementation` configuration. There is no Yarn line: 26.1+ uses Mojang names.

## Source sets

Use Loom `splitEnvironmentSourceSets()`.

- `src/main/java/dev/greyeye` — common and server-safe code
- `src/client/java/dev/greyeye/client` — rendering, flashlight, HUD
- `src/main/java/dev/greyeye/logic` — no Minecraft imports
- `src/test/java` — JUnit for `logic` only

The dedicated server must start without LambDynamicLights. That mod is client-only. Integrate it from the client source set behind a presence check. Do not depend on the Illuminated mod; Grey Eye has its own role-gated flashlight.

## Build

Wrapper tasks: `./gradlew build` and `./gradlew test`. If a pin fails to resolve, stop. Do not guess another Loom or API version.

Before writing against Fabric API, open the resolved sources for 26.3 and use those type names. Snippets from 1.21 (`modImplementation`, Yarn, `fabric-loom` without the 26.1 plugin id) are not valid here.

## Registries

Register blocks, items, entities, payloads, and sounds through one common initializer. Register client renderers and HUD in the client entrypoint only. The generator block entity stores no rule logic; it calls `dev.greyeye.logic`.

## Done check

- `fabric.mod.json` depends on Minecraft `26.3` and Fabric Loader `>=0.20.1`.
- Common sources do not import `net.minecraft.client`.
- Config defaults match the balance table in the technical design.
