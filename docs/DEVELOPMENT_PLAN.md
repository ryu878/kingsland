# Grey Eye — development plan

Team Lead owns scope. Gameplay milestones use the full pipeline in [team-workflow.mdc](../.cursor/rules/team-workflow.mdc): Programmer, QA, Critic, Judge, at most seven iterations. The counter is [AGENT_REVIEW_LOG.md](AGENT_REVIEW_LOG.md).

No milestone below is permission to write the whole mod in one pass.

## Milestone 0 — foundation (current)

Deliver the five agents, two rules, six skills, and these design docs. No Gradle project and no gameplay code.

Done when:

- The static rows S1–S8 in [TEST_PLAN.md](TEST_PLAN.md) have been checked.
- A trial pass of the pipeline has a Judge decision recorded in the review log.
- The user has a proposed milestone 1, and milestone 1 has not been started unless they ask.

Manual play is out of scope here. Accepting milestone 0 does not mean the game runs.

## Milestone 1 — toolchain scaffold

Generate the Fabric 26.3 project from the pins in [TECHNICAL_DESIGN.md](TECHNICAL_DESIGN.md). Empty common and client initializers, `fabric.mod.json`, split source sets, JUnit wired for an empty `logic` package, `.gitignore` from the Fabric template.

Done when `./gradlew build` and `./gradlew test` exit 0, `src/main` does not reference client classes, and the resolved Loom build is written into [DECISIONS.md](DECISIONS.md). Loom's split is `main` plus `client` only. The dedicated server runs from `main`. There is no third source set. No items, blocks, or commands.

When copying the example build script, delete the `license` value `CC0-1.0` and the `jar { from("LICENSE") }` copy. ADR-010 forbids inheriting that license. Do not add a `LICENSE` file to satisfy the example task.

This is the first coding stage. It starts only after the user accepts this plan.

## Milestone 2 — pure rules

Implement `dev.greyeye.logic` for generators, wounds, stun, roles, phase locks, and win checks. Unit rows U1–U18 that do not need a config loader may land here. Config file parsing can be a focused follow-up inside this milestone if the build stays green.

Done when the unit tests for the included rows pass and no Minecraft class is imported from `logic`.

## Milestone 3 — world state and roles

`SavedData`, full snapshot on join, `/greyeye role set`, phase command, and a dedicated server that keeps state across a restart. Still no pretty models.

## Milestone 4 — survivor abilities

John's tag and break speed, Fritz's flashlight with optional client lighting, Felix's repair channel on a stand-in block, James's server flare and stun. Each ability is server-checked.

## Milestone 5 — generators, gate, winners

Five generator block entities, fuel item, sabotage, gate channel, team win, killer win. World reactions may use vanilla sounds.

## Milestone 6 — killer and director

Attack windup, footsteps, the rest of the `/greyeye` command list, scene save/load, `take`, subtitles, and data-driven events. Russian strings for the prologue keys in [STORY.md](STORY.md).

## Milestone 7 — film pass

Custom audio where it matters, balance edits only through the config file, and a five-client session. Manual rows M1–M8 are mandatory before Judge can call the game playable. Until that session exists, nobody reports those rows as passed.

## Write ownership

| Area | Writer during an iteration |
| --- | --- |
| `src/**`, Gradle files | Programmer |
| `docs/AGENT_REVIEW_LOG.md`, `docs/DECISIONS.md` | Team Lead |
| QA and Critic text | Returned to Team Lead, not written into source |
| Same file, two agents | Not allowed |

## Risks

- Loom `1.19-SNAPSHOT` can move. Milestone 1 stops if resolution fails.
- LambDynamicLights can change its client API. The flashlight flag must work without it.
- Fabric API type names on 26.3 may not match older tutorials. Programmer reads the resolved sources.
- This environment may not have a Minecraft client. QA then marks manual rows `NOT RUN` rather than inventing a playtest.
- Cursor cannot force the pipeline with a hard scheduler. The always-on rule plus the Judge cap is the control. The parent session has to launch the four subagents itself because a nested subagent cannot launch the rest.

## License

Do not apply the Fabric example's license to gameplay code. Until the user picks a license, new source files carry no third-party license header and are not published.
