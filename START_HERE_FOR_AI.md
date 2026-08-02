# Start Here for AI — Adapting Configs to Semantic Config IDE

This file is the highest-priority guide when an AI is asked to adapt a project config archive to Semantic Config IDE.

## Required inputs

Use all available inputs:

1. this documentation repository;
2. the source config archive;
3. application code that reads the configs, when available;
4. generated models or schemas, when available;
5. the user's explicit decisions about compatibility, public/client data, and desired output layout.

Do not infer domain semantics from filenames alone when code, schemas, or references can resolve them.

## Goal

Produce IDE-compatible JSONC configs without changing the application's intended data semantics.

The result should normally include:

- converted `.jsonc` source files;
- explicit `//@config` declarations;
- one explicit `//@template` per logical config;
- justified primary and foreign keys;
- reusable types/enums where repetition is real and stable;
- constraints supported by evidence;
- optional public/private export annotations when requested;
- a migration report;
- a ready archive preserving the intended target structure.

## Hard rules

### Do not fabricate semantics

Never invent:

- a primary key merely because a property is named `Id`;
- a foreign key merely because two fields have similar names;
- enum restrictions from a small accidental sample;
- numeric ranges from observed minimum/maximum values alone;
- optionality because a property is absent from one malformed record;
- defaults without application or domain evidence;
- public visibility because a field looks harmless.

Record unresolved decisions instead.

### Preserve compatibility by default

Unless explicitly authorized:

- do not rename runtime properties;
- do not change identifier values;
- do not change number/string representation;
- do not split or merge configs in a way that changes consumer paths;
- do not normalize casing;
- do not remove unknown properties;
- do not rewrite comments unrelated to migration;
- do not replace missing values with guessed values.

### Explicit templates only

A valid migrated config needs an explicit template element:

```jsonc
//@config Items
[
  {
    //@template
    "GameName": "",
    "DisplayName": ""
  },
  {
    "GameName": "iron_sword",
    "DisplayName": "Iron Sword"
  }
]
```

Do not treat the first real entity as the template. Create a new template object or use an existing dedicated schema-like object only when its role is clear.

### Directives attach to the next JSON node

Place directives immediately before the property, array, object, or array element they describe.

Good:

```jsonc
//@primary_key
"GameName": ""
```

Unsafe/ambiguous:

```jsonc
//@primary_key
// unrelated comment or another node
"GameName": ""
```

Keep directive blocks contiguous with their target.

## Adaptation algorithm

### 1. Inventory the source

Create a table for every source file:

- path;
- format;
- root shape (`object`, `array`, scalar);
- candidate logical configs;
- runtime consumer, when known;
- repeated structures;
- identifier candidates;
- reference candidates;
- parse problems;
- migration risk.

Do not edit files before this inventory is complete.

### 2. Determine logical config boundaries

A logical config is normally a homogeneous collection of runtime entities.

Create explicit `//@config <Name>` for arrays that represent such collections. The current IDE can infer some root/top-level arrays for legacy compatibility, but migration output must not rely on filename/property-name inference.

Use the same config name across multiple files only when those arrays are intended to form one logical collection with one template and one identity namespace.

Do not use `//@config` for:

- arbitrary nested value arrays;
- lists of primitive values inside an entity;
- reusable object declarations;
- reusable enum declarations;
- unrelated arrays that merely share a shape.

### 3. Establish the explicit template

For each config:

1. collect all runtime shapes;
2. determine the stable superset required by consumers;
3. create one template object;
4. use representative values only for type inference and defaults;
5. mark truly non-required properties with `//@optional`;
6. mark null-capable properties with `//@nullable` where needed;
7. keep runtime entities separate from the template.

If records are polymorphic, prefer reusable types and `//@union[...]` rather than one unbounded object with many optional fields.

### 4. Resolve identity

A primary key must uniquely and stably identify an entity within the logical config.

Evidence can include:

- dictionary lookup keys in application code;
- references from other configs;
- save-game/storage identity;
- network identifiers;
- uniqueness tests across all records;
- domain documentation.

Add `//@primary_key` only after uniqueness and stability are established.

### 5. Resolve references

Add `//@foreign_key TargetConfig` when a scalar or array item stores primary-key values from another config.

Verify:

- target config name;
- target primary key;
- source value type;
- optional/null behavior;
- whether all existing references resolve;
- whether one field can reference more than one config.

Do not encode object embedding as a foreign key.

### 6. Extract reusable definitions

Use `//@global_type` for stable object shapes reused project-wide.

Use `//@local_type` for shapes only relevant to one file/config scope.

Use `//@global_enum` or `//@local_enum` for stable named value sets.

Use `//@ref_type` and `//@ref_enum` on template properties.

Do not create a reusable type for a single trivial object unless it improves generated naming or resolves recursion/polymorphism.

### 7. Add constraints conservatively

Supported constraints include:

- `//@enum[...]` or reusable enum references;
- `//@regex`;
- `//@clamp[...]`;
- `//@array_length(...)`;
- `//@uniqueBy[...]`;
- `//@choiceBetween(...)`;
- `//@type`;
- `//@defaultValue`;
- `//@optional`;
- `//@nullable`.

Add a constraint only when it is a domain rule, not merely a property of the current sample.

### 8. Add generated naming only where useful

Use:

- `//@name` for an explicit generated type/property name;
- `//@elements_name` for array element type names;
- `//@codegen` for language-specific declaration shape.

Do not rename runtime JSON keys to improve generated code. Use metadata directives instead.

### 9. Design public export when requested

For a client/server split:

- mark client-facing fields with `//@public`;
- remember that PK/FK are public by default;
- use `//@private` to override PK/FK or close an entire subtree;
- leave server-only unmarked fields private by default;
- verify templates, schemas, and generated code as well as runtime JSON.

See [PUBLIC_EXPORT.md](./PUBLIC_EXPORT.md).

### 10. Validate before packaging

At minimum verify:

- every edited file parses as JSONC;
- every logical config has exactly one explicit template;
- all primary keys are unique;
- all foreign keys resolve or are explicitly reported;
- template-only directives do not appear in runtime entities;
- aliases use supported canonical syntax;
- public export does not contain private sentinel names or values;
- folder structure and filenames match the requested replacement root.

## Decision policy

Use this order:

1. explicit user requirements;
2. application code and tests;
3. existing schemas/generated models;
4. domain documentation;
5. complete data analysis;
6. conservative unresolved issue.

Never reverse this order by preferring a guess over available evidence.

## Required output report

The final migration report must include:

- files changed, added, removed, or intentionally untouched;
- config mapping: source collection → `//@config` name;
- template decisions;
- PK/FK decisions and evidence;
- reusable types/enums introduced;
- constraints introduced;
- public/private decisions;
- compatibility changes, if any;
- validation performed;
- unresolved questions;
- known risks and next steps.

Use [MIGRATION_REPORT_TEMPLATE.md](./MIGRATION_REPORT_TEMPLATE.md).

## Packaging rule

When the user requests a replacement archive:

- place files at their real target paths;
- avoid an unnecessary intermediate patch folder;
- exclude caches, build outputs, dependencies, and VCS metadata unless requested;
- include the migration report inside the archive;
- provide an archive checksum when possible.
