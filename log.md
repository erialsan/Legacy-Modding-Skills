# Bundle Update Log

## 2026-11-15

- **Rename**: Named the [shared skill entry](./skills/Legacy-Modding-Skills/SKILL.md) Legacy-Modding-Skill.
- **Split**: Organized the same repository under `skills/Legacy-Modding-Skills/` with separate [1.7.10 GTNH](./skills/Legacy-Modding-Skills/1.7.10/SKILL.md) and [1.12.2 CleanroomMC](./skills/Legacy-Modding-Skills/1.12.2/SKILL.md) playbooks, as confirmed by the user. No independent Git repositories were created.
- **Addition**: Distinguished CleanroomModTemplate/CleanroomGradle from ForgeDevEnv/RetroFuturaGradle using their upstream READMEs, properties, and builds. Kept runtime compatibility claims separate from template selection.
- **Extraction**: Moved preservation/verification rules, dependency reasoning, and illustrative patterns into [shared knowledge](./skills/Legacy-Modding-Skills/common/index.md). Kept template-specific property names and commands in versioned playbooks.
- **Generalization**: Replaced individual-mod port histories and fixed artifact tables with reusable decisions. Retained representative common dependency names, including CoFHCore/CoFHLib; removed developer-specific jar paths and unconditional ID-expander requirements.
- **Conversion**: Added OKF v0.2 concept types, titles, descriptions, source provenance where applicable, and progressive-disclosure indexes. The root index declares the format version; nested indexes have no frontmatter.
- **Trust**: No human-verification or tested-mod claims are attached to these concepts. Structural format validation does not establish that migration recipes have been executed against a mod.
