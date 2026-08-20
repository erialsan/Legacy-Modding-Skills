# GTNH template port — worked examples

Consult when the target is Calculator, SonarCore, or a similar CoFH/optional-compat tree. Workflow stays in [SKILL.md](SKILL.md); file IDs in [reference.md](reference.md).

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
