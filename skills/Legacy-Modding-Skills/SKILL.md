---
type: Playbook
name: Legacy-Modding-Skill
title: Legacy-Modding-Skill
description: >-
  Route legacy Minecraft mod build migrations to Forge 1.7.10 with the GTNH
  template or Minecraft 1.12.2 with CleanroomModTemplate or ForgeDevEnv,
  using shared preservation, dependency, and verification rules.
tags: [minecraft, legacy-modding, migration, gtnh, cleanroom]
sources:
  - resource: ./1.7.10/SKILL.md
    title: Forge 1.7.10 GTNH migration
  - resource: ./1.12.2/SKILL.md
    title: Minecraft 1.12.2 template migration
---

# Legacy-Modding-Skill

## Choose the version before changing files

| Existing project / requested target | Load and follow |
|---|---|
| Forge 1.7.10 → GTNH ExampleMod1.7.10 build | [1.7.10 — GTNH](./1.7.10/SKILL.md) |
| Minecraft 1.12.2 → CleanroomModTemplate or ForgeDevEnv | [1.12.2 — CleanroomMC templates](./1.12.2/SKILL.md) |

1. Confirm the Minecraft version from the original build and the requested runtime/template. If they disagree or the target is unspecified, resolve that choice before migration. A build-template migration is not permission to upgrade Minecraft or drop Forge compatibility.
2. Load the [shared migration procedure](./common/workflow.md), then the matching version-specific playbook. The shared procedure owns preservation and verification rules; each version playbook owns its build-tool configuration.
3. For 1.12.2, explicitly select CleanroomModTemplate or ForgeDevEnv using that playbook. Do not combine their plugins, token systems, or property names.
4. Apply the chosen workflow and report build/startup evidence, remaining issues, and untested behavior.

**Done:** one version and template are selected, shared safeguards are applied, and verification matches the claimed runtime.

The versioned playbooks are branches within this repository, not separate Git repositories. Shared knowledge lives under `common/`; version-specific settings belong only to their matching playbook. Use the [bundle index](./index.md) for navigation.
