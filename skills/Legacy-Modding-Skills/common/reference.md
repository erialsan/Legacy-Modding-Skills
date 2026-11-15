---
type: Reference
title: Shared dependency and compatibility reference
description: Resolve compatible artifacts, mapping states, runtime mod IDs, unpublished jars, and dependency API changes across legacy Minecraft versions.
tags: [minecraft, dependencies, compatibility]
---

# Shared dependency and compatibility reference

Apply this during the [shared migration procedure](./workflow.md). Dependency configuration names and deobfuscation helpers belong to the selected [1.7.10 GTNH](../1.7.10/SKILL.md) or [1.12.2 CleanroomMC](../1.12.2/SKILL.md) playbook; they are not portable between all Gradle plugins.

## Artifact resolution

1. Establish the target Minecraft version, runtime, mappings, and required API before selecting artifacts. A matching display name is not sufficient compatibility evidence.
2. Search maintained artifacts and author repositories. Verify exact coordinates/releases from Maven metadata, POMs, source jars, and distribution metadata; a historical table or truncated directory listing is not authoritative.
3. For CurseMaven, verify both project and file IDs and the selected file's Minecraft version. Distinguish MCP/development jars from obfuscated universal jars.
4. Use the chosen plugin's supported remapping/deobfuscation mechanism for universal jars. Do not remap an already mapped development jar merely because it came from CurseForge.
5. Use local jars only when no compatible reproducible artifact exists. Document source, version, mapping state, license, checksum where available, and a configurable path outside the repository rather than a developer-specific home directory.

Select an API-only artifact only when it contains all imported classes. Genuine use of implementation internals requires the compatible implementation/development artifact or a separately scoped API refactor.

**Check:** resolution succeeds, required classes exist, and compile/runtime mapping states match their use.

## Representative recurring distinctions

Names illustrate recurring legacy patterns, not required dependencies or interchangeable cross-version artifacts.

| Example | General lesson |
|---|---|
| CoFHCore / CoFHLib | A library requirement does not imply the whole companion mod is required; inspect imports, reflection, metadata, and transitives for the chosen version. |
| CodeChickenCore / CodeChickenLib | Forks and Minecraft versions may package library classes differently; inspect the selected artifact instead of assuming a universal layout. |
| Recipe-viewer integrations | Resolve the artifact that owns the imported API; similar names do not establish source or binary compatibility. |
| Multipart integrations | A runtime-scoped POM dependency may not be available to compile-only consumers; inspect the compile classpath. |

## Runtime mod IDs and optional integrations

Read IDs/version constraints from the actual dependency jar's annotation, metadata, or coremod container. Preserve case and punctuation. Artifact coordinates, display names, and the caller's `Loader.isModLoaded` string are not substitutes for dependency metadata.

A mismatch can produce `The mod X (X) requires mods [Y] to be available` followed by `Not beginning mod initialization phase`. Resolve the missing artifact or wrong ID against metadata, then rerun startup. Keep the required behavior rather than deleting the requirement.

Unconditional reflective class loading that throws on absence can establish a runtime dependency even without `required-after`. Guarded reflection in an isolated optional integration does not establish the same requirement. Compile-only configuration alone does not make unconditional class references safe when the target is absent.

**Check:** required mods are present under their real IDs, and optional branches remain safe without their targets.

## Legacy IDE projects and vendored APIs

An empty Gradle dependency block does not establish that the project is dependency-free. Inspect IDE classpaths, bundled jars, imports, manifests, reflective strings, and build tasks.

Avoid adding a second artifact containing classes already vendored in the source tree. Retaining licensed sources versus replacing them with a dependency is a deliberate compatibility/licensing choice, not a mechanical migration step.

## Development support versus production requirements

Run tasks may inject libraries not represented in a published mod's dependency contract. Inspect the actual launcher configuration and resolved classpath instead of assuming a fixed injected set.

- Reuse launcher-provided support needed only by the development environment.
- Declare compile requirements explicitly when the mod imports the library's classes.
- Preserve production runtime requirements and installation guidance even if the development launcher supplies those jars.

**Check:** successful startup is not masking an undeclared production dependency.

## Dependency API drift

Compare old imports/calls with the selected version's source jars. Check moved packages, changed parameters, abstract methods, mapping changes, and sided class loading before editing.

Record old symbol → resolved symbol, dependency version, and the demonstrated failure. Do not apply another mod's historical package rename without confirming the same target API. Keep gameplay redesign separate from blocker fixes.

## ID-expander migration branch

Use only when replacing an existing expansion mechanism is explicitly in scope for the selected Minecraft version/runtime.

1. Identify expanded registries and persisted world/ID-format assumptions.
2. Trace old downloader metadata, setup hooks, coremod checks, and warnings before removing superseded code.
3. Verify the replacement's Minecraft version, runtime IDs, required libraries, and mixin-loader requirements from current distribution metadata.
4. Update annotations/resources consistently and test affected registrations and representative copied worlds with backups.

EndlessIDs is an example from the 1.7.10 ecosystem, not a mandatory dependency or a 1.12.2 recommendation. Never infer that an expansion solution carries across Minecraft versions. Startup alone does not prove save compatibility.
