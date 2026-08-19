---
name: gtnh-template-port
description: >-
  Ports a Forge 1.7.10 mod onto the GTNH ExampleMod1.7.10 template
  (gradle.properties, mcmod.info tokens, dependencies.gradle, addon.gradle),
  then spotlessApply, build, and runClient25. Use when the user asks to
  migrate/port a 1.7.10 mod to the GTNH template, bump an old GTNH RFG
  build.gradle to gtnhconvention, adapt build.gradle.old / mcmod.info.old,
  run updateBuildScript, or verify modern 1.7.10 via runClient25.
---

# GTNH template port

End-to-end: old ForgeGradle 1.2 **or** old GTNH RFG template → latest ExampleMod1.7.10 convention (`gtnhconvention`) → `spotlessApply` → `build` → `runClient25`.

Official generic steps live in the template `docs/migration.md`. This skill is the rest: identity mapping, extra source roots, version tokens, dependency rewrite, `updateBuildScript`, and modern-Java verification.

Do not expand into full code porting (API rewrites, feature stubs) unless asked. Compile-breakers required for `build` / `runClient25` are in scope.

## 0. Inventory

Read, do not invent:

- `build.gradle` / `build.gradle.old` — `group`, `archivesBaseName`, `minecraft.version`, `replaceIn` / `@VERSION@`, extra `sourceSets`, repos, `dependencies`
- `gradle.properties` / `.old` — old GTNH identity (`modName`/`modId`/`modGroup`, mixins, AT, `disableSpotless`)
- `src/main/resources/mcmod.info` / `.old` — name, authors, URL, deps
- `version.properties` / changelog — numeric version if no git tags
- `@Mod` class — `modid`, `name`, `version`, `dependencies`. `Loader.isModLoaded("X")` / old `requiredMods` spelling is **not** the FML id — confirm hard-dep `modid` from the resolved jar's `mcmod.info` / `@Mod` (see §4).
- `.gitmodules` — extra source trees (e.g. BeefCore)
- `src/api/java` — vendored compile-only APIs (CC, MFR, …)
- leftover ExampleMod tree `src/main/java/com/myname/mymodid/` — delete after `modGroup` is set

Three starting states:

