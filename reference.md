# GTNH template port — reference

Lookup tables. Workflow stays in [SKILL.md](SKILL.md).

## Artifact resolution

1. GTNH Maven: `com.github.GTNewHorizons:<Name>:<latest non-pre>:dev` or `:api`.
   - Authoritative latest is `maven-metadata.xml` `<release>`. Directory HTML listings truncate.
   - Browse `https://nexus.gtnewhorizons.com/service/rest/repository/browse/releases/com/github/GTNewHorizons/`.
   - **Not** on this Maven: CoFHCore / CoFHLib, SonarCore, Calculator, AgriCraft, IC2 — CurseMaven or the IC2 `player.to` repo in SKILL.md §6.
2. CurseMaven (already on the classpath when `includeWellKnownRepositories = true`): `curse.maven:<slug>-<projectId>:<fileId>`.
   - Prefer the author `-dev` file when one exists (MCP names; **do not** `rfg.deobf`).
   - Obfuscated universal jars need `devOnlyNonPublishable(rfg.deobf("curse.maven:…"))`.
   - `api("curse.maven:…")` without deobf will not compile against MCP names and must not be published.
3. Do not pin 2014 ivy/cursecdn URLs; they are dead.

### Known 1.7.10 CurseMaven files

| Mod | Version | Coordinate | Notes |
|---|---|---|---|
| CoFHCore | 1.7.10-3.1.4-329 | `cofh-core-69162:2388751` | |
| CoFHLib | 1.7.10-1.2.1-185 | `cofh-lib-220333:2388747` | obfuscated; `rfg.deobf`. Standalone — CoFHCore not required. Replaces local `libs/CoFHLib-…-dev.jar` |
| SonarCore | 1.7.10-1.1.3 | `sonar-core-239418:2306173` | obfuscated; latest 1.7.10, not the old `1.0.9-dev` local jar |
| Calculator | 1.7.10-1.9.11 | `calculator-225104:2306256` | obfuscated; `rfg.deobf`. A `1.9.4-dev` file exists at `2284067` — older API |
| Mekanism | 1.7.10-9.1.1.1031 | `mekanism-268560:2475797` | obfuscated; `rfg.deobf` |
| AgriCraft | 1.7.10-1.5.0-dev | `agricraft-225635:2284131` | already MCP — no deobf |

## Overlay zip (master-packages)

Does replace: wrapper, `settings.gradle.kts`, `build.gradle.kts`, `gradle.properties`, `dependencies.gradle`, `repositories.gradle`, `.editorconfig`, `.github/`, `CODEOWNERS`, `jitpack.yml`, `.java-version`, `gradle-daemon-jvm.properties`, `LICENSE-template`.

Does **not**: replace `mcmod.info`, add `com/myname/mymodid` sources, delete Groovy `build.gradle`.

After unzip: `chmod +x gradlew`; remove leftover Groovy `build.gradle` (keep `.old`) so Gradle uses the kts convention script.

## FG 1.2 empty `dependencies {}`

Old `build.gradle` often has no `dependencies` block. Compile path was Eclipse + `libs/*-dev.jar`. Inventory `libs/` and `.classpath` / `eclipse-*-settings.xml`. Rewrite jars to GTNH Maven / CurseMaven — do not keep `files("libs/…")` when a coordinate exists. After a green verify, delete the rewritten local jars.

Version may live only as `version = "1.5.3"` in that Groovy script. AT may live only as `jar { manifest { attributes 'FMLAT': 'modid_at.cfg' } }`.

## Hard-dep FML id

Hard `@Mod` deps (`required-after:CoFHCore`, or `dependencies = CoFHCore.version_group`) must be on the **runtime** classpath of `runClient25` or FML will refuse the mod.

`Class.forName("cofh.lib…")` / a thrown "X is not installed" exception is also runtime-required even with no `required-after`. Put that jar on `runClient25` (`devOnlyNonPublishable`). CoFHLib standalone is enough; do not also add CoFHCore unless `cofh.core` types appear.

The string must be the **dependency's** `@Mod modid` / `mcmod.info` `modid`, read from the deobf jar (`~/.gradle/caches/**/*-deobf.jar:mcmod.info`). Do not copy `Loader.isModLoaded("…")` or a hyphenated display name.

Mismatch signature (boot failure):

- classpath has `FMLFileResourcePack:SonarCore`
- **and** `The mod Calculator (Calculator) requires mods [Sonar-Core] to be available`
- then `Not beginning mod initialization phase`

Splash `---- Minecraft Crash Report ----` is **not** this failure.

Optional integrations (`after:`, `@Optional`, separate `ModId|Compat*` `@Mod`s) stay `compileOnly` **only if this mod imports that mod's types** (`transitive = false` if only the API jar). `GameRegistry.findItem` / `findBlock` / `FMLInterModComms` with no type imports → no compileOnly jar. Missing optional targets log `Mod X has been disabled through configuration` — not a boot failure.

A method named `getWailaInfo` on this mod's tiles is not a Waila API import; do not add Waila unless `mcp.mobius` / `IWaila*` types appear.

## Runtime shims vs run-task shims

RFG `runClient25` already pulls Hodgepodge / GTNHLib / lwjgl3ify / UniMixins as run-task shims. Do not add them to `dependencies.gradle` unless the port itself needs them published or at compile time.

GTNH runtime shims that the port needs in-game but must not publish (Hodgepodge DSU mixins, GTNHLib) → `runtimeOnlyNonPublishable`. Hodgepodge late-mixin phase will then see `MINEFACTORY_RELOADED` / the real modid.

## ForgeMultipart / CodeChickenCore / CodeChickenLib

No standalone `CodeChickenLib` on GTNH Maven — `codechicken.lib.*` lives in `CodeChickenCore`.

GTNH `ForgeMultipart` POM lists `CodeChickenCore:dev` as **runtime** only, so `compileOnly("…:ForgeMultipart:…:dev")` will not see `codechicken.lib.data` / `IChunkLoadTile`. Add:

```
compileOnly("com.github.GTNewHorizons:CodeChickenCore:<latest non-pre>:dev")
```
