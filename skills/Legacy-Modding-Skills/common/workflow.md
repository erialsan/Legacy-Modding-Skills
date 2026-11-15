---
type: Playbook
title: Shared legacy-mod migration procedure
description: Preserve project identity and behavior, reconstruct dependencies, and verify migrations independently of Minecraft version or build template.
tags: [minecraft, migration, preservation, verification]
---

# Shared migration procedure

Read this after choosing a version in [Legacy-Modding-Skill](../SKILL.md). Template-specific plugins, configurations, tokens, and task names come from the selected [GTNH](../1.7.10/SKILL.md) or [CleanroomMC](../1.12.2/SKILL.md) playbook, not from this common procedure.

## 1. Inventory and protect originals

Read the original build/settings scripts, properties, wrapper, mod annotations, metadata, manifests, source sets, IDE classpaths, bundled jars, version files, and submodules before applying a template. Record:

- Minecraft/runtime version, mappings, build JVM, compilation toolchain, and output bytecode target separately; a modern Gradle JVM does not imply modern bytecode compatibility.
- Case-sensitive mod IDs, names, package roots, authors, URLs, license, version sources, publishing identity, and additional original mod containers.
- AT/coremod/mixin hooks, custom resource processing/tasks, extra sources, vendored APIs, and reflective runtime checks.
- Conflicts between release metadata, hardcoded versions, and Git tags; resolve them against actual release intent rather than silently selecting a stale value.

Inspect the selected template revision and archive/tree before overlaying. Record the revision and create recoverable copies of every overwritten customization. Use unique backup names or a recoverable worktree/snapshot; never overwrite an earlier backup. Record exact backup paths for later cleanup.

**Gate:** every original customization has a mapped destination or recoverable copy. A missing properties/dependency file is recorded as absent, not invented.

## 2. Transfer without changing scope

Keep the selected template's build entry point and supported extension mechanisms. Move root-level IDE source trees into standard locations only after checking for destination collisions; merge explicitly when both trees exist. Register additional sources exactly once and restore submodules at recorded commits.

Preserve original mod containers, metadata, licenses, and gameplay. Treat full API rewrites, coremod redesign, Minecraft-version upgrades, publishing, license changes, and ID-expander replacement as separate scope unless requested. Remove template example sources only after distinguishing them from original plugins.

**Gate:** original sources/resources and build behavior have explicit destinations, with no accidental overwrites or new gameplay changes.

## 3. Reconstruct dependencies

Classify each dependency by public API exposure, internal implementation, optional type use, development startup, and production runtime needs. Resolve artifacts for the selected Minecraft version, mappings, and runtime using the [dependency reference](./reference.md).

Preserve required integrations instead of dropping them to reach the menu. Guarded registry/IMC/OreDictionary integration without imported types does not by itself need a compile jar. Reflection can establish a runtime requirement, but only after inspecting whether the path is unconditional.

**Gate:** every retained or removed dependency has source/build/runtime evidence, and no duplicate vendored APIs or accidental hard dependency cycle is introduced.

## 4. Fix demonstrated migration blockers

Limit source edits to reproduced compilation, packaging, formatter, or startup failures. Compare changed dependency APIs with the selected source jars before adapting calls. Match filenames to public Java class names on case-sensitive systems. Detect invalid UTF-8 and determine the original encoding before transcoding.

Replace unsupported JDK-internal APIs with supported mechanisms; delete obsolete downloaders only after proving their function is no longer required. Use configured formatters rather than manually restyling the code. Preserve quality checks; fix reported failures or document an agreed exception instead of hiding them with copied disable flags.

**Gate:** each functional source edit addresses demonstrated evidence or separately authorized behavior.

## 5. Verify and clean up

1. Inspect the selected template's available tasks and configured toolchains. Run formatting/checks and a fresh build using its actual task names.
2. Inspect the packaged jar: identity/version metadata, AT/coremod/mixin manifest entries and resources, refmap when applicable, and absence of unresolved tokens or template identities.
3. Launch the appropriate client as a managed long-lived process. Follow logs past mod initialization and resource reload; confirm the intended mod ID/version and completed startup. A generic crash-report heading alone is not a verdict: inspect the exception and subsequent progress.
4. Test dedicated-server startup for server-capable mods when available; report any unexecuted side or runtime as untested. Client startup does not prove gameplay, save compatibility, server safety, or compatibility with a different runtime.
5. Stop managed processes unless the user requests they remain open. Record exact commands, exit results, startup evidence, and remaining warnings. Toolchain/display/artifact blockers are limitations, not successful checks.
6. Remove only the recorded migration-created backups, unused replaced jars/IDE configuration, and confirmed template leftovers after verification. Preserve original licenses/docs, unrelated backups, and user data. Review the final diff for content loss and separate formatting churn from functional edits.

**Gate:** the report's claims match fresh evidence; untested behavior is explicit and no unrelated content is removed.

For recurring implementation shapes, consult the [illustrative patterns](./examples.md); they are not previously verified ports.
