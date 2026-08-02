# Config Adaptation Workflow

This workflow is for converting an existing config repository into Semantic Config IDE format while preserving application behavior.

## Phase 0 — Define migration boundaries

Before editing, record:

- source root;
- target root;
- whether filenames and folders must stay identical;
- whether runtime consumers can be changed;
- whether output must remain valid plain JSON;
- whether comments are important;
- server/full export requirements;
- client/public export requirements;
- desired generated languages;
- whether destructive normalization is allowed.

Default policy: preserve all runtime keys, values, paths, and collection boundaries.

## Phase 1 — Inventory

Create a source inventory.

| Field | Description |
|---|---|
| File | Relative path. |
| Root shape | Object, array, or scalar. |
| Collections | Candidate entity arrays. |
| Consumer | Code or subsystem reading it. |
| Identity | Candidate stable key and evidence. |
| References | Candidate links to other collections. |
| Variants | Different entity shapes. |
| Repetition | Candidate reusable types/enums. |
| Problems | Invalid JSON, mixed types, duplicates, broken refs. |
| Risk | Low/medium/high migration risk. |

Parse every file. Do not assume an archive is homogeneous.

## Phase 2 — Classify arrays

For each array, decide whether it is:

1. a logical config of entities;
2. a nested property array;
3. a reusable value list;
4. an enum-like declaration;
5. a tuple/fixed-position structure;
6. unrelated data that should remain unannotated.

Only case 1 normally receives explicit `//@config`. Do not rely on legacy implicit array discovery in migrated output.

### Good logical config candidates

- items;
- recipes;
- units;
- buildings;
- skills;
- quests;
- status effects;
- environment profiles.

### Usually not configs

- an entity's tags;
- ingredient lists;
- coordinate arrays;
- animation frames;
- numeric curves;
- nested modifiers owned by one entity.

## Phase 3 — Choose config names

Config names are semantic identifiers used by:

- cross-file merging;
- FK targets;
- navigation;
- generated artifacts;
- selection during export.

Rules:

- use stable PascalCase domain nouns;
- prefer plural names for entity collections;
- keep one name for one identity namespace;
- do not derive names from temporary folder paths;
- preserve existing canonical names when already used in code.

Example mapping:

```text
configs/items/weapons.json -> Items
configs/items/food.json    -> Items
configs/recipes.json       -> Recipes
```

## Phase 4 — Build one explicit template

### 4.1 Collect shape evidence

For every property across all runtime records, record:

- observed types;
- missing count;
- null count;
- distinct values;
- nested shapes;
- array item shapes;
- consumer access patterns;
- comments/domain descriptions.

### 4.2 Resolve type conflicts

Use these options:

- fix invalid data when evidence shows it is erroneous;
- use `//@one_of[...]` for real scalar alternatives;
- use reusable types plus `//@union[...]` for real object variants;
- split one mixed collection into separate configs only when consumer semantics permit it;
- leave an unresolved migration issue rather than flattening to `unknown` without review.

### 4.3 Choose template example values

Template values help inference but are not automatically defaults.

Recommended representatives:

```jsonc
{
  "Text": "",
  "Count": 0,
  "Ratio": 0.0,
  "Enabled": false,
  "OptionalObject": null,
  "Items": [],
  "Nested": {
    "Value": 0
  }
}
```

Use `//@type` when representative JSON numbers are insufficient for generated types, such as `int`, `long`, `float`, `decimal`, `datetime`, or `guid`.

### 4.4 Place the template

Inline template:

```jsonc
//@config Items
[
  {
    //@template
    "GameName": "",
    "DisplayName": ""
  },
  // runtime entities
]
```

Cross-file config parts may have the template in a dedicated sibling file. Ensure the same `//@config` name is used and only one effective template exists.

## Phase 5 — Required, optional, and nullable

Use evidence from both data and consumers.

| Domain meaning | Suggested directive |
|---|---|
| Property always required and non-null | no optional/nullable directive |
| Property may be absent | `//@optional` |
| Property is present but may be null | `//@nullable` |
| Property may be absent or null | `//@optional` and review current generated-schema null semantics |

Do not add `optional` simply to suppress existing bad data without deciding whether the data or template is wrong.

## Phase 6 — Primary keys

Evaluate candidate keys using:

- uniqueness across all config parts;
- non-null presence;
- stable value format;
- references in other files;
- code lookup behavior;
- persistence/network use;
- migration compatibility.

