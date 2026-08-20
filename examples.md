# GTNH template port — worked examples

Consult when the target is Calculator, SonarCore, SimplyJetpacks, AppleMilkTea2, or a similar CoFH / Japanese-plugin / Eclipse-dump tree. Workflow stays in [SKILL.md](SKILL.md); file IDs in [reference.md](reference.md).

## Calculator 1.9.11 (Ollie Lansdell)

After overlay: `build.gradle.kts` is `gtnhconvention`.

Identity:

- `modName` / `modId` = `Calculator` (case-sensitive)
- `modGroup` = `sonar.calculator.mod`
- `customArchiveBaseName` = `Calculator`
- `useModGroupForPublishing` = false
- `generateGradleTokenClass` = `sonar.calculator.mod.Tags`

No `version.properties` / usable tags → `gtnh.modules.gitVersion = false` and `addon.gradle` `ext.modVersion = "1.9.11"`; `Calculator.version = Tags.VERSION`.

Hard runtime: SonarCore. Jar `mcmod.info` `modid` is `SonarCore` (not `Sonar-Core`).

```
devOnlyNonPublishable(rfg.deobf("curse.maven:sonar-core-239418:2306173"))
```

`@Mod(dependencies = "required-after:SonarCore")`.

Optional `compileOnly` (`transitive = false`):

- `NotEnoughItems:2.8.125-GTNH:dev`
- `CraftTweaker:3.4.8:dev`
- `Applied-Energistics-2-Unofficial:rv3-beta-1035-GTNH:dev`
- AgriCraft `-dev` curse file in [reference.md](reference.md)
- `net.industrial-craft:industrialcraft-2:2.2.828-experimental:dev`
- CoFHLib deobf curse file in [reference.md](reference.md)

Waila / Natura were name-only — no API imports.

`runClient25` success:

- `LoadedMods […, Calculator, SonarCore, …]`
- join `Calculator@1.9.11`
- `Calculator: Successfully loaded with Sonar Core`
- `Forge Mod Loader has successfully loaded 17 mods`

Texture-missing is not a failure.

## SonarCore 1.1.3 (Ollie Lansdell)

After overlay: `build.gradle.kts` is `gtnhconvention`. FG 1.2 leftovers as `*.old`.

Identity:

- `modName` / `modId` = `SonarCore` (case-sensitive)
- `modGroup` = `sonar.core`
- `customArchiveBaseName` = `SonarCore`
- `useModGroupForPublishing` = false
- `generateGradleTokenClass` = `sonar.core.Tags`
- `apiPackage` = `api`

No tags; `gradle.properties.old` `mod_version=1.1.3` → `gtnh.modules.gitVersion = false` and `addon.gradle` `ext.modVersion = "1.1.3"`; `SonarCore.version = Tags.VERSION`.

`@Mod` has no `dependencies`. Old mcmod listed `CoFHLib` / `CoFHCore` / `CoFHAPI` / `WailaAPI` / `ForgeMultipart` — last two are not FML ids; CoFH/IC2/MFR APIs are vendored under `src/main/java`. `requiredMods` empty; `dependants` `Calculator`, `PracticalLogistics`.

Optional `compileOnly`:

- `NotEnoughItems:2.8.125-GTNH:dev`
- `Applied-Energistics-2-Unofficial:rv3-beta-1035-GTNH:dev`
- `ForgeMultipart:1.7.12:dev` **plus** `CodeChickenCore:1.4.17:dev` (FMP runtime-scopes CCC)
- `waila:1.19.31:dev`
- `LogisticsPipes:1.5.35-GTNH:dev`
- `StorageDrawers:2.2.28-GTNH:dev`
- Mekanism / Calculator curse files in [reference.md](reference.md) (`rfg.deobf`)

Circular: Calculator hard-depends on this mod — keep Calculator `compileOnly`.

`runClient25` success:

- `LoadedMods […, SonarCore, …]`
- `Forge Mod Loader has successfully loaded 16 mods`
- `SonarCore: Initilised API` / `Registered Events`
- `Sound engine started`

Optional-provider “wasn't loadable” + texture-missing are not failures. Artifact `SonarCore-1.1.3.jar`.

## SimplyJetpacks 1.5.3 (tonius11)

FG 1.2, no `gradle.properties`, no `dependencies {}`. Version `1.5.3` from `build.gradle`. AT via `jar.manifest FMLAT=simplyjetpacks_at.cfg`.

Identity:

- `modName` = `Simply Jetpacks` (mcmod; `@Mod` has no `name`)
- `modId` = `simplyjetpacks`
- `modGroup` = `tonius.simplyjetpacks`
- `customArchiveBaseName` = `SimplyJetpacks-MC1.7.10`
- `accessTransformersFile` = `simplyjetpacks_at.cfg`
- `generateGradleTokenClass` = `tonius.simplyjetpacks.Tags` (same package — no import)
- `gtnh.modules.gitVersion = false`; `addon.gradle` `ext.modVersion = "1.5.3"`

