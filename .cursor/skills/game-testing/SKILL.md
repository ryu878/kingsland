---
name: game-testing
description: >-
  Tests Grey Eye with a fixed defect report and separates unit tests from
  unplayed multiplayer checks. Use when writing or running tests, reviewing a
  QA report, deciding if a mechanic was actually verified, or preparing a
  Judge evidence packet.
---

# Game testing

Plan: [TEST_PLAN.md](../../../docs/TEST_PLAN.md). Acceptance evidence is what Judge is allowed to trust.

## Layers

1. **Static**: files, pins, and rule consistency. Proves the spec matches itself. Does not prove the game runs.
2. **Unit**: JUnit on `dev.greyeye.logic`. No Minecraft client. Required for generator flags, stun immunity, roles, and win checks once that code exists.
3. **Build**: `./gradlew build` and `./gradlew test`.
4. **Manual**: a launched client or dedicated server. If it was not started, the status is `NOT RUN`. Do not upgrade a unit test into a five-player result.

## Defect block

```text
ID: QA-<task>-<nn>
Severity: BLOCKER | CRITICAL | MAJOR | MINOR
Description:
Steps to reproduce:
Expected result:
Actual result:
Related files:
Verification status: OPEN | CLOSED
```

- BLOCKER: cannot build, cannot save/load, or the task's main path is dead.
- CRITICAL: wrong winner, role bypass, stun with no immunity, or client-owned world state.
- MAJOR: required behavior wrong, with a filming workaround.
- MINOR: copy or log mismatch, behavior still holds.

## Retest

A closed defect names the command or test that passed after the fix. "Looks right" is not a close.

## Milestone 0

The foundation task is static only. Missing Gradle is not a failure of that task. From the scaffold milestone onward, a code change without `./gradlew test` stays unverified.

## Done check

The report lists commands, counts, and a separate "not verified" list. Manual rows stay `NOT RUN` until a client launch is in the log.
