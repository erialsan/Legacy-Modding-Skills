---
type: Playbook
name: Legacy-Modding-Skill-Cleanroom
title: Legacy-Modding-Skill — Minecraft 1.12.2 / CleanroomMC
description: Migrate Minecraft 1.12.2 builds using either CleanroomModTemplate or ForgeDevEnv, keeping their build systems and runtime compatibility targets distinct.
tags: [minecraft, 1.12.2, cleanroom, forge, migration]
sources:
  - id: cleanroom-template
    resource: https://github.com/CleanroomMC/CleanroomModTemplate
    title: CleanroomModTemplate
  - id: cleanroom-properties
    resource: https://github.com/CleanroomMC/CleanroomModTemplate/blob/main/gradle.properties
    title: CleanroomModTemplate properties
  - id: cleanroom-build
    resource: https://github.com/CleanroomMC/CleanroomModTemplate/blob/main/build.gradle
    title: CleanroomModTemplate build
  - id: forge-env
    resource: https://github.com/CleanroomMC/ForgeDevEnv
    title: ForgeDevEnv
  - id: forge-properties
    resource: https://github.com/CleanroomMC/ForgeDevEnv/blob/master/gradle.properties
    title: ForgeDevEnv properties
  - id: forge-build
    resource: https://github.com/CleanroomMC/ForgeDevEnv/blob/master/build.gradle
    title: ForgeDevEnv build
---

# Minecraft 1.12.2 — CleanroomMC templates

Load the [shared procedure](../common/workflow.md) first. This playbook owns the 1.12.2 build choices; [1.7.10 GTNH properties and tasks](../1.7.10/SKILL.md) are not interchangeable with them.

## 1. Choose one template and runtime contract