Run duplicate detection before adding `//@primary_key`.

If no stable key exists, do not invent one unless the user approves adding runtime fields and consumer changes.

## Phase 7 — Foreign keys

Build a reference matrix:

| Source config | Property | Target config | Cardinality | Optional/null | Broken count |
|---|---|---|---|---|---|

Example:

```jsonc
//@foreign_key Items
"ResultItemGameName": ""
```

For arrays:

```jsonc
//@foreign_key Items
"RequiredItemGameNames": []
```

Resolve whether broken references are:

- spelling mistakes;
- stale deleted IDs;
- optional sentinels;
- references to a different config;
- intentionally external values.

Do not blindly convert arbitrary strings to foreign keys.

## Phase 8 — Reusable types

Extract a reusable type when:

- the same object shape appears repeatedly;
- it has stable domain meaning;
- generated code benefits from a named type;
- multiple configs share it;
- it participates in a union;
- it is recursive or structurally complex.

Choose scope:

- `global_type` for project-wide use;
- `local_type` for one file/config concern.

Avoid premature extraction of tiny one-off objects.

## Phase 9 — Enums

Use inline `//@enum[...]` for a property-specific closed set.

Use reusable enums when the value set has a stable domain name and multiple consumers.

Before declaring an enum:

- inspect all existing values;
- inspect code constants/switches;
- determine case sensitivity;
- determine whether unknown future values are allowed;
- preserve numeric mappings when codegen requires them.

A list of currently observed values is not automatically a closed enum.

## Phase 10 — Constraints

Candidate evidence sources:

- validation code;
- database constraints;
- protocol/schema definitions;
- user-facing authoring rules;
- documented domain invariants;
- tests.

Use:

- `clamp` for true numeric bounds;
- `regex` for identifier/string grammar;
- `array_length` for true cardinality;
- `uniqueBy` for uniqueness within arrays;
- `choiceBetween` for mutually exclusive alternatives;
- `defaultValue` for intentional defaults used by tooling.

Do not infer hard constraints from current sample extrema.

## Phase 11 — Sorting and authoring quality

Sorting directives are optional. Add them when deterministic authoring order matters.

Common pattern:

```jsonc
//@sort_elements ["Category", "GameName"]
//@config Items
[
  {
    //@template
    //@sort_properties

    //@order 0
    "GameName": "",

    "Category": "",

    //@order -1
    "DesignerNote": ""
  }
]
```

Sorting must not change identity or runtime semantics.

## Phase 12 — Public/client contract

When client and server need different views:

1. identify the exact client contract;
2. mark only intentional client fields public;
3. review implicit PK/FK visibility;
4. hide sensitive PK/FK with `@private` where required;
5. decide visibility for reusable nested structures;
6. add server-only sentinel tests;
7. inspect projected JSON, templates, schemas, and generated code.

See [PUBLIC_EXPORT.md](./PUBLIC_EXPORT.md).

## Phase 13 — Validation passes

### Structural checks

- all files parse as JSONC;
- config directives target arrays;
- one explicit template per logical config;
- template root shapes are valid;
- template-only directives stay inside templates;
- reusable refs resolve;
- union members resolve.

### Data checks

- all required properties exist;
- value types match;
- enums match;
- clamps/regex/array lengths pass;
- PKs are present and unique;
- FKs resolve;
- uniqueness constraints pass;
- choice groups are valid.

### Export checks

- full cleaned export preserves required runtime data;
- template elements are excluded from runtime files;
- ignored authoring data is excluded;
- public export excludes all private data;
- generated schemas/code match the intended contract;
- output path collisions are reviewed.

## Phase 14 — Compatibility comparison

Compare source and migrated full runtime export semantically.

For each config:

- same runtime entity count, unless explicitly changed;
- same key set per entity, except approved cleanup;
- same scalar values and types;
- same nesting and array order unless sorting was approved;
- same filenames/paths if consumers depend on them.

A useful migration invariant is:

```text
Original runtime data == Full cleaned export of migrated source
```

Differences must be listed and justified.

## Phase 15 — Package and report

Deliver:

- ready project/config archive;
- migration report;
- unresolved questions;
- compatibility diff summary;
- validation results;
- recommended export settings;
- known risks and next steps.

Use [MIGRATION_REPORT_TEMPLATE.md](./MIGRATION_REPORT_TEMPLATE.md).
