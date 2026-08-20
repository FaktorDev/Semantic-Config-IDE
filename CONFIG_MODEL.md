# Configuration Model

This document describes how Semantic Config IDE interprets source files.

## Source file

A source file is editable JSONC:

- `.jsonc` is recommended because directives are comments;
- `.json` can be imported, but adding directives makes the source logically JSONC;
- comments and directives are authoring metadata and can be removed during ready-files export.

A single file may contain:

- one root config array;
- multiple config arrays inside an object;
- reusable type declarations;
- reusable enum declarations;
- non-config supporting data.

### Non-config project files

Projects may also contain supporting files such as Markdown documentation. These files are editable project content but are not treated as JSONC semantic configs.

Semantic config discovery, directives, IR, schema generation, and config validation apply only to supported JSON/JSONC source files. Non-config files remain opaque to the config pipeline so a documentation file cannot accidentally create config diagnostics.

## JSON node attachment

A directive is a line comment beginning with `//@` and is attached to the next JSON node.

```jsonc
//@desc Name shown to the player.
//@public
"DisplayName": ""
```

The attachment target may be a property, array, object, or array element depending on the directive.

## Logical config

A logical config should be declared by `//@config <ConfigName>` on an array. The current implementation also has legacy implicit discovery: a root array can use the filename as its config key, and a top-level object array can use the property name. Explicit declarations are required for robust migrated output.

Root array:

```jsonc
//@config Items
[
  // elements
]
```

Array-valued property:

```jsonc
{
  //@config Items
  "Items": [
    // elements
  ]
}
```

Arrays in multiple files can join the same logical config when they use the same config name. They then share:

- one logical identity namespace;
- one effective template;
- one primary-key index;
- cross-file validation and code generation.

Do not merge arrays only because they have similar shapes.

### Legacy implicit discovery

For compatibility, the IDE can infer configs from:

- a root JSON array, using the filename without `.json/.jsonc`;
- top-level array properties, using the property name.

When a file contains `@global_type`, `@local_type`, `@global_enum`, or `@local_enum` declarations and has no explicit `@config`, implicit top-level array discovery is suppressed to avoid treating declaration arrays as configs.

Migration rule: add explicit `@config` to every intended logical config.

## Config part

A config part is one physical array contributing to a logical config. This distinction matters when a logical config is split across files.

Example:

```text
configs/items/weapons.jsonc  -> Items
configs/items/food.jsonc     -> Items
configs/items/template.jsonc -> Items with explicit template
```

The IDE can combine these parts for semantic analysis while retaining physical file paths.

## Explicit template

A template is one config array element marked with `//@template`.

```jsonc
{
  //@template
  "GameName": "",
  "DisplayName": ""
}
```

The template defines the expected runtime entity shape and drives:

- inferred IR;
- required/optional properties;
- validation;
- schema generation;
- code generation;
- completions and hover;
- many Quick Fixes and Global Fixes;
- public export projection.

Important behavior:

- only explicit `//@template` creates a template;
- the first array item is not automatically a template;
- only one explicit template is allowed per logical config;
- the template is authoring metadata, not runtime data;
- full cleaned export normally removes the template element unless template output is explicitly requested.

A config without an explicit template remains editable, but template-driven IR, validation, schema, and code generation are skipped.

## Runtime entity

Every non-template, non-ignored config element is runtime data.

```jsonc
{
  "GameName": "iron_sword",
  "DisplayName": "Iron Sword"
}
```

Template-only directives should not be copied into runtime entities. The runtime entity should contain values only, plus ordinary comments when useful.

## Primary key

`//@primary_key` marks a template property as the stable identifier for entities in that logical config.

Requirements:

- the template root must be an object;
- runtime values must be suitable key values;
- values must be unique within the complete logical config, including all files;
- the key should be stable across saves, references, and deployments.

A primary key is also implicitly public in `publicOnly` export unless explicitly overridden by `//@private`.

## Foreign key

`//@foreign_key TargetConfig` marks a scalar property or array property whose values refer to primary keys in another config.

Scalar:

```jsonc
//@foreign_key Items
"ResultItem": ""
```

Array:

```jsonc
//@foreign_key Items
"RequiredItems": []
```

For arrays, each array item is validated as a reference.

A foreign key is implicitly public in `publicOnly` export unless explicitly overridden by `//@private`.

## Reusable object types

Reusable types describe object shapes independent of logical configs.

Project-wide declaration inside a valid JSON object:

```jsonc
{
  //@global_type Price
  "PriceShape": {
    "Amount": 0,
    "Currency": "gold"
  }
}
```

File/local declaration:

```jsonc
{
  //@local_type DropRange
  "DropRangeShape": {
    "Min": 0,
    "Max": 1
  }
}
```

Reference from a template property:

```jsonc
//@ref_type Price
"Price": {
  "Amount": 0,
  "Currency": "gold"
}
```

When attached to an array property, `//@ref_type` applies to array items.

Reusable type declarations do not create configs and do not have a runtime identity namespace.

## Reusable enums

Project-wide:

```jsonc
{
  //@global_enum Rarity
  "RarityValues": ["Common", "Rare", "Epic"]
}
```

Local:

```jsonc
{
  //@local_enum DamageType
  "DamageTypeValues": ["Physical", "Fire", "Ice"]
}
```

Reference:

```jsonc
//@ref_enum Rarity
"Rarity": "Common"
```

For arrays, every item must be a member of the referenced enum.

## Scalar unions and reusable-type unions

Simple value alternatives:

```jsonc
//@one_of[string,number,bool]
"Value": ""
```

Reusable object alternatives:

```jsonc
//@union[FlatPrice,PercentPrice]
"PriceRule": {}
```

Use `//@one_of` for primitive/simple alternatives and `//@union` for named reusable object types.

## Optionality and nullability

`//@optional` means a property may be absent. In the current schema behavior it is also generated as nullable.

`//@nullable` allows an explicitly present value to be `null`.

Do not use these interchangeably when domain semantics distinguish missing from null. Confirm generated-consumer expectations.

## Ignored authoring data

`//@ignore_element` excludes an array element from runtime config processing.

`//@ignore_property` excludes a template property from runtime schema/codegen properties and cleaned runtime output.

These directives are different from `//@private`:

- `ignore_*` affects normal runtime processing/export;
- `private` affects only `publicOnly` export;
- full export still contains a private property, but not an ignored property in cleaned runtime output.

## Intermediate representation

The IDE builds an internal representation from the explicit template. Users do not edit this IR directly.

The IR carries:

- object, array, scalar, enum, ref, and union shapes;
- required/nullable metadata;
- constraints;
- generated names;
- primary/foreign-key metadata;
- export visibility metadata.

Schemas and generated code are derived from this representation.

## Public export projection

Visibility metadata does not remove data from the editor or full semantic model.

In `publicOnly` export:

- explicit `//@private` closes a property subtree;
- explicit `//@public` opens a property subtree;
- PK/FK are public by default;
- unmarked properties are private by default;
- unknown properties absent from the template IR are removed;
- templates, schemas, and generated code are projected as well as runtime JSON.

See [PUBLIC_EXPORT.md](./PUBLIC_EXPORT.md).