| Target | Template | Build family |
|---|---|---|
| Cleanroom-specific development | [CleanroomModTemplate](https://github.com/CleanroomMC/CleanroomModTemplate) | CleanroomGradle + CleanroomMC Conventions |
| Forge 1.12.2 development environment | [ForgeDevEnv](https://github.com/CleanroomMC/ForgeDevEnv) | RetroFuturaGradle with the template's Forge/mapping configuration |

CleanroomModTemplate's README currently requires Cleanroom 0.7.0 or above. ForgeDevEnv is a Forge 1.12.2 environment, not another name for CleanroomModTemplate.[^cleanroom-template][^forge-env]

Confirm whether the artifact must support Cleanroom only, Forge only, or both. If unspecified, resolve that choice before changing runtime/build assumptions. Do not infer Cleanroom-only APIs are safe for a Forge-compatible artifact, or that running ForgeDevEnv proves Cleanroom compatibility.

Record the selected upstream commit, wrapper, plugins, mappings, Gradle JVM, compilation toolchain, bytecode target, and launch runtime. These evolve independently: do not copy the README's current Java/Gradle numbers as permanent requirements. In particular, ForgeDevEnv can use a modern Gradle JVM while configuring Java 8-compatible output.[^forge-build]

**Gate:** one template revision and an explicit runtime/bytecode compatibility contract are recorded.

## 2. Transfer originals into the selected template

Use a protected staging copy or reviewed file-by-file transplant after the shared backup gate. There is no assumption of a GTNH-style `migration.zip` for these templates. Preserve original Git identity/history and license; do not copy upstream template tags into the mod's release history.

Normalize source/resources, map original identity and custom tasks, and remove only proven example sources. Both current templates expose `root_package`, `mod_id`, and `mod_name`; other keys and processing mechanisms differ.[^cleanroom-properties][^forge-properties]

Keep the chosen template's extension pattern. Do not apply GTNH `addon.gradle`, `generateGradleTokenClass`, `gtnh.modules.gitVersion`, or `runClient25` by analogy.

**Gate:** source/resource mapping is complete and original build customizations have supported destinations.

## 3A. CleanroomModTemplate branch

Follow its current README and adjacent property documentation.[^cleanroom-template]

- Set identity with `root_package`, `mod_id`, and `mod_name`. Preserve package stability unless a rename is deliberately required; update source locations and references consistently if changing packages.
- Versioning is Git-tag-derived. Use `./gradlew -q printVersion` to inspect the result. If tags are absent or inappropriate, determine the pinned Conventions/Versioning plugin's supported approach; do not assume ForgeDevEnv's `mod_version` overrides it or create release tags without authorization.
- Token Envoy uses `@{NAME}` markers declared through `tokens.properties`; the current build also supplies version/author tokens. Inspect generated sources and processed resources rather than retaining another template's `${...}` or `@VERSION@` assumptions.[^cleanroom-build]
- Preserve the existing license. Review `conventions.license` before using `extractConventions` or `applyLicenseHeader`; those tasks can write license/header content. Do not select a different upstream default license on the user's behalf.
- Transfer AT filenames to `access_transformers` and the current template's `META-INF` resource layout. Transfer the core plugin through `coremod_plugin_class` and `coremod_includes_mod`.
- Where mixins exist, align the actual config with `cg.userdev.refmap`, annotation processing, and packaged refmap. Inspect the pinned Cleanroom mixin-loading mechanism; do not transplant ForgeDevEnv's `use_mixins` toggles or add a second loader without evidence.
- Use the CleanroomGradle dependency/model configuration supported by the selected revision. Its current build uses `cleanroom.userdev(cleanroom_version)`; this is not an RFG deobfuscation API.[^cleanroom-build]

**Gate:** the resolved version, tokens, coremod/AT/mixin resources, license handling, and Cleanroom runtime agree with the selected revision.

## 3B. ForgeDevEnv branch

Use the template's current properties and scripts rather than replacing its RFG build with CleanroomGradle.[^forge-env]

- Set `mod_version`, `root_package`, `mod_id`, and `mod_name` from the original project.
- If generated tags are used, configure `use_tags`, `tag_class_name`, and `tags.properties`. Resource expansion is handled separately by the build's `processResources`; inspect its actual placeholders rather than substituting Cleanroom Token Envoy syntax.[^forge-build]
- Keep `mapping_channel`/`mapping_version` compatible with existing code or explicitly adapt demonstrated mapping failures.
- Add dependencies through `gradle/scripts/dependencies.gradle` and custom build behavior through `gradle/scripts/extra.gradle`, as supported by the selected revision. Inspect the RFG helper/configuration comments before remapping artifacts.
- Transfer ATs through `use_access_transformer` and `access_transformer_locations`; current source placement/packaging differs from CleanroomModTemplate's AT layout.
- Preserve coremod flags through `is_coremod`, `coremod_plugin_class_name`, and `coremod_includes_mod`.
- Preserve mixin behavior through the selected revision's `use_mixins`, configs/refmap, and optional connector settings. Verify whether its configured loader is needed in the intended production runtime rather than stacking loaders by default.[^forge-properties]

**Gate:** identity/version, generated tags, expanded resources, RFG classpaths, and packaged hooks match the intended Forge runtime.

## 4. Resolve dependencies without cross-template assumptions

Apply the [shared dependency reference](../common/reference.md) using only 1.12.2-compatible artifacts. Similar mod names across 1.7.10 and 1.12.2 do not establish matching APIs, IDs, or loaders.

Inspect compile/runtime mappings and the chosen plugin's supported dependency configurations. RFG-specific `rfg.deobf` and GTNH non-published configurations must not be copied into CleanroomGradle unless that pinned build explicitly provides them. Likewise, Cleanroom `userdev` configuration is not a ForgeDevEnv dependency recipe.

Separate runtime APIs from optional integrations, and production requirements from launcher-provided support. Check both guarded optional absence and required reflective paths. The [shared examples](../common/examples.md) illustrate these decisions without imposing fixed mod lists.

**Gate:** resolved artifacts match Minecraft 1.12.2, the selected mappings/runtime, and actual type/runtime use.

## 5. Verify the selected branch

Inspect `./gradlew tasks --all` and toolchain settings first, then run:

```bash
./gradlew build
./gradlew runClient
```

Use a managed process for launch tasks. For CleanroomModTemplate, run configured checks/formatting tasks when available; the current properties document `clearSkiesApply` and `formatJavaApply`. Treat license-writing tasks separately, not as a harmless formatter.[^cleanroom-properties] ForgeDevEnv does not inherit GTNH's Spotless task names merely because both use RFG.

Apply the shared packaged-jar and startup gates. For server-capable mods, use `runServer` where available; do not silently accept server EULA or modify user saves to complete verification. ForgeDevEnv also defines obfuscated run tasks in the inspected build: exercise them when available to detect development-only mapping assumptions.[^forge-build]

If Forge and Cleanroom compatibility are both claimed, validate the built artifact in both requested environments; one development launch proves only the environment exercised. Record missing display/toolchain/artifact access as explicit limitations.

**Gate:** build and relevant runtime/side tests match the promised compatibility, packaged metadata has no unresolved tokens, and the final diff passes the shared cleanup gate.

[^cleanroom-template]: [CleanroomModTemplate README](https://github.com/CleanroomMC/CleanroomModTemplate).
[^cleanroom-properties]: [CleanroomModTemplate properties](https://github.com/CleanroomMC/CleanroomModTemplate/blob/main/gradle.properties).
[^cleanroom-build]: [CleanroomModTemplate build](https://github.com/CleanroomMC/CleanroomModTemplate/blob/main/build.gradle).
[^forge-env]: [ForgeDevEnv README](https://github.com/CleanroomMC/ForgeDevEnv).
[^forge-properties]: [ForgeDevEnv properties](https://github.com/CleanroomMC/ForgeDevEnv/blob/master/gradle.properties).
[^forge-build]: [ForgeDevEnv build](https://github.com/CleanroomMC/ForgeDevEnv/blob/master/build.gradle).
