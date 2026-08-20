# Semantic Config IDE — Documentation Contract

Semantic Config IDE is a browser IDE for designing, validating, refactoring, analyzing, versioning, and exporting large JSONC-based configuration systems.

This repository is the documentation contract for both human users and AI-assisted config adaptation. It is intentionally self-contained so it can be uploaded together with a config archive into a new AI chat.

## Current documentation baseline

This documentation is aligned with the `ide-frontend` `0.1.2` release scope.

- application package: `ide-frontend`;
- application version: `0.1.2`;
- release date: `2026-08-20`;
- documentation refresh: `2026-08-21`;
- config format: JSONC (`.json` and `.jsonc` semantic source files);
- supporting project files: Markdown and other non-config files can be stored without being treated as semantic configs;
- editor: Monaco;
- generated outputs: JSON, JSON Schema, C#, TypeScript, and supported codegen strategies;
- export visibility: `full` and `publicOnly`;
- project evolution: Project Groups, version comparison, semantic analysis, activity history, and immutable releases.

The documentation describes implemented behavior. Known limitations are explicitly marked rather than presented as future behavior.

## What's new in 0.1.2

`0.1.2` is a major expansion of the IDE beyond editing a single configuration snapshot.

Highlights:

- Project Groups and ordered project versions;
- `FROM` / `TO` version comparison with file, config, entity, and property-level analysis;
- standalone Project **Analyze** views;
- 52-week Project Activity history and day drilldown;
- `Draft` / `Released` / `Archived` version lifecycle;
- immutable release snapshots with canonical SHA-256 identity;
- public/private export contracts and deterministic export manifests;
- substantially more reliable large-project Context Worker and ZIP export behavior;
- Markdown editing, configurable indentation, Source Tree and Import/Export UX improvements.

See the full [Product Changelog](./CHANGELOG.md) and [Project Evolution](./PROJECT_EVOLUTION.md) guide.

## Start here

For an AI adapting a real project:

1. Read [START_HERE_FOR_AI.md](./START_HERE_FOR_AI.md).
2. Read [CONFIG_ADAPTATION_WORKFLOW.md](./CONFIG_ADAPTATION_WORKFLOW.md).
3. Use [DIRECTIVES.md](./DIRECTIVES.md) as the canonical directive reference.
4. Use [PUBLIC_EXPORT.md](./PUBLIC_EXPORT.md) when configs are shared with a client.
5. Use [IMPORT_EXPORT.md](./IMPORT_EXPORT.md) for Project transfer and Ready Files export.
6. Produce the report described in [MIGRATION_REPORT_TEMPLATE.md](./MIGRATION_REPORT_TEMPLATE.md).

For a human user:

- [CONFIG_MODEL.md](./CONFIG_MODEL.md) — how the IDE interprets files, configs, templates, runtime data, reusable types, and references;
- [DIRECTIVES.md](./DIRECTIVES.md) — canonical directive syntax and placement;
- [CONFIG_ADAPTATION_WORKFLOW.md](./CONFIG_ADAPTATION_WORKFLOW.md) — how to migrate an existing config repository;
- [PUBLIC_EXPORT.md](./PUBLIC_EXPORT.md) — server-full and client-public export rules;
- [IMPORT_EXPORT.md](./IMPORT_EXPORT.md) — Project transfer, raw import, Ready Files export, deterministic manifests, and conflicts;
- [PROJECT_EVOLUTION.md](./PROJECT_EVOLUTION.md) — Project Groups, Analyze, semantic history, Activity, and release lifecycle;
- [QUICK_FIXES.md](./QUICK_FIXES.md) — local diagnostic fixes;
- [GLOBAL_FIXES.md](./GLOBAL_FIXES.md) — preview-first multi-file operations;
- [EXAMPLES.md](./EXAMPLES.md) — complete examples and anti-patterns;
- [CHANGELOG.md](./CHANGELOG.md) — product release notes and user-visible changes;
- [CHAT_HANDOFF_PROMPT.md](./CHAT_HANDOFF_PROMPT.md) — ready prompt for a new AI chat;
- [DOCUMENTATION_UPDATE_REPORT.md](./DOCUMENTATION_UPDATE_REPORT.md) — documentation reconciliation status and remaining risks;
- [DOCUMENTATION_CHANGELOG.md](./DOCUMENTATION_CHANGELOG.md) — documentation-maintenance history, separate from the product changelog.

