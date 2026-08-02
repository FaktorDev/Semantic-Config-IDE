# Quick Fixes

Quick Fixes are local Monaco editor actions attached to diagnostics. They are designed for small, focused corrections, usually around one issue.

## Implemented action families

The verified IDE snapshot includes builders for:

- add a missing required property;
- add an unknown property to the explicit template as optional;
- remove an additional/unsupported property;
- replace an invalid enum value with an allowed value;
- add an observed value to the enum directive in the explicit template;
- convert a value to the expected type;
- clamp one numeric value to a valid range;
- remove a broken/invalid value when the diagnostic permits it;
- insert a suggested directive such as primary key, optional, numeric type, or enum template;
- move a misplaced template-only directive to the explicit template;
- remove an invalid directive;
- insert `//@ignore_element` for a config element;
- reveal the issue location.

Availability depends on the diagnostic code, issue metadata, current file state, and whether an explicit template can be resolved.

## Example: add missing property

Template:

```jsonc
//@config Items
[
  {
    //@template
    //@primary_key
    "GameName": "",
    "DisplayName": ""
  },
  {
    "GameName": "iron_sword"
  }
]
```

The Quick Fix can insert:

```jsonc
{
  "GameName": "iron_sword",
  "DisplayName": ""
}
```

The inserted value may come from template/default/type fallback logic. Review domain-sensitive defaults.

## Example: add runtime property to template

When a runtime entity contains a legitimate additional property:

```jsonc
{
  "GameName": "iron_sword",
  "DisplayName": "Iron Sword",
  "Icon": "items/iron_sword.png"
}
```

A Quick Fix can update the explicit template with an optional property instead of deleting runtime data:

```jsonc
{
  //@template
  "GameName": "",
  "DisplayName": "",

  //@optional
  "Icon": ""
}
```

Use this only when the property is a real part of the domain model.

## Example: enum correction

Template:

```jsonc
//@enum[Common,Rare,Epic]
"Rarity": "Common"
```

Runtime value:

```jsonc
"Rarity": "Legendary"
```

Possible Quick Fixes:

- replace `Legendary` with an allowed value;
- add `Legendary` to the template enum.

These represent different domain decisions. Do not add values to a closed enum merely to silence an error.

## Example: misplaced directive

Incorrect runtime placement:

```jsonc
{
  "GameName": "iron_sword",
  //@optional
  "Description": ""
}
```

The move-directive Quick Fix can relocate `@optional` to the matching explicit template property.

## Local does not always mean one physical file

Some Quick Fixes resolve the explicit template in another physical file and apply a controlled project edit, for example adding a value to a cross-file template enum.

They are still issue-focused and smaller in scope than Global Fixes.

## Safety policy

Before applying a Quick Fix, decide whether the issue is caused by:

- invalid runtime data;
- an incomplete template;
- an overly strict constraint;
- a broken reference;
- an intentional exception;
- a migration artifact.

A syntactically valid fix can still be semantically wrong.

## What Quick Fixes should preserve

- JSONC validity;
- unrelated properties;
- comments and directives when possible;
- runtime identifiers;
- project structure;
- stable diagnostic codes and navigation.

## When to use a Global Fix instead

Use a Global Fix when:

- many entities have the same issue;
- references must be cascaded across files;
- files must be created/deleted/split;
- a rename affects a logical config;
- the operation requires a preview of broad changes.

See [GLOBAL_FIXES.md](./GLOBAL_FIXES.md).
