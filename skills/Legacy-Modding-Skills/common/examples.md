---
type: Examples
title: Shared legacy migration patterns
description: Template-neutral examples for version consistency, source layout, optional integrations, runtime libraries, and vendored APIs.
tags: [minecraft, migration, examples]
sources:
  - resource: ./workflow.md
    title: Shared migration procedure
  - resource: ./reference.md
    title: Shared dependency reference
---

# Shared migration patterns

These are illustrative reasoning patterns, not tested port records. Use the chosen version playbook for actual property names, configuration helpers, and task commands. Apply the [shared verification gates](./workflow.md#5-verify-and-clean-up).

## Explicit version without usable Git tags

**Situation:** release metadata establishes a version, while tags are absent or inherited from a template.

Determine the selected template's supported version source. Preserve the intended release identity by configuring that source, then wire annotations, processed metadata, and archive version to it. Do not copy another template's Git-version disable flag or token syntax. If tag creation is needed, obtain authorization rather than creating release tags implicitly.

**Check:** built jar, main mod annotation, and processed metadata all carry the intended version, without relying on unrelated template tags.

## Main source tree plus an additional API tree

**Situation:** IDE configuration uses root-level `java/` and `resources/`, alongside a separate API tree.

After checking destination collisions, normalize the main tree:

```text
java/       → src/main/java/
resources/  → src/main/resources/
```

Register the additional API tree only through the chosen build's supported source-set mechanism. Restore AT/coremod resource semantics from original evidence, not from the old directory name. Check formatter coverage for every additional root.

**Check:** each source/resource is included once, the additional API compiles, and manifest/resource paths agree in the packaged jar.

## Optional integration without imported types

**Situation:** integration uses only registry names, OreDictionary entries, or IMC messages and safely skips work when its target is absent.

Do not add a compile jar solely for these names. Preserve guards and ordering metadata. Callback-like method names are not proof of external API imports.

**Check:** build and startup work without the optional target. Test with it when integration behavior is in scope; otherwise report that branch as untested.

## Runtime requirement discovered through reflection

**Situation:** an unconditional path loads a library class and throws on absence. CoFHLib is a representative legacy example where applicable to the version.

Resolve the matching library for runtime even if annotations omit `required-after`. Inspect the selected file's mapping state before choosing the template's dependency helper. Do not add companion CoFHCore merely because its namespace is related; verify actual requirements. Document production installation separately from development-only classpath configuration.

**Check:** the original reflective path succeeds during startup and the library's own requirements are met.

## Vendored APIs and circular integrations

**Situation:** API sources are already included, or an optional consumer depends back on the library being migrated.

Resolve duplicate-class/licensing implications before adding a jar for the vendored API. Keep optional consumer types compile-only where that matches the integration boundary, rather than creating an unnecessary runtime dependency cycle.

**Check:** no duplicate classes, no new hard cycle, and optional absence remains safe.

## Changed dependency internals

**Situation:** a maintained fork moved an internal class or changed an abstract method.

Verify the selected artifact contains the required implementation, compare its sources with failing imports/calls, then make the smallest supported adaptation. A successful compile does not by itself establish changed runtime semantics.

**Check:** the demonstrated failure is fixed, relevant tests pass, and runtime-sensitive behavior is exercised or explicitly untested.
