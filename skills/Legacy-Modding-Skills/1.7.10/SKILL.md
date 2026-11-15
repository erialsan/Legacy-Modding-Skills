---
type: Playbook
name: Legacy-Modding-Skill-GTNH
title: Legacy-Modding-Skill — Forge 1.7.10 / GTNH
description: Migrate Forge 1.7.10 builds to GTNH ExampleMod1.7.10 conventions while preserving identity, integrations, and runtime requirements.
tags: [minecraft, forge, 1.7.10, gtnh, migration]
sources:
  - id: migration
    resource: https://github.com/GTNewHorizons/ExampleMod1.7.10/blob/master/docs/migration.md
    title: Official GTNH migration guide
  - id: porting
    resource: https://github.com/GTNewHorizons/ExampleMod1.7.10/blob/master/docs/porting.md
    title: Official GTNH porting guide
  - id: style
    resource: https://wiki.gtnewhorizons.com/wiki/Code_Style
    title: GTNH code style
---

# Forge 1.7.10 — GTNH

Use only for an existing Forge 1.7.10 project targeting [GTNH ExampleMod1.7.10](https://github.com/GTNewHorizons/ExampleMod1.7.10). Load the [shared procedure](../common/workflow.md) first; it owns inventory, backups, scope, dependency classification, and cleanup. Minecraft 1.12.2 uses the [separate playbook](../1.12.2/SKILL.md).

## 1. Apply the template

Follow the current official guide and inspect the archive before extraction.[^migration] The official overlay is:

```text
https://github.com/GTNewHorizons/ExampleMod1.7.10/releases/download/master-packages/migration.zip
```

Download to a unique temporary directory. Complete the shared backup gate, inspect archive paths, then extract into the intended project root and make `gradlew` executable. Record the template/archive revision or digest because the release asset is mutable.

Keep `build.gradle.kts` with `com.gtnewhorizons.gtnhconvention` as the build entry point. Remove an old Groovy `build.gradle` only after verifying its recoverable backup, so Gradle cannot select the wrong script. The overlay's current contents, not historical assumptions, determine which settings/resources need restoration.

For root-level Eclipse `java/` and `resources/`, normalize the main tree to `src/main/java` and `src/main/resources`. Additional API/submodule roots belong in `addon.gradle` rather than treating the whole main tree as an extra root.

Skip the overlay if the project already has a complete convention build with its real identity. Updating an old convention version is the optional update branch below, not reason to overwrite configured files.

**Gate:** one active entry point, protected originals, and complete source/resource mapping.

## 2. Map identity and custom behavior

Use the selected template revision's supported properties:

| Original evidence | GTNH setting |
|---|---|
| Main display name / case-sensitive FML ID | `modName` / `modId` |
| Main package / archive base | `modGroup` / `customArchiveBaseName` |
| Public API beneath mod group | `apiPackage` |
| Replaced version tokens | `generateGradleTokenClass = <modGroup>.Tags` |
| AT, core plugin, mixins | `accessTransformersFile`, `coreModClass`, `usesMixins`, and related package/plugin properties |

Check relative-package and resource-path semantics in the current template before transferring manifest values. Preserve valid hand-written mixin configuration rather than changing frameworks during a build-only migration.

Select the template-supported Forge and Java settings for the requested environment. Use `enableModernJavaSyntax = jabel` when modern syntax with Java 8-compatible output is intended; obsolete boolean settings are not interchangeable. Preserve the intended publishing identity with `useModGroupForPublishing` instead of copying a historical default.

Without usable project Git tags, set `gtnh.modules.gitVersion = false` and assign `ext.modVersion` during `addon.gradle` evaluation. Use the generated `Tags.VERSION` in the mod version constant. See the [shared version-consistency pattern](../common/examples.md#explicit-version-without-usable-git-tags).

Keep list-style `mcmod.info` with `${modId}`, `${modName}`, `${modVersion}`, and `${minecraftVersion}` in the main entry. Original child plugins retain literal identities/parents and share version tokens. Populate dependency metadata with real FML IDs, consistent with runtime annotations.

Move custom tasks/version logic/extra source roots into `addon.gradle`, dependencies into `dependencies.gradle`, and additional repositories into `repositories.gradle`. Avoid repositories already supplied by `includeWellKnownRepositories`.

**Gate:** built metadata, annotation, and archive agree on identity/version and original hooks remain configured.

## 3. Resolve mapped dependencies

Apply the [shared dependency reference](../common/reference.md) with GTNH-specific configurations:

| Requirement | GTNH configuration shape |
|---|---|
| Publicly exposed runtime API | `api` with a compatible development/API artifact |
| Internal runtime implementation | `implementation`, when appropriate for publication |
| Optional imported types | `compileOnly` |
| Development runtime, not published | `devOnlyNonPublishable` / `runtimeOnlyNonPublishable`, according to mapping needs |

```groovy
// Placeholders: resolve names, versions, and mapping state before use.
compileOnly("com.github.GTNewHorizons:<artifact>:<version>:dev")
devOnlyNonPublishable(rfg.deobf("curse.maven:<slug>-<projectId>:<fileId>"))
```

Do not deobfuscate an already MCP-mapped development jar. Do not publish an obfuscated Curse artifact as an ordinary `api` dependency. Select development rather than API-only artifacts if the original code needs implementation internals.[^porting]

Inspect the current run task's injected compatibility libraries before duplicating them. Development shims do not replace production requirements. CoFHCore/CoFHLib and CodeChicken packaging distinctions are covered in the reference, without fixed historical file pins.

Only when ID-expander replacement is in scope, follow the [conditional migration branch](../common/reference.md#id-expander-migration-branch); EndlessIDs is not mandatory for every legacy mod.

**Gate:** both compile and runtime classpaths preserve the original required integrations.

## 4. Verify the GTNH branch

Fix only demonstrated migration blockers. Typical 1.7.10 cases include obsolete utility imports, `FluidStack` accessor changes, stricter generic inference, and MCP `__OBFID` markers rejected by reobfuscation. Check the actual resolved APIs rather than applying edits blindly.

For templates exposing these tasks:

```bash
./gradlew spotlessApply
./gradlew build
./gradlew runClient25
```

Confirm task/toolchain availability first. Follow configured style rules.[^style] Confirm FML loaded the intended mod and progressed through initialization/resource reload; `Not beginning mod initialization phase` is a failure, not an optional-integration warning. Apply the shared packaged-jar, side-testing, process-stop, cleanup, and final-diff gates.

**Gate:** fresh build and confirmed modern-Java startup, or explicit reporting of any unexecuted check.

## Optional build-script update

Only when requested, run `./gradlew updateBuildScript` after setting the real identity. Re-read wrapper/settings/properties changes, resolve duplicate keys, and repeat verification. This updates build infrastructure; it does not migrate source/dependency behavior by itself.

[^migration]: [Official migration guide](https://github.com/GTNewHorizons/ExampleMod1.7.10/blob/master/docs/migration.md).
[^porting]: [Official porting guide](https://github.com/GTNewHorizons/ExampleMod1.7.10/blob/master/docs/porting.md).
[^style]: [GTNH Code Style](https://wiki.gtnewhorizons.com/wiki/Code_Style).
