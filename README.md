# Semantic Config IDE — Documentation Contract

Semantic Config IDE is a browser IDE for designing, validating, refactoring, and exporting large JSONC-based configuration systems.

This repository contains the documentation contract for adapting existing application or game configs to the IDE. It is intentionally self-contained so it can be uploaded together with a config archive into a new AI chat.

## Verified implementation snapshot

This documentation was reconciled with the supplied IDE frontend source snapshot:

- application package: `ide-frontend`;
- application version: `0.1.1`;
- verification date: `2026-08-02`;
- config format: JSONC (`.json` and `.jsonc` source files);
- editor: Monaco;
- generated outputs: JSON, JSON Schema, C#, TypeScript, and supported codegen strategies;
- export visibility: `full` and `publicOnly`.

The documentation describes implemented behavior, not a future roadmap. Any remaining limitations are explicitly marked.

## Start here

For an AI adapting a real project:

1. Read [START_HERE_FOR_AI.md](./START_HERE_FOR_AI.md).
2. Read [CONFIG_ADAPTATION_WORKFLOW.md](./CONFIG_ADAPTATION_WORKFLOW.md).
3. Use [DIRECTIVES.md](./DIRECTIVES.md) as the directive reference.
4. Use [PUBLIC_EXPORT.md](./PUBLIC_EXPORT.md) when configs are shared with a client.
5. Produce the report described in [MIGRATION_REPORT_TEMPLATE.md](./MIGRATION_REPORT_TEMPLATE.md).

For a human user:

- [CONFIG_MODEL.md](./CONFIG_MODEL.md) — how the IDE interprets files, configs, templates, runtime data, reusable types, and references;
- [DIRECTIVES.md](./DIRECTIVES.md) — canonical directive syntax and placement;
- [CONFIG_ADAPTATION_WORKFLOW.md](./CONFIG_ADAPTATION_WORKFLOW.md) — how to migrate an existing config repository;
- [PUBLIC_EXPORT.md](./PUBLIC_EXPORT.md) — server-full and client-public export rules;
- [IMPORT_EXPORT.md](./IMPORT_EXPORT.md) — backup, raw import, ready-files export, and conflict behavior;
- [QUICK_FIXES.md](./QUICK_FIXES.md) — local diagnostic fixes;
- [GLOBAL_FIXES.md](./GLOBAL_FIXES.md) — preview-first multi-file operations;
- [EXAMPLES.md](./EXAMPLES.md) — complete examples and anti-patterns;
- [CHAT_HANDOFF_PROMPT.md](./CHAT_HANDOFF_PROMPT.md) — ready prompt for a new chat.
- [DOCUMENTATION_UPDATE_REPORT.md](./DOCUMENTATION_UPDATE_REPORT.md) — what was corrected, validated, and what remains risky.

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

## Non-negotiable rules

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

## What the IDE provides

- JSONC parsing and directive diagnostics;
- explicit template-driven validation;
- cross-file logical configs;
- primary-key and foreign-key validation;
- reusable global/local object types and enums;
- scalar and reusable-type unions;
- constraints such as enum, regex, clamp, array length, uniqueness, optionality, and nullability;
- Monaco completion, hover, navigation, symbols, references, rename, and local Quick Fixes;
- preview-first Global Fixes for multi-file refactoring;
- JSON Schema and typed code generation;
- project backup/restore;
- ready-files export with project, flat, and combined structures;
- client-safe `publicOnly` export.

## Scope and limitations

- This repository documents config usage, not the internal React/Redux architecture of the IDE.
- `privateOnly` export is not implemented.
- Public export of ambiguous unions is intentionally conservative and may omit fields when a safe branch cannot be selected.
- Raw source JSONC cannot be public-only because private values would remain in the source text.
- A project without an explicit template can still be edited, but template-driven IR, schema, validation, and code generation are skipped for that config.
- Documentation aliases should not be treated as permission to invent new directive spellings. Prefer canonical syntax from [DIRECTIVES.md](./DIRECTIVES.md).