Overlay: zip did not replace `mcmod.info` or add ExampleMod sources. Removed leftover Groovy `build.gradle` after `.old` backup.

Runtime CoFHLib via `Class.forName("cofh.lib.util.helpers.FireworksHelper$Explosion")` — not `required-after`. Local `libs/CoFHLib-[1.7.10]1.0.2-160-dev.jar` replaced with:

```
devOnlyNonPublishable(rfg.deobf("curse.maven:cofh-lib-220333:2388747"))
```

`@Mod` `after:` ThermalExpansion / RedstoneArsenal / RArm / EnderIO / BuildCraft|Core — GameRegistry + IMC only, no type imports, no compileOnly.

`runClient25` success:

- `LoadedMods […, simplyjetpacks, …]`
- `Forge Mod Loader has successfully loaded 16 mods`
- `Starting Simply Jetpacks` / `Successfully found CoFHLib`
- `Sound engine started`

Texture-missing is not a failure. Artifact `SimplyJetpacks-MC1.7.10-1.5.3.jar`.

After green: dropped `*.old`, `LICENSE-template`, eclipse xml, unused `libs/` jar.

## AppleMilkTea2 1.7.10_2.9m (defeatedcrow)

Eclipse FG 1.2: root `java/` + `resources/`, no `gradle.properties`, no `src/main`. After overlay, moved to `src/main/{java,resources}`. Leftover Groovy `build.gradle` removed (`.old` was the backup).

Identity:

- `modName` = `Apple&Milk&Tea!`
- `modId` = `DCsAppleMilk` (case-sensitive)
- `modGroup` = `mods.defeatedcrow`
- `customArchiveBaseName` = `AppleMilkTea2`
- `apiPackage` = `api`
- `coreModClass` = `asm.AppleMilkCorePlugin` (old `FMLCorePlugin` manifest)
- `generateGradleTokenClass` = `mods.defeatedcrow.Tags`
- `useModGroupForPublishing` = false

`@Mod` / mcmod `1.7.10_2.9m` over stale gradle `1.7.10_2.8`. No usable tags → `gtnh.modules.gitVersion = false` and `addon.gradle` `ext.modVersion = "1.7.10_2.9m"`; `DCsAppleMilk` version = `Tags.VERSION`.

Original extra `@Mod` `DCsNEIPluginAMT` and DummyModContainer `AppleMilkCore` stayed — not ExampleMod leftovers.

Old ID expander pull-in deleted (`DepLoader`, `AMTdependencies.info`, `RequiredCoreModChecker`, PotionExtension login chat). `AppleMilkCorePlugin.getSetupClass()` → `null`. Hard runtime EndlessIDs:

```
devOnlyNonPublishable(rfg.deobf("curse.maven:fplib-665627:8169770"))
devOnlyNonPublishable(rfg.deobf("curse.maven:chunkapi-844484:8609071"))
devOnlyNonPublishable(rfg.deobf("curse.maven:endlessids-665730:8169819"))
```

`@Mod(dependencies = "required-after:Forge@[10.13.4.1448,);required-after:endlessids")`. Old mcmod `requiredMods` (`Minecraft Forge1.7.10 …`, `PotionExtention-…`) were not FML ids.

Optional `compileOnly` (`transitive = false`; curse files `rfg.deobf`):

- `NotEnoughItems:2.8.125-GTNH:dev` (not old CodeChickenLib — only `codechicken.nei.*`)
- `CodeChickenCore:1.4.17:dev`
- `AppleCore:3.3.11:dev`
- `ForestryMC:4.11.36:dev` (internals; GTNH package drift in [reference.md](reference.md))
- `BuildCraft:7.1.63:dev`
- `Railcraft:9.17.30:dev`
- IC2 experimental `:dev` (uses `ic2.core.block.TileEntityBarrel`)
- CoFHLib / Thaumcraft / BoP / CraftGuide / MCEconomy2 / SextiarySector2 curse files in [reference.md](reference.md)

Japanese, not on CurseForge — `~/Minecraft/Prop_mod/` (jars stay outside the repo; do not gitignore `*.jar` for this):

```
compileOnly(rfg.deobf(files("…/Prop_mod/Bamboo-2.6.8.5.jar")))
compileOnly(rfg.deobf(files("…/Prop_mod/Wa-1.7.10.17.jar")))
```

TofuCraft / MapleTree were OreDictionary / `Class.forName` only — no compileOnly. TE4 plugin was IMC only. Unused old `ejml` compile dropped. `disableCheckstyle = true` after 71 star-import failures.

`runClient25` success:

- `LoadedMods [DCsAppleMilk, …, AppleMilkCore, …, endlessids, …, DCsNEIPluginAMT, …, endlessids_potion, …]`
- `Forge Mod Loader has successfully loaded 27 mods`
- `Sound engine started`

Texture-missing is not a failure. Artifact `AppleMilkTea2-1.7.10_2.9m.jar`.

After green: dropped `*.old`, `LICENSE-template`.
