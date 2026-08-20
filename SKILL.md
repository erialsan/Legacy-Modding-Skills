---
name: gtnh-template-port
description: >-
  Ports a Forge 1.7.10 mod onto the GTNH ExampleMod1.7.10 template
  via official migration.zip overlay (backup originals, unzip, rewrite
  gradle.properties / mcmod.info / dependencies.gradle / addon.gradle),
  then spotlessApply, build, and runClient25. Use when the user asks to
  migrate/port a 1.7.10 mod to the GTNH template, overlay migration.zip,
  bump an old GTNH RFG build.gradle to gtnhconvention, adapt
  build.gradle.old / mcmod.info.old, run updateBuildScript, or verify
  modern 1.7.10 via runClient25.
---

# GTNH template port

Official ExampleMod1.7.10 migration ([docs/migration.md](https://github.com/GTNewHorizons/ExampleMod1.7.10/blob/master/docs/migration.md)): backup originals → overlay [migration.zip](https://github.com/GTNewHorizons/ExampleMod1.7.10/releases/download/master-packages/migration.zip) → rewrite to template form. No user pre-work. Then identity, extra source roots, version tokens, dependency rewrite, leftovers, compile-breakers, `updateBuildScript`, and modern-Java verification.

Official (always):

- Procedure: [migration.md](https://github.com/GTNewHorizons/ExampleMod1.7.10/blob/master/docs/migration.md) · [porting.md](https://github.com/GTNewHorizons/ExampleMod1.7.10/blob/master/docs/porting.md) · [ExampleMod1.7.10](https://github.com/GTNewHorizons/ExampleMod1.7.10)
- Modding best practices: [Development](https://wiki.gtnewhorizons.com/wiki/Development) · [Code Style](https://wiki.gtnewhorizons.com/wiki/Code_Style)
- Most advanced 1.7.10 example: [OmoshiroiKamo](https://github.com/Shigure-Ruiseki/OmoshiroiKamo)

Compile-breakers required for `build` / `runClient25` are in scope. Full API/feature ports are not, unless asked.

## Non-goals

- Full gameplay port, mixin/coremod rewrite, or dropping hard deps "so the menu opens"
- `disableSpotless` to hide format debt
- `./gradlew updateBuildScript` unless asked

## Workflow

Copy and track:

```
- [ ] Backup build.gradle, gradle.properties, mcmod.info → .old (or .back)
- [ ] Overlay migration.zip; rewrite to template form
- [ ] Inventory from backups (do not invent)
- [ ] Identity → gradle.properties
- [ ] mcmod.info tokens
- [ ] addon.gradle extras / version
- [ ] dependencies.gradle + repositories.gradle
- [ ] Delete ExampleMod leftovers
- [ ] Compile-breakers
- [ ] spotlessApply → build → runClient25
- [ ] updateBuildScript only if asked
```

Read [reference.md](reference.md) when resolving artifacts, hard-dep modids, FMP/CCC, or curse file IDs. Read [examples.md](examples.md) when the target is Calculator, SonarCore, or a similar CoFH/optional-compat tree.

## 1. Backup + overlay

No pre-work. Do not ask the user to overlay first.

1. Copy (do not delete) these to the same path + `.old`. If that exists, `.back`:
   - `build.gradle` / `build.gradle.kts`
   - `gradle.properties`
   - `src/main/resources/mcmod.info`

2. Overlay the official zip:

```
curl -fsSL -o /tmp/gtnh-migration.zip \
  https://github.com/GTNewHorizons/ExampleMod1.7.10/releases/download/master-packages/migration.zip
unzip -o /tmp/gtnh-migration.zip
rm /tmp/gtnh-migration.zip
```

Replaces wrapper, `settings.gradle.kts`, `build.gradle.kts`, `gradle.properties`, `dependencies.gradle`, `repositories.gradle`, `mcmod.info`, and ExampleMod sources. Backups stay. Official `migration.md` keeps `build.gradle` until deps are copied — backups make a full overlay safe.

3. Rewrite **template** files from the backups (never edit `.old`/`.back` in place):
   - identity → `gradle.properties` (§3)
   - tokens → `mcmod.info` (§4)
   - extra roots / version / custom tasks → `addon.gradle` (§5)
   - old `repositories {}` → `repositories.gradle`, old `dependencies` → `dependencies.gradle` (§6)

Skip the zip only if `build.gradle.kts` is already only `id("com.gtnewhorizons.gtnhconvention")`, `modId` is the real mod (not `mymodid`), and backups already exist. A lagging `settings.gradle.kts` version is then `updateBuildScript` (§10), not a zip redo.

Restore deleted `.gitmodules` if a gitlink remains (`git ls-files -s <path>` mode `160000`). Origin may still be `GTNewHorizons/ExampleMod1.7.10` after overlay — do not trust its git tags as the mod version.

## 2. Inventory

Read, do not invent. Prefer `*.old` / `*.back` over the post-overlay template copies:

- `build.gradle` / `build.gradle.old` — `group`, `archivesBaseName`, `minecraft.version`, `replaceIn` / `@VERSION@`, extra `sourceSets`, repos, `dependencies`
- `gradle.properties` / `.old` — `modName`/`modId`/`modGroup`, mixins, AT, `disableSpotless`
- `src/main/resources/mcmod.info` / `.old` — name, authors, URL, deps
- `version.properties` / changelog — numeric version if no git tags
- `@Mod` class — `modid`, `name`, `version`, `dependencies`. `Loader.isModLoaded("X")` / old `requiredMods` spelling is **not** the FML id
- `.gitmodules` — extra source trees (e.g. BeefCore)
- `src/api/java` **or** already-in-tree `src/main/java/{cofh,ic2,powercrystals}/` — vendored compile-only APIs. Do **not** also add matching maven deps (duplicate classes)
- leftover ExampleMod tree `src/main/java/com/myname/mymodid/` — delete after `modGroup` is set

## 3. Identity → `gradle.properties`

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
- `enableModernJavaSyntax = jabel` unless the user wants `modern` / `jvmDowngrader`. Old boolean `true` → `jabel`
- `includeWellKnownRepositories = true` (CurseMaven)
- `useModGroupForPublishing = false` for GTNH-style `com.github.GTNewHorizons` coordinates (old template default; new defaults `true`)
- carry `accessTransformersFile`, `usesMixins`, `mixinsPackage`, `mixinPlugin` from `.old`. A hand-written `mixins.<modId>.json` is fine if its `package` matches `mixinsPackage`. Do not convert to IMixins unless asked

No usable git tags (or the tags belong to ExampleMod / another overlay) and version comes from `version.properties` / a hardcoded `@Mod` string:

```
gtnh.modules.gitVersion = false
```

Then in `addon.gradle` set `ext.modVersion` **during evaluation** (§5). Env `VERSION` also overrides if git version stays on. Leaving git version on after an ExampleMod overlay stamps tags like `0.2.3` onto the ported jar.

## 4. `mcmod.info`

Must be list-v2. The **main** entry must contain exactly these tokens:

`${modId}` `${modName}` `${modVersion}` `${minecraftVersion}`

Multi-entry files (parent + `ModId|Compat*` children) are valid. Children keep literal `modid`/`name`/`parent`; every entry still needs `${modVersion}` and `${minecraftVersion}`.

Keep description / url / authors / credits from `.old`. Map old hard `dependencies` on the main entry to:

```json
"requiredMods": ["HardDep"],
"dependencies": ["HardDep"],
"dependants": [],
"useDependencyInformation": true
```

`@Mod(dependencies = …)` remains the runtime source of truth.

Old `dependencies` that are API package names (`CoFHAPI`, `WailaAPI`) are not FML ids. If `@Mod` has no `required-after`, leave `requiredMods` empty; keep `dependants` from `.old`.

## 5. `addon.gradle`

Old `minecraft { replaceIn … }`, extra `srcDirs`, and `ant.loadproperties` do **not** belong in `build.gradle.kts`.

```groovy
def versionProps = new Properties()
file("version.properties").withInputStream { versionProps.load(it) }
ext.modVersion = "${versionProps.vMajor}.${versionProps.vMinor}.${versionProps.vPatch}${versionProps.vSuffix}"
version = ext.modVersion

sourceSets.main.java.srcDir("BeefCore/src/main/java")
sourceSets.main.java.srcDir("src/api/java")
```

- Extra Java roots from old `sourceSets.main.java.srcDirs += …` go here
- Init missing git submodules to the recorded commit (`git submodule status` line starting with `-`)
- Spotless java target is `src/*/java/**/*.java` — submodule trees outside `src/` are not formatted
- Custom Gradle tasks from the old script also go here

## 6. Dependencies

Copy old `repositories {}` into `repositories.gradle` only if they are **not** already provided by `includeWellKnownRepositories` (GTNH Maven, CurseMaven, Modrinth). That flag does **not** add `https://maven.ic2.player.to/` — IC2 experimental is `net.industrial-craft:industrialcraft-2:2.2.828-experimental:dev` (`:api` exists too):

```groovy
maven {
    name = "ic2"
    url = uri("https://maven.ic2.player.to/")
    metadataSources { mavenPom(); artifact() }
}
```

Rewrite `compile` in `dependencies.gradle`:

| Old use | Config |
|---|---|
| types appear in this mod's public API; needed at runtime | `api` (GTNH/`-dev` jars). Obfuscated Curse universal → `devOnlyNonPublishable(rfg.deobf(…))` — do not `api("curse.maven:…")` |
| internal only, needed at runtime / `runClient*` | `implementation` or `devOnlyNonPublishable` |
| `@Optional` / `Loader.isModLoaded` / compile-only API | `compileOnly` (`transitive = false` if only the API jar) |
| runtime in run tasks, do not publish | `runtimeOnlyNonPublishable` / `devOnlyNonPublishable` |

Resolution: GTNH Maven first (`com.github.GTNewHorizons:<Name>:<latest non-pre>:dev` or `:api`; latest = `maven-metadata.xml` `<release>`, not directory HTML). **Not** on that Maven: CoFHCore / CoFHLib, SonarCore, Calculator, AgriCraft, IC2. Then CurseMaven `curse.maven:<slug>-<projectId>:<fileId>` — prefer author `-dev` (MCP; no `rfg.deobf`); obfuscated universal needs `devOnlyNonPublishable(rfg.deobf(…))`. Never pin 2014 ivy/cursecdn URLs.

Hard `@Mod` deps (`required-after:…`) must be on the **runtime** classpath of `runClient25`. The string is the dependency's `@Mod` / `mcmod.info` `modid` from the deobf jar — not `Loader.isModLoaded` and not a hyphenated display name. Optional integrations (`after:`, `@Optional`, `ModId|Compat*` `@Mod`s) stay `compileOnly`. A method named `getWailaInfo` is not a Waila import; add Waila only if `mcp.mobius` / `IWaila*` types appear.

`runClient25` already pulls Hodgepodge / GTNHLib / lwjgl3ify / UniMixins as run-task shims. Add them to `dependencies.gradle` only if the port needs them published or at compile time. If the port itself needs Hodgepodge DSU mixins / GTNHLib in-game and must not publish them → `runtimeOnlyNonPublishable`.

No standalone `CodeChickenLib` on GTNH Maven — `codechicken.lib.*` is in `CodeChickenCore`. `compileOnly` ForgeMultipart does not pull CCC (runtime-scoped in the POM); add `compileOnly("com.github.GTNewHorizons:CodeChickenCore:<latest non-pre>:dev")` when compiling against `codechicken.lib` / `IChunkLoadTile`.

File IDs, mismatch signatures, and FMP/CCC detail: [reference.md](reference.md).

## 7. Leftovers

- Delete `src/main/java/com/myname/mymodid/` (and any `MyMod` `@Mod`) once `modGroup` is the real package. Two `@Mod` classes will both load
- Do **not** copy `disableSpotless = true` / `disableCheckstyle = true` from `.old`. First `spotlessApply` on a previously-unformatted tree rewrites the whole `src/*/java` set. Expected
- `checkstyleMain` (star-import ban) often still passes after spotless on CoFH-style code. Disable checkstyle only if it actually fails
- Hardcoded `public static final String version = "1.7.10R…"` with no `replaceIn` still becomes `Tags.VERSION` so the jar, `mcmod.info`, and `@Mod` agree after git version is turned off

## 8. Compile breakers

Fix only what blocks `compileJava` / `reobfJar` / first `spotlessApply`. Typical FG 1.2 leftovers:

- `scala.actors.threadpool.Arrays` → `java.util.Arrays`
- `FluidStack.fluidID` → `getFluidID()`
- leftover `@VERSION@` → `Tags.VERSION`
- Linux case-sensitive: `AgriCraftAPIwrapper.java` vs `public class AgriCraftAPIWrapper` — `lsp` `rename_file` to the public class
- `enableGenericInjection = true` (template default): `List<Entity> x = world.selectEntitiesWithinAABB(EntityItem.class, …)` → `List<EntityItem>`
- `reobfJar`: `DO NOT USE __OBFID` — delete MCP `private static final String __OBFID = "CL_…"`
- Non-UTF-8 sources (CP1252 `÷` = `0xF7` in MineTweaker handlers): `:spotlessJava` fails with only `Issue processing file: <path>`. Scan `src/**/*.java` for invalid UTF-8 **before** the first `spotlessApply`; recode CP1252 → UTF-8

Do not reformat by hand; `spotlessApply` owns style. After a manual edit that fails `spotlessJavaCheck` (including leftover blank lines from a delete), re-run `spotlessApply` and `build`.

## 9. Verify

```bash
./gradlew spotlessApply
./gradlew build
./gradlew runClient25
```

`runClient25` is a long-lived process (hub `start`, not blocking bash). Ready-line **must not** be `---- Minecraft Crash Report ----` — FML prints that as splash-screen machine-spec dump (`THIS IS NOT A ERROR`).

Wait for:

- `Forge Mod Loader has successfully loaded N mods`
- `LoadedMods […, <modId>, …]` or join line `<modId>@<version>`
- `<modName>` / `<modId>: Load Complete.` if the mod logs that

Then follow logs past resource reload / sound engine. Texture-missing and an integrated server starting (focus/click) are not boot failures.

`The mod X (X) requires mods [Y] to be available` / `Not beginning mod initialization phase` **is** a boot failure (wrong hard-dep modid or missing runtime jar).

Stop the client after confirmation unless the user wants it left open.

## 10. `updateBuildScript` (only if asked)

```bash
./gradlew updateBuildScript
```

- Bumps `settings.gradle.kts` `gtnhsettingsconvention` (latest: https://github.com/GTNewHorizons/GTNHGradle/releases) and the Gradle wrapper
- Configure may print `Build script update from X to Y available!` and apply it in the same run
- Rewrites / reorders known keys in `gradle.properties` (e.g. moves `gtnh.modules.gitVersion` to the official slot). Re-read after. Do not leave a duplicate key
- Wrapper + configuration-cache may warn that `:wrapper` cannot serialize `ResolvedArtifact`. Benign
- Set identity **before** this task so a rewrite cannot resurrect `MyMod` / `mymodid`. Already-set identity survives
- Does **not** replace `dependencies.gradle`, `addon.gradle`, or source. Still run §9 after it

## Additional resources

- Official procedure: [migration.md](https://github.com/GTNewHorizons/ExampleMod1.7.10/blob/master/docs/migration.md) · [porting.md](https://github.com/GTNewHorizons/ExampleMod1.7.10/blob/master/docs/porting.md) · [ExampleMod1.7.10](https://github.com/GTNewHorizons/ExampleMod1.7.10)
- Official modding best practices: [Development](https://wiki.gtnewhorizons.com/wiki/Development) · [Code Style](https://wiki.gtnewhorizons.com/wiki/Code_Style)
- Most advanced 1.7.10 example: [OmoshiroiKamo](https://github.com/Shigure-Ruiseki/OmoshiroiKamo)
- Artifact IDs, hard-dep mismatch, FMP/CCC: [reference.md](reference.md)
- Worked ports (Calculator, SonarCore): [examples.md](examples.md)