1. **Latest convention already applied** if `build.gradle.kts` is only `id("com.gtnewhorizons.gtnhconvention")` and `settings.gradle.kts` uses `gtnhsettingsconvention`. Skip zip overlay; only adapt identity + deps. `settings.gradle.kts` version may still lag — that is `updateBuildScript` (§7), not a zip redo.
2. **Old GTNH template** if `build.gradle(.old)` is a huge RFG script (`com.gtnewhorizons.retrofuturagradle`, header `//version: <epoch>`) and there is no `gtnhconvention` plugin. Overlay [migration.zip](https://github.com/GTNewHorizons/ExampleMod1.7.10/releases/download/master-packages/migration.zip) **except** keep the old `build.gradle` / `gradle.properties` as `*.old` until extras are copied. Then treat as (1).
3. **Raw FG 1.2**: same zip overlay as (2).

Keep the old files as `*.old`. Restore deleted `.gitmodules` if a gitlink remains (`git ls-files -s <path>` mode `160000`). Origin may still be `GTNewHorizons/ExampleMod1.7.10` after a zip/clone overlay — do not trust its git tags as the mod version.

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
- `enableModernJavaSyntax = jabel` unless the user wants `modern` / `jvmDowngrader`. Old GTNH `gradle.properties` used a boolean (`true`/`false`); map `true` → `jabel`.
- `includeWellKnownRepositories = true` (CurseMaven)
- `useModGroupForPublishing = false` for GTNH-style `com.github.GTNewHorizons` coordinates (old template default; new template defaults `true`)
- carry `accessTransformersFile`, `usesMixins`, `mixinsPackage`, `mixinPlugin` from `.old`. A hand-written `mixins.<modId>.json` is fine if its `package` matches `mixinsPackage`. Do not convert to IMixins unless asked.

No usable git tags (or the tags belong to ExampleMod / another overlay) and version comes from `version.properties` / a hardcoded `@Mod` string:

```
gtnh.modules.gitVersion = false
```

Then in `addon.gradle` set `ext.modVersion` **during evaluation** (see §3). Env `VERSION` also overrides if git version stays on. Leaving git version on after an ExampleMod overlay stamps tags like `0.2.3` onto the ported jar.

## 2. `mcmod.info`

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

Copy old `repositories {}` into `repositories.gradle` only if they are **not** already provided by `includeWellKnownRepositories` (GTNH Maven, CurseMaven, Modrinth). That flag does **not** add `https://maven.ic2.player.to/` — IC2 experimental lives there as `net.industrial-craft:industrialcraft-2:2.2.828-experimental:dev` (`:api` exists too). Add:

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

Resolution order:

1. GTNH Maven (`com.github.GTNewHorizons:<Name>:<latest non-pre>:dev` or `:api`). Authoritative latest is `maven-metadata.xml` `<release>` (directory HTML listings truncate). Browse `https://nexus.gtnewhorizons.com/service/rest/repository/browse/releases/com/github/GTNewHorizons/`. **Not** on this Maven: CoFHCore / CoFHLib, SonarCore, Calculator, AgriCraft, IC2 — use CurseMaven or the IC2 player.to repo above.
2. CurseMaven already on the classpath: `curse.maven:<slug>-<projectId>:<fileId>`. Prefer the `-dev` file when the author published one (MCP names; **do not** `rfg.deobf`). Obfuscated universal jars need `devOnlyNonPublishable(rfg.deobf("curse.maven:…"))`. Known 1.7.10 files:
   - CoFHCore 1.7.10-3.1.4-329 = `cofh-core-69162:2388751`
   - CoFHLib 1.7.10-1.2.1-185 = `cofh-lib-220333:2388747`
   - SonarCore 1.7.10-1.1.3 = `sonar-core-239418:2306173` (obfuscated; latest 1.7.10, not the old `1.0.9-dev` local jar)
   - AgriCraft 1.7.10-1.5.0-dev = `agricraft-225635:2284131` (already MCP — no deobf)
   `api("curse.maven:…")` without deobf will not compile against MCP names and must not be published.
3. Do not pin 2014 ivy/cursecdn URLs; they are dead.

Hard `@Mod` deps (`required-after:CoFHCore`, or `dependencies = CoFHCore.version_group`) must be on the **runtime** classpath of `runClient25` or FML will refuse the mod. The string must be the **dependency's** `@Mod modid` / `mcmod.info` `modid`, read from the deobf jar (`~/.gradle/caches/**/*-deobf.jar:mcmod.info`). Do not copy `Loader.isModLoaded("…")` or a hyphenated display name. Mismatch looks like: `FMLFileResourcePack:SonarCore` on the classpath **and** `The mod Calculator (Calculator) requires mods [Sonar-Core] to be available` — then `Not beginning mod initialization phase`. That is a boot failure. Optional integrations (`after:`, `@Optional`, separate `ModId|Compat*` `@Mod`s) stay `compileOnly` (`transitive = false` if only the API jar). Missing optional targets log `Mod X has been disabled through configuration` — not a boot failure. A method named `getWailaInfo` on this mod's tiles is not a Waila API import; do not add Waila unless `mcp.mobius` / `IWaila*` types appear.

GTNH runtime shims that the port needs in-game but must not publish (Hodgepodge DSU mixins, GTNHLib) → `runtimeOnlyNonPublishable`. Hodgepodge late-mixin phase will then see `MINEFACTORY_RELOADED` / the real modid.

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
- `<modName>` / `<modId>: Load Complete.` if the mod logs that

Then follow logs past resource reload / sound engine. Texture-missing lines are not a boot failure. An integrated server starting (focus/click on the window) is not a failure either.
- `The mod X (X) requires mods [Y] to be available` / `Not beginning mod initialization phase` **is** a boot failure (wrong hard-dep modid or missing runtime jar). Splash `---- Minecraft Crash Report ----` is not.
- RFG `runClient25` already pulls Hodgepodge / GTNHLib / lwjgl3ify / UniMixins as run-task shims. Do not add them to `dependencies.gradle` unless the port itself needs them published or at compile time.

Stop the client after confirmation unless the user wants it left open.

## 6. Compile breakers on 10.13.4.1614

Fix only what blocks `compileJava` / `reobfJar` / first `spotlessApply`. Typical FG 1.2 leftovers:

- `scala.actors.threadpool.Arrays` → `java.util.Arrays`
- `FluidStack.fluidID` (old public field) → `getFluidID()`
- `@VERSION@` left in source after dropping `replaceIn` → `Tags.VERSION`
- Linux is case-sensitive: `AgriCraftAPIwrapper.java` vs `public class AgriCraftAPIWrapper` compiles on Windows FG 1.2 and fails here. Rename the file to the public class (`lsp` `rename_file`).
- `enableGenericInjection = true` (template default) makes `List<Entity> x = world.selectEntitiesWithinAABB(EntityItem.class, …)` a compile error. Use `List<EntityItem>`.
- `reobfJar`: `DO NOT USE __OBFID, it's for internal reobfuscator use`. Delete MCP `private static final String __OBFID = "CL_…"`.
- Non-UTF-8 sources (CP1252 `÷` = `0xF7` in MineTweaker handlers) make `:spotlessJava` fail with only `Issue processing file: <path>` and no encoding hint. Scan `src/**/*.java` for invalid UTF-8 **before** the first `spotlessApply`; recode CP1252 → UTF-8.

Do not reformat by hand; `spotlessApply` owns style. After a manual edit that fails `spotlessJavaCheck` (including leftover blank lines from a delete), re-run `spotlessApply` and `build`.

## Non-goals

- Full gameplay port, mixin/coremod rewrite, or dropping hard deps "so the menu opens"
- Disabling Spotless (`disableSpotless`) to hide format debt
- Updating the buildscript (`./gradlew updateBuildScript`) unless asked

## 7. `updateBuildScript` (only if asked)

```bash
./gradlew updateBuildScript
```

- Bumps `settings.gradle.kts` `gtnhsettingsconvention` (check latest at https://github.com/GTNewHorizons/GTNHGradle/releases) and the Gradle wrapper (`gradle-wrapper.properties` + jar).
- Configure may print `Build script update from X to Y available!` and then apply it in the same run.
- Rewrites / reorders known keys in `gradle.properties` (e.g. moves `gtnh.modules.gitVersion` to the official slot at the top). Re-read the file after the task. Do not leave a duplicate key.
- Wrapper + configuration-cache may warn that `:wrapper` cannot serialize `ResolvedArtifact`. Benign; the update still applies.
- Identity values already set in `gradle.properties` survive. Set identity **before** this task so a rewrite cannot resurrect `MyMod` / `mymodid`.
- Does **not** replace `dependencies.gradle`, `addon.gradle`, or source. Still run §5 after it.

## 8. Old-GTNH-template leftovers

- Delete `src/main/java/com/myname/mymodid/` (and any `MyMod` `@Mod`) once `modGroup` is the real package. Two `@Mod` classes will both load.
- Do **not** copy `disableSpotless = true` / `disableCheckstyle = true` from `.old`. First `spotlessApply` on a previously-unformatted tree rewrites the whole `src/*/java` set (hundreds of files). Expected; do not disable to hide it.
- `checkstyleMain` (star-import ban) often still passes after spotless on CoFH-style code. Disable checkstyle only if it actually fails.
- Hardcoded `public static final String version = "1.7.10R…"` with no `replaceIn` still becomes `Tags.VERSION` so the jar, `mcmod.info`, and `@Mod` agree after git version is turned off.

## 9. Field notes — Calculator 1.9.11 (Ollie Lansdell)

State (1): `build.gradle.kts` already `gtnhconvention`. Identity: `modName`/`modId` = `Calculator` (case-sensitive), `modGroup` = `sonar.calculator.mod`, `customArchiveBaseName` = `Calculator`, `useModGroupForPublishing` = false, `generateGradleTokenClass` = `sonar.calculator.mod.Tags`. No `version.properties` / usable tags → `gtnh.modules.gitVersion = false` and `addon.gradle` `ext.modVersion = "1.9.11"`; `Calculator.version = Tags.VERSION`.

Hard runtime: SonarCore. Jar `mcmod.info` `modid` is `SonarCore` (not `Sonar-Core`). `devOnlyNonPublishable(rfg.deobf("curse.maven:sonar-core-239418:2306173"))`. `@Mod(dependencies = "required-after:SonarCore")`.

Optional compileOnly (`transitive = false`): `NotEnoughItems:2.8.125-GTNH:dev`, `CraftTweaker:3.4.8:dev`, `Applied-Energistics-2-Unofficial:rv3-beta-1035-GTNH:dev`, AgriCraft `-dev` curse file above, `net.industrial-craft:industrialcraft-2:2.2.828-experimental:dev`, CoFHLib deobf curse file above. Waila / Natura were name-only — no API imports.

`runClient25` success: `LoadedMods […, Calculator, SonarCore, …]`, join `Calculator@1.9.11`, `Calculator: Successfully loaded with Sonar Core`, `Forge Mod Loader has successfully loaded 17 mods`. Texture-missing is not a failure.
