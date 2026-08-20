# GTNH template port — reference

Lookup tables. Workflow stays in [SKILL.md](SKILL.md).

## Artifact resolution

1. GTNH Maven: `com.github.GTNewHorizons:<Name>:<latest non-pre>:dev` or `:api`.
   - Authoritative latest is `maven-metadata.xml` `<release>`. Directory HTML listings truncate.
   - Browse `https://nexus.gtnewhorizons.com/service/rest/repository/browse/releases/com/github/GTNewHorizons/`.
   - Artifact names that differ: Forestry is `ForestryMC`. **Not** on this Maven: CoFHCore / CoFHLib, SonarCore, Calculator, AgriCraft, IC2, EndlessIDs / FalsePatternLib / ChunkAPI, Thaumcraft, BoP, CraftGuide, MCEconomy2, SextiarySector2 — CurseMaven, the IC2 `player.to` repo in SKILL.md §6, or `~/Minecraft/Prop_mod/`.
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
| Biomes O' Plenty | 1.7.10-2.1.0.2308 | `biomes-o-plenty-220318:2499612` | obfuscated universal; `rfg.deobf` |
| Thaumcraft | 1.7.10-4.2.3.5 | `thaumcraft-223628:2227552` | obfuscated; `rfg.deobf` |
| CraftGuide | 1.7.1.0-forge | `craftguide-75557:2459319` | obfuscated; `rfg.deobf`. Project id is `75557` not `75553` |
| MCEconomy2 | 2.5.4.a-1.7.0 | `mceconomy2-242484:2294405` | obfuscated; `rfg.deobf` |
| SextiarySector2 | 2.4.8.b-1.7.10 | `sextiarysector2-244083:2294538` | obfuscated; `rfg.deobf`. Needs MCEconomy2 at runtime if SS2 itself is runtime |
| EndlessIDs | 1.7.4 | `endlessids-665730:8169819` | obfuscated; `rfg.deobf`. FML id `endlessids`. Requires FalsePatternLib + ChunkAPI. UniMixins is a run-task shim |
| FalsePatternLib | 1.12.2 | `fplib-665627:8169770` | obfuscated; `rfg.deobf`. FML id `falsepatternlib`. EndlessIDs wants `falsepatternlib@[1.12.2,)` |
| ChunkAPI | 0.8.4 | `chunkapi-844484:8609071` | obfuscated; `rfg.deobf`. FML id `chunkapi`. EndlessIDs wants `chunkapi@[0.8.3,)` |

Discover unknown curse ids via `https://api.cfwidget.com/minecraft/mc-mods/<slug>` (`id` = projectId, `files[].id` = fileId). Always confirm the file is 1.7.10.

## Overlay zip (master-packages)

Does replace: wrapper, `settings.gradle.kts`, `build.gradle.kts`, `gradle.properties`, `dependencies.gradle`, `repositories.gradle`, `.editorconfig`, `.github/`, `CODEOWNERS`, `jitpack.yml`, `.java-version`, `gradle-daemon-jvm.properties`, `LICENSE-template`.

Does **not**: replace `mcmod.info`, add `com/myname/mymodid` sources, delete Groovy `build.gradle`.

After unzip: `chmod +x gradlew`; remove leftover Groovy `build.gradle` (keep `.old`) so Gradle uses the kts convention script.

## FG 1.2 empty `dependencies {}`

Old `build.gradle` often has no `dependencies` block. Compile path was Eclipse + `libs/*-dev.jar`. Inventory `libs/` and `.classpath` / `eclipse-*-settings.xml`. Rewrite jars to GTNH Maven / CurseMaven — do not keep `files("libs/…")` when a coordinate exists. After a green verify, delete the rewritten local jars.

Version may live only as `version = "1.5.3"` in that Groovy script. AT may live only as `jar { manifest { attributes 'FMLAT': 'modid_at.cfg' } }`.

Eclipse dumps often have the **entire** source tree at repo-root `java/` + `resources/` with no `sourceSets` override. That is not an extra root — move to `src/main/{java,resources}` after overlay (SKILL.md §1). Extra `srcDir` is only for additional trees.

## Japanese mods — `~/Minecraft/Prop_mod/`

Some Japanese 1.7.10 mods are not on CurseForge (Bamboo `ruby.bamboo.*`, Wa `wa.block.Blocks`, TofuCraft if types are imported). Place those jars in `~/Minecraft/Prop_mod/` (outside the repo). Do not add `*.jar` to gitignore for this — jars are not in the repo. Do not copy them into `libs/`.

```
compileOnly(rfg.deobf(files("${System.getProperty('user.home')}/Minecraft/Prop_mod/Bamboo-2.6.8.5.jar")))
compileOnly(rfg.deobf(files("${System.getProperty('user.home')}/Minecraft/Prop_mod/Wa-1.7.10.17.jar")))
```

OreDictionary / `Class.forName` / IMC-only Japanese plugins (TofuCraft `tofuKinu`, MapleTree) still get **no** compileOnly jar.

## Old ID expanders → EndlessIDs

Delete the pull-in, do not keep it compiling:

- chickenbone `DepLoader` (`getSetupClass()`, `sun.misc.URLClassPath`)
- `AMTdependencies.info` / `AMTdependancies.info` (PotionExtension dropbox)
- `RequiredCoreModChecker` / login chat that PotionExtension is missing

`@Mod` `required-after:endlessids`. Runtime:

```
devOnlyNonPublishable(rfg.deobf("curse.maven:fplib-665627:8169770"))
devOnlyNonPublishable(rfg.deobf("curse.maven:chunkapi-844484:8609071"))
devOnlyNonPublishable(rfg.deobf("curse.maven:endlessids-665730:8169819"))
```

Do not add UniMixins — `runClient25` already shims it. `LoadedMods` will show `endlessids`, `endlessids_potion`, `falsepatternlib`, `chunkapi`, plus this mod.

GTNH Maven `NotEnoughIds` is a different expander. Do not substitute it when the user asked for EndlessIDs.

## GTNH Forestry package drift

GTNH `ForestryMC` (not `Forestry`):

- `forestry.core.vect.Vect` → `forestry.core.utils.vect.Vect` / `IVect` (`getX()`/`getY()`/`getZ()`)
- FarmLogic `cultivate` / `harvest` take `forestry.api.farming.FarmDirection`, not `ForgeDirection`
- `isAcceptedWindfall(ItemStack)` is abstract on `FarmLogic` — implement it
- Internals (`forestry.farming.logic.Crop`, `forestry.core.proxy.Proxies`) need `:dev`

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

`codechicken.nei.*` is NotEnoughItems, not CodeChickenLib. Old `compile "codechicken:CodeChickenLib:…:dev"` is waste if the only chicken types are NEI.
