# Legacy Modding Skills

Migration playbooks for existing Minecraft 1.7.10 and 1.12.2 mods. Use them to move a mod to a supported Gradle template while preserving its mod ID, license, gameplay, and integrations.

## Choose a playbook

| Minecraft | Target | Start here |
| --- | --- | --- |
| 1.7.10 | Forge with GT New Horizons' ExampleMod1.7.10 | [GTNH playbook](skills/Legacy-Modding-Skills/1.7.10/SKILL.md) |
| 1.12.2 | Cleanroom with CleanroomModTemplate and CleanroomGradle | [CleanroomModTemplate steps](skills/Legacy-Modding-Skills/1.12.2/SKILL.md#3a-cleanroommodtemplate-branch) |
| 1.12.2 | Forge with ForgeDevEnv and RetroFuturaGradle | [ForgeDevEnv steps](skills/Legacy-Modding-Skills/1.12.2/SKILL.md#3b-forgedevenv-branch) |

For 1.12.2, decide which runtime the finished mod needs to support before choosing a template. If you need both Forge and Cleanroom, verify each runtime separately.

## Use with a coding agent

Clone this repository somewhere your agent can read it.

```sh
git clone https://github.com/erialsan/Legacy-Modding-Skills.git
```

Open your mod project in the agent and point it to the [main skill file](skills/Legacy-Modding-Skills/SKILL.md) in the clone. Keep the skill directory together so the agent can follow its links to the version-specific and shared instructions.

Example prompt for a Forge 1.12.2 project:

> Read `skills/Legacy-Modding-Skills/SKILL.md` in my Legacy-Modding-Skills checkout. Migrate this Minecraft 1.12.2 mod to ForgeDevEnv and preserve Forge compatibility. Keep the existing mod ID, license, gameplay, and integrations. Report the Gradle checks you ran, whether client or server startup succeeded, and what remains untested.

## What to check after migration

1. Run the build and inspect the packaged JAR for the expected mod ID, version, and resources.
2. Start the intended client and, for server-capable mods, check dedicated-server startup when available.
3. Record the commands, results, and any runtime or behavior you could not test.

The [shared workflow](skills/Legacy-Modding-Skills/common/workflow.md), [dependency reference](skills/Legacy-Modding-Skills/common/reference.md), and [migration examples](skills/Legacy-Modding-Skills/common/examples.md) cover the work common to all three targets.

## Project links

[OKF v0.3 bundle index](index.md) · [Change log](log.md) · [Unlicense](LICENSE)
