# Examples

The `examples/` folder contains complete JSONC examples designed as migration references.

## Files

- [`examples/01-minimal-config.jsonc`](./examples/01-minimal-config.jsonc) — explicit config, template, PK, constraints, and runtime entities.
- [`examples/02-reusable-types-and-relations.jsonc`](./examples/02-reusable-types-and-relations.jsonc) — global type, global enum, two logical configs, FK, nested public data.
- [`examples/03-polymorphic-union.jsonc`](./examples/03-polymorphic-union.jsonc) — reusable types and union-based variants.
- [`examples/04-public-export.jsonc`](./examples/04-public-export.jsonc) — full/server data and client/public projection.
- [`examples/05-multi-file-template.template.jsonc`](./examples/05-multi-file-template.template.jsonc) and [`examples/05-multi-file-data.jsonc`](./examples/05-multi-file-data.jsonc) — one logical config split across files.

## Minimal pattern

```jsonc
{
  //@config Items
  "Items": [
    {
      //@template

      //@primary_key
      //@regex ^[a-z][a-z0-9_]*$
      "GameName": "",

      "DisplayName": "",

      //@clamp[0,999]
      "MaxStack": 1
    },
    {
      "GameName": "iron_sword",
      "DisplayName": "Iron Sword",
      "MaxStack": 1
    }
  ]
}
```

## Multi-file logical config

Two files can use the same config name:

```jsonc
// template file
{
  //@config Items
  "Items": [
    {
      //@template
      //@primary_key
      "GameName": "",
      "DisplayName": ""
    }
  ]
}
```

```jsonc
// data file
{
  //@config Items
  "Items": [
    {
      "GameName": "iron_sword",
      "DisplayName": "Iron Sword"
    }
  ]
}
```

Only one part should contain the explicit template.

## Reusable declaration pattern

Declarations are attached to JSON properties so the file remains valid single-root JSONC:

```jsonc
{
  //@global_enum Rarity
  "RarityValues": ["Common", "Rare", "Epic"],

  //@global_type Price
  "PriceShape": {
    "Amount": 0,
    "Currency": "gold"
  },

  //@config Items
  "Items": [
    {
      //@template
      //@primary_key
      "GameName": "",

      //@ref_enum Rarity
      "Rarity": "Common",

      //@ref_type Price
      "Price": {
        "Amount": 0,
        "Currency": "gold"
      }
    }
  ]
}
```

The property names `RarityValues` and `PriceShape` are declaration containers; semantic reference names are `Rarity` and `Price` from the directives.

## Anti-patterns

### First runtime record used as template

Incorrect:

```jsonc
//@config Items
[
  {
    "GameName": "iron_sword",
    "DisplayName": "Iron Sword"
  }
]
```

This has no explicit template. Do not add template-only directives to the real entity.

### Enum inferred from a tiny sample

Observed values `A` and `B` do not prove that only `A` and `B` are legal.

### Foreign key inferred by naming

A property named `OwnerId` may refer to a database user, external service, or embedded entity, not necessarily an IDE config.

### Public root without review

Marking a large root object public makes current and future descendants public unless overridden. Prefer narrow public fields when the contract is sensitive.

### Reusable declarations as multiple JSON roots

Invalid JSONC:

```jsonc
//@global_type Price
{ "Amount": 0 }

//@config Items
[]
```

Use one root object with declaration/config properties instead.