A machine-readable summary is available in [DOCUMENTATION_MANIFEST.json](./DOCUMENTATION_MANIFEST.json).

## Core model in one example

```jsonc
{
  //@global_enum ItemRarity
  "ItemRarityValues": ["Common", "Rare", "Epic"],

  //@global_type ItemVisual
  "ItemVisualShape": {
    //@public
    "Icon": "",

    "InternalAtlasId": 0
  },

  //@config Items
  //@sort_elements ["Rarity", "GameName"]
  "Items": [
    {
      //@template
      //@codegen csharp:record
      //@name ItemConfig

      //@primary_key
      "GameName": "",

      //@public
      "DisplayName": "",

      //@ref_enum ItemRarity
      "Rarity": "Common",

      //@ref_type ItemVisual
      "Visual": {
        "Icon": "",
        "InternalAtlasId": 0
      },

      //@clamp[0,100]
      "ServerDropWeight": 0
    },
    {
      "GameName": "iron_sword",
      "DisplayName": "Iron Sword",
      "Rarity": "Common",
      "Visual": {
        "Icon": "items/iron_sword.png",
        "InternalAtlasId": 41
      },
      "ServerDropWeight": 12
    }
  ]
}
```

The IDE interprets this as:

- one logical config named `Items`;
- one explicit template defining the entity shape;
- `GameName` as the primary key;
- a reusable enum and reusable object type;
- validation constraints;
- generated model metadata;
- public export metadata that does not change normal validation or full export.

## Non-negotiable config rules

1. New and migrated configs should use explicit `//@config` attached to an array. The IDE retains legacy implicit discovery for root/top-level arrays, but it is path-dependent and is disabled in reusable-declaration files without explicit configs.
2. A template is created only by explicit `//@template` on one array element.
3. The first runtime element must never be silently converted into a template.
4. Only one explicit template is allowed per logical config.
5. Template-only directives belong in the explicit template, not in runtime entities.
6. Primary keys and foreign keys must be based on actual domain identity and references, not guessed from property names alone.
7. Existing identifiers, casing, filenames, values, comments, and folder structure should be preserved unless a migration decision explicitly changes them.
8. `//@public` and `//@private` affect export only.
9. Public-only export is fail-closed: unknown or unmodeled properties are not safe to expose.
10. Large changes should be previewed and reported before destructive replacement.
11. Released and Archived Project versions are sealed; create a new Draft instead of mutating historical release state.

## What the IDE provides

### Config authoring

- JSONC parsing and directive diagnostics;
- explicit template-driven validation;
- cross-file logical configs;
- primary-key and foreign-key validation;
- reusable global/local object types and enums;
- scalar and reusable-type unions;
- constraints such as enum, regex, clamp, array length, uniqueness, optionality, and nullability;
- Markdown and non-config project files without semantic-config parsing;
- Monaco completion, hover, navigation, symbols, references, rename, and local Quick Fixes;
- preview-first Global Fixes for multi-file refactoring;
- JSON Schema and typed code generation.

### Project evolution

- Project Groups containing ordered independent Project versions;
- file-level `FROM` / `TO` comparison;
- semantic config, primary-key entity, and property-level change analysis;
- standalone Project analytics and quality views;
- local 52-week activity history;
- `Draft`, `Released`, and `Archived` lifecycle;
- immutable release snapshots with SHA-256 identity;
- creation of the next editable Draft from a release.

### Import and export

- Project backup/restore without Project Group metadata;
- raw ZIP import;
- Ready Files export with project, flat, and combined structures;
- full/server and client-safe `publicOnly` projection;
- deterministic export manifests and artifact digests;
- progress-aware, cancellable large ZIP exports.

## Scope and limitations

- This repository focuses on user-visible IDE/config contracts rather than internal React/Redux implementation details.
- `privateOnly` export is not implemented.
- Public export of ambiguous unions is intentionally conservative and may omit fields when a safe branch cannot be selected.
- Raw source JSONC cannot be public-only because private values would remain in the source text.
- A config without an explicit template can still be edited, but template-driven IR, schema, validation, and code generation are skipped for that config.
- Property-level semantic analysis compares arrays atomically unless an explicit safe element identity exists; it does not guess identity by index.
- Activity history is local to the current browser storage and begins only after tracking is initialized.
- Immutable releases store frozen Project snapshots and therefore consume additional local IndexedDB storage.
- Documentation aliases should not be treated as permission to invent new directive spellings. Prefer canonical syntax from [DIRECTIVES.md](./DIRECTIVES.md).
