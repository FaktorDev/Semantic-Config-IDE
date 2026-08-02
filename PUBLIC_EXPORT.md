# Public-Only Export

Public-only export creates a client-safe projection while full export remains available for server-side use.

## Modes

| Visibility mode | Behavior |
|---|---|
| `full` | Exports public and private properties. This is the backward-compatible default. |
| `publicOnly` | Exports only effective public properties and projected artifacts. |

`privateOnly` is not implemented.

## Visibility rules

Effective precedence:

```text
1. @private
2. @public
3. @primary_key or @foreign_key
4. unmarked property
```

In `publicOnly`:

- `@private` removes the property and its entire subtree;
- `@public` includes the property and opens its subtree;
- PK and FK properties are public unless overridden by `@private`;
- unmarked properties are private by default.

In `full`, `@public` and `@private` do not filter data.

## Example

```jsonc
//@config Items
[
  {
    //@template

    //@primary_key
    "GameName": "",

    //@public
    "DisplayName": "",

    "ServerDropWeight": 0,

    //@public
    "Visual": {
      "Icon": "",

      //@private
      "InternalAtlasId": 0
    },

    //@foreign_key Resources
    //@private
    "SecretRequiredResource": ""
  },
  {
    "GameName": "iron_sword",
    "DisplayName": "Iron Sword",
    "ServerDropWeight": 12,
    "Visual": {
      "Icon": "items/iron_sword.png",
      "InternalAtlasId": 41
    },
    "SecretRequiredResource": "server_meteorite"
  }
]
```

Public runtime result:

```json
[
  {
    "GameName": "iron_sword",
    "DisplayName": "Iron Sword",
    "Visual": {
      "Icon": "items/iron_sword.png"
    }
  }
]
```

## Nested behavior

### Public descendant retains structural parents

```jsonc
"Visual": {
  //@public
  "Icon": "",
  "InternalAtlasId": 0
}
```

Result:

```json
{
  "Visual": {
    "Icon": "..."
  }
}
```

The unmarked parent object is kept only as a structural container. Unmarked siblings are removed.

### Public parent opens descendants

```jsonc
//@public
"Visual": {
  "Icon": "",
  "Color": "",
  //@private
  "InternalAtlasId": 0
}
```

`Icon` and `Color` are included; `InternalAtlasId` is removed.

### Private parent closes the subtree

```jsonc
//@private
"ServerData": {
  //@public
  "DebugName": ""
}
```

The complete `ServerData` property is removed. A public descendant cannot reopen a private ancestor.

## Export-only semantics

Visibility directives do not change:

- editor data;
- normal/full validation;
- primary/foreign-key behavior;
- full JSON Schema;
- full code generation;
- full export.

They are metadata used only when building a public export projection.

## Artifacts filtered by public export

Public-only filtering applies to:

- cleaned runtime JSON;
- extracted template JSON;
- JSON Schema;
- generated C#;
- generated TypeScript and supported generated-code artifacts.

This is essential because a full schema or generated model would reveal private property names even if runtime values were removed.

## Raw source restriction

Raw JSONC source cannot be public-only. The original text still contains private names, values, comments, and directives.

When `publicOnly` is selected:

- raw source mode is unavailable/normalized to cleaned runtime JSON;
- comments are not preserved;
- directives are not preserved.

## Fail-closed security behavior

Public export is intentionally conservative:

- properties absent from the template IR are removed;
- additional/unmodeled runtime properties are removed;
- config parts without computed IR are skipped with export errors;
- schemas and generated code are rebuilt from projected IR instead of copied from full cached artifacts;
- raw reusable declarations are not blindly copied into combined public templates;
- ambiguous union branches use conservative projection.

A missing model must never cause fallback to full data.

## Union limitation

When runtime data can match multiple reusable union branches and no branch can be selected safely, public projection may keep only data allowed by all compatible branches.

Recommendation: use a stable discriminator property in polymorphic data and ensure it is public when the client needs it.

## Verification checklist

Before distributing a client archive:

1. export with `Property visibility = Public only`;
2. inspect export warnings;
3. verify removed-private statistics;
4. search every ZIP entry for known server-only property names;
5. search every ZIP entry for a unique server-only sentinel value;
6. inspect JSON, template, schema, and generated code artifacts;
7. ensure configs without IR were not silently exported in full;
8. compare client consumer requirements with the projected schema.

Recommended test value during migration:

```text
SERVER_SECRET_SENTINEL_DO_NOT_EXPORT
```

That value must not occur anywhere in the public archive.

## Visibility design guidance

Prefer marking small, stable client contracts public rather than marking an entire large root object public.

Good:

```jsonc
"Combat": {
  //@public
  "AnimationId": "",
  //@public
  "DisplayDamage": 0,
  "ServerFormula": ""
}
```

Riskier:

```jsonc
//@public
"Combat": {
  // dozens of fields, future fields become public unless explicitly private
}
```

Use a public parent when the whole subtree is intentionally part of the client contract and future additions will be reviewed.
