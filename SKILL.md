---
name: gtnh-template-port
description: >-
  Ports a Forge 1.7.10 mod onto the GTNH ExampleMod1.7.10 template
  (gradle.properties, mcmod.info tokens, dependencies.gradle, addon.gradle),
  then spotlessApply, build, and runClient25. Use when the user asks to
  migrate/port a 1.7.10 mod to the GTNH template, adapt build.gradle.old /
  mcmod.info.old, or verify modern 1.7.10 via runClient25.
---

# GTNH template port

End-to-end: old ForgeGradle 1.2 project → GTNH ExampleMod1.7.10 layout → `spotlessApply` → `build` → `runClient25`.

Official generic steps live in the template `docs/migration.md`. This skill is the rest: identity mapping, extra source roots, version tokens, dependency rewrite, and modern-Java verification.

Do not expand into full code porting (API rewrites, feature stubs) unless asked. Compile-breakers required for `build` / `runClient25` are in scope.

## 0. Inventory

Read, do not invent:

- `build.gradle` / `build.gradle.old` — `group`, `archivesBaseName`, `minecraft.version`, `replaceIn` / `@VERSION@`, extra `sourceSets`, repos, `dependencies`
- `src/main/resources/mcmod.info` / `.old` — name, authors, URL, deps
- `version.properties` / changelog — numeric version if no git tags
- `@Mod` class — `modid`, `name`, `version`, `dependencies`
- `.gitmodules` — extra source trees (e.g. BeefCore)
- `src/api/java` — vendored compile-only APIs (CC, MFR, …)

Template already applied if `build.gradle.kts` is only `id("com.gtnewhorizons.gtnhconvention")` and `settings.gradle.kts` uses `gtnhsettingsconvention`. Then skip zip overlay; only adapt identity + deps.

If the repo is still raw FG 1.2: overlay [migration.zip](https://github.com/GTNewHorizons/ExampleMod1.7.10/releases/download/master-packages/migration.zip) **except** keep the old `build.gradle` as `.old` until extras are copied.

Keep the old files as `*.old`. Restore deleted `.gitmodules` if a gitlink remains (`git ls-files -s <path>` mode `160000`).

## 1. Identity → `gradle.properties`

| Old | Template |
|---|---|
| `@Mod name` / mcmod `name` | `modName` |
| `@Mod modid` / mcmod `modid` (case-sensitive) | `modId` |
| root Java package of the `@Mod` class | `modGroup` |
| `archivesBaseName` | `customArchiveBaseName` |
| public API package under `modGroup` | `apiPackage` (e.g. `api` → `modGroup.api`) |
| `replaceIn Foo.java` + `@VERSION@` | `generateGradleTokenClass = <modGroup>.Tags` and `Foo.VERSION = Tags.VERSION` |

Also set:

- `forgeVersion = 10.13.4.1614` (template; old often `10.13.2.xxxx`)
- `enableModernJavaSyntax = jabel` unless the user wants `modern` / `jvmDowngrader`
- `includeWellKnownRepositories = true` (CurseMaven)

No git tags and version comes from `version.properties` / a hardcoded string:

```
gtnh.modules.gitVersion = false
```

Then in `addon.gradle` set `ext.modVersion` **during evaluation** (see §3). Env `VERSION` also overrides if git version stays on.

## 2. `mcmod.info`

Must be list-v2 and contain exactly these tokens:

`${modId}` `${modName}` `${modVersion}` `${minecraftVersion}`

Keep description / url / authors / credits from `.old`. Map old `dependencies` to:

```json
"requiredMods": ["HardDep"],
"dependencies": ["HardDep"],
"dependants": [],
"useDependencyInformation": true
```

`@Mod(dependencies = …)` remains the runtime source of truth.

## 3. `addon.gradle` extras

Old `minecraft { replaceIn … }`, extra `srcDirs`, and `ant.loadproperties` do **not** belong in `build.gradle.kts`. Put them here.

```groovy
def versionProps = new Properties()
file("version.properties").withInputStream { versionProps.load(it) }
ext.modVersion = "${versionProps.vMajor}.${versionProps.vMinor}.${versionProps.vPatch}${versionProps.vSuffix}"
version = ext.modVersion

sourceSets.main.java.srcDir("BeefCore/src/main/java")
sourceSets.main.java.srcDir("src/api/java")
```

- Extra Java roots from the old `sourceSets.main.java.srcDirs += …` go here.
- Init missing git submodules to the recorded commit (`git submodule status` line starting with `-`).
- Spotless java target is `src/*/java/**/*.java` — submodule trees outside `src/` are not formatted.
- Custom Gradle tasks from the old script also go in `addon.gradle`.

## 4. Dependencies

Copy old `repositories {}` into `repositories.gradle` only if they are **not** already provided by `includeWellKnownRepositories` (GTNH Maven, CurseMaven, Modrinth).

Rewrite `compile` in `dependencies.gradle`:

| Old use | Config |
|---|---|
| types appear in this mod's public API; needed at runtime | `api` |
| internal only, needed at runtime / `runClient*` | `implementation` or `devOnlyNonPublishable` |
| `@Optional` / `Loader.isModLoaded` / compile-only API | `compileOnly` (`transitive = false` if only the API jar) |
| runtime in run tasks, do not publish | `runtimeOnlyNonPublishable` / `devOnlyNonPublishable` |

Resolution order:

1. GTNH Maven (`com.github.GTNewHorizons:<Name>:<latest non-pre>:dev` or `:api`). Browse `https://nexus.gtnewhorizons.com/service/rest/repository/browse/releases/com/github/GTNewHorizons/`.
2. CurseMaven already on the classpath: `curse.maven:<slug>-<projectId>:<fileId>`. Prefer the `-dev` file when the author published one (MCP names; **do not** `rfg.deobf`). Obfuscated universal jars need `devOnlyNonPublishable(rfg.deobf("curse.maven:…"))`.
3. Do not pin 2014 ivy/cursecdn URLs; they are dead.

Hard `@Mod` deps (`required-after:CoFHCore`) must be on the **runtime** classpath of `runClient25` or FML will refuse the mod. Optional integrations stay `compileOnly`.

## 5. Verify (required, in order)

```bash
./gradlew spotlessApply
./gradlew build
```

Then launch modern 1.7.10:

```bash
./gradlew runClient25
```

Treat as a long-lived process (hub `start`, not a blocking bash). Ready-line **must not** be `---- Minecraft Crash Report ----` — FML prints that as splash-screen machine-spec dump (`THIS IS NOT A ERROR`).

Wait for:

- `Forge Mod Loader has successfully loaded N mods`
- `LoadedMods […, <modId>, …]` or join line `<modId>@<version>`

Then follow logs past resource reload / sound engine. Texture-missing lines are not a boot failure.

Stop the client after confirmation unless the user wants it left open.

## 6. Compile breakers on 10.13.4.1614

Fix only what blocks `compileJava`. Typical FG 1.2 leftovers:

- `scala.actors.threadpool.Arrays` → `java.util.Arrays`
- `FluidStack.fluidID` (old public field) → `getFluidID()`
- `@VERSION@` left in source after dropping `replaceIn` → `Tags.VERSION`

Do not reformat by hand; `spotlessApply` owns style. After a manual edit that fails `spotlessJavaCheck`, re-run `spotlessApply` and `build`.

## Non-goals

- Full gameplay port, mixin/coremod rewrite, or dropping hard deps "so the menu opens"
- Disabling Spotless (`disableSpotless`) to hide format debt
- Updating the buildscript (`./gradlew updateBuildScript`) unless asked
