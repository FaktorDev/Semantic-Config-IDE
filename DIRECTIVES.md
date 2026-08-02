# Directive Reference

Directives are JSONC line comments that begin with `//@`. They add semantic metadata without changing ordinary JSON values.

This reference uses canonical spellings implemented by the verified IDE snapshot.

## Attachment and placement

A directive applies to the next JSON node.

```jsonc
//@public
"DisplayName": ""
```

Multiple directives may be stacked:

```jsonc
//@foreign_key Items
//@private
//@optional
"HiddenFallbackItem": null
```

Keep directives immediately adjacent to their target. Prefer one directive per line.

## Scope vocabulary

| Scope | Meaning |
|---|---|
| File/declaration | Reusable declaration or top-level semantic declaration. |
| Config array | The array marked with `//@config`. |
| Template element | The single array element marked `//@template`. |
| Template property | A property inside the explicit template element. |
| Array property | A property whose JSON value is an array. |
| Runtime element | A non-template config entity. Template-only directives are invalid here. |

## Canonical directive index

| Directive | Main target | Purpose |
|---|---|---|
| `//@config Name` | array | Declares or joins a logical config. |
| `//@template` | config array element | Marks the explicit source-of-truth template. |
| `//@primary_key` | template property | Declares the config identity property. |
| `//@foreign_key Config` | template property | Declares references to another config PK. |
| `//@public` | template property | Includes property in public-only export. |
| `//@private` | template property | Excludes property subtree from public-only export. |
| `//@type Type` | template property | Overrides inferred scalar/codegen type. |
| `//@optional` | template property | Makes property non-required; current schema also permits null. |
| `//@nullable` | template property | Allows explicit `null`. |
| `//@defaultValue Value` | template property | Declares default metadata for schema/codegen/fixes. |
| `//@enum[...]` | template property | Declares inline allowed values. |
| `//@regex Pattern` | template property | Validates strings or string-array items. |
| `//@clamp[...]` | template property | Declares numeric bounds. |
| `//@array_length(...)` | config/template array | Declares array length bounds. |
| `//@uniqueBy[...]` | config/template array | Requires unique property combinations. |
| `//@choiceBetween("id", n)` | template properties | Declares mutually exclusive property groups. |
| `//@one_of[...]` | template property | Declares simple/scalar alternatives. |
| `//@union[...]` | template property | Declares alternatives between reusable types. |
| `//@global_type Name` | object declaration | Declares project-wide reusable object shape. |
| `//@local_type Name` | object declaration | Declares local reusable object shape. |
| `//@ref_type Name` | template property | Reuses an object type; on arrays applies to items. |
| `//@global_enum Name` | array declaration | Declares project-wide reusable values from the attached array. |
| `//@local_enum Name` | array declaration | Declares local reusable values from the attached array. |
| `//@ref_enum Name` | template property | Reuses enum; on arrays applies to each item. |
| `//@name Name` | template/property | Sets generated declaration/property naming metadata. |
| `//@elements_name Name` | array | Sets generated array element type name. |
| `//@items Name` | array | Legacy alias behavior for element type naming. |
| `//@codegen Target` | template/property | Controls generated declaration shape per language. |
| `//@desc Text` | array/template/property | Adds human-readable metadata. |
| `//@order Number` | template property | Controls property order during sorting. |
| `//@sort_properties` | template element | Enables template-based property sorting. |
| `//@sort_elements [...]` | array | Defines element sort keys in priority order. |
| `//@sort_config Property` | array | Legacy one-key form of `sort_elements`. |
| `//@first [priority]` | array element | Pins an element before ordinary sorted elements. |
| `//@ignore_property` | template property | Removes property from normal runtime processing/output. |
| `//@ignore_element` | array element | Removes element from normal runtime processing/output. |

## Config and template directives

### `//@config <ConfigName>`

Explicitly declares an array as a logical config. Arrays in multiple files using the same canonical name contribute to one config.

The current IDE retains legacy implicit discovery for root and top-level arrays, but migrated configs should always use explicit `@config`. A root array without `@config` derives its key from the filename; a top-level property array derives it from the property name. Reusable declaration files suppress this implicit discovery unless they also contain explicit config declarations.

```jsonc
//@config Items
[
  // template and runtime entities
]
```

or:

```jsonc
{
  //@config Items
  "Items": []
}
```

Rules:

- target must be an array;
- only `@config` creates a config;
- reusable type/enum declarations do not create configs;
- use stable domain names such as `Items`, `Recipes`, or `CharacterTraits`;
- do not reuse one config name for unrelated collections.

### `//@template`

Marks one config array element as the explicit template.

```jsonc
//@config Items
[
  {
    //@template
    "GameName": "",
    "DisplayName": ""
  }
]
```

Rules:

- exactly one explicit template is allowed per logical config;
- the first array element is not implicitly a template;
- template-driven IR/schema/validation/codegen are skipped if no explicit template exists;
- the template is removed from cleaned runtime output unless template export is enabled.

## Keys and references

### `//@primary_key`

Marks the stable unique identifier property.

```jsonc
//@primary_key
"GameName": ""
```

Rules:

- use on a property inside the explicit template;
- values must be unique across all physical parts of the logical config;
- the property is implicitly public in `publicOnly` export unless `//@private` overrides it.

### `//@foreign_key <TargetConfig>`

Declares references to the target config primary key.

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

For arrays, each array item is validated. The property is implicitly public in `publicOnly` export unless overridden with `//@private`.

## Export visibility

### `//@public`

Includes a template property in `publicOnly` export.

```jsonc
//@public
"DisplayName": ""
```

On objects and arrays, public visibility opens the subtree except for explicitly private descendants.

### `//@private`

Excludes a property and its complete subtree from `publicOnly` export.

```jsonc
//@primary_key
//@private
"InternalId": ""
```

Precedence:

```text
@private > @public > @primary_key/@foreign_key > unmarked
```

`@public` and `@private` affect export only. They do not change normal validation, full schema/codegen, or full export.

Using both on the same property is a diagnostic error. A public descendant inside a private ancestor is ineffective and diagnosed.

See [PUBLIC_EXPORT.md](./PUBLIC_EXPORT.md).

## Scalar and value constraints

### `//@type <type>`

Overrides inferred scalar type.

```jsonc
//@type int
"Level": 0
```

Canonical supported scalar names:

```text
int
long
float
double
decimal
string
bool
datetime
guid
```

The parser normalizes `boolean` to `bool`, but `bool` is preferred.

### `//@optional`

Marks a property as non-required.

```jsonc
//@optional
"Description": ""
```

Current generated-schema behavior also permits null. Confirm consumer semantics when missing and null must be distinguished.

### `//@nullable`

Allows explicit `null`.

```jsonc
//@nullable
"Icon": null
```

### `//@defaultValue <JSON value>`

Defines default metadata.

```jsonc
//@defaultValue 1
"Weight": 1
```

```jsonc
//@defaultValue "unknown"
"DisplayName": ""
```

Use canonical `@defaultValue`. Do not rely on the undocumented `@default_value` spelling.

### `//@enum[...]`

Declares inline allowed values.

```jsonc
//@enum[Common,Rare,Epic]
"Rarity": "Common"
```

Quoted values are supported:

```jsonc
//@enum["Light Armor","Heavy Armor"]
"ArmorClass": "Light Armor"
```

Extended mappings are supported for codegen/sort metadata:

```jsonc
//@enum[Common=1,Rare=2,Epic=3]
"Rarity": "Common"
```

When attached to an array property, each string item is constrained.

### `//@regex <pattern>`

Validates a string or each string-array item.

```jsonc
//@regex ^[a-z][a-z0-9_]*$
"GameName": ""
```

Keep patterns compatible with the JavaScript regular expression engine used by the IDE.

### `//@clamp<range>`

Declares numeric bounds.

```jsonc
//@clamp[0,100]
"HealthPercent": 100
```

Bound syntax:

- `[` / `]` includes the bound;
- `(` / `)` excludes the bound;
- either side may be omitted.

Examples:

```jsonc
//@clamp[0,100]
//@clamp[,3)
//@clamp(0,]
```

### `//@array_length(...)`

Restricts array length.

```jsonc
//@array_length(1,3)
"Rewards": []
```

Supported forms:

```text
@array_length(,3)   maximum 3
@array_length(1,)   minimum 1
@array_length(1,3)  between 1 and 3
@array_length(3)    exactly 3
```

### `//@uniqueBy[...]`

Requires config/array objects to be unique by one or more properties.

```jsonc
//@uniqueBy[GameName]
//@config Items
[
  // ...
]
```

Composite uniqueness:

```jsonc
//@uniqueBy[CharacterId,SkillId]
```

Multiple `uniqueBy` rules may be used when they express distinct uniqueness constraints.

### `//@choiceBetween("chooserId", groupId)`

Declares mutually exclusive property groups.

```jsonc
//@choiceBetween("cost", 1)
"FixedCost": 0,

//@choiceBetween("cost", 2)
"PercentCost": 0
```

Properties sharing the same chooser id participate in the same choice set. Use stable chooser ids and explicit numeric group ids.

## Reusable types and enums

### `//@global_type <TypeName>`

Declares a project-wide reusable object shape. Attach it to an object-valued property in a valid JSON document.

```jsonc
{
  //@global_type Price
  "PriceShape": {
    "Amount": 0,
    "Currency": "gold"
  }
}
```

### `//@local_type <TypeName>`

Declares a reusable object shape limited to local file/config scope.

```jsonc
{
  //@local_type DropRange
  "DropRangeShape": {
    "Min": 0,
    "Max": 1
  }
}
```

### `//@ref_type <TypeName>`

References a reusable object type.

```jsonc
//@ref_type Price
"Price": {
  "Amount": 0,
  "Currency": "gold"
}
```

On an array property, the referenced type applies to each item:

```jsonc
//@ref_type Modifier
"Modifiers": []
```

### `//@global_enum <EnumName>`

Declares a project-wide enum. Values come from the attached JSON array.

```jsonc
{
  //@global_enum Rarity
  "RarityValues": ["Common", "Rare", "Epic"]
}
```

### `//@local_enum <EnumName>`

Declares a local enum. Values come from the attached JSON array.

```jsonc
{
  //@local_enum DamageType
  "DamageTypeValues": ["Physical", "Fire", "Ice"]
}
```

### `//@ref_enum <EnumName>`

References a reusable enum.

```jsonc
//@ref_enum Rarity
"Rarity": "Common"
```

On arrays, every item must be a member:

```jsonc
//@ref_enum Tag
"Tags": []
```

## Union directives

### `//@one_of[...]`

Allows one of several simple types.

```jsonc
//@one_of[string,number,bool]
"Value": ""
```

Use canonical `one_of`. The parser also recognizes `oneOf`.

### `//@union[...]`

Allows one of several named reusable object types.

```jsonc
//@union[FlatPrice,PercentPrice]
"PriceRule": {}
```

Each listed name must resolve to a reusable type.

## Generated naming and documentation

### `//@name <Name>`

Sets an explicit generated name on a template or property.

```jsonc
//@name ItemConfig
```

Use this instead of renaming runtime JSON keys solely for generated-code style.

### `//@elements_name <Name>`

Sets the generated type name for array elements.

```jsonc
//@elements_name ResourceCost
"Costs": []
```

The parser also recognizes `elementsName`. Canonical `elements_name` is preferred.

### `//@items <Name>`

Legacy element naming directive. Prefer `//@elements_name` in new configs.

### `//@desc <text>`

Adds descriptive metadata used by hover, schema, and code generation.

```jsonc
//@desc Localized name displayed to the player.
"DisplayName": ""
```

### `//@codegen <target>`

Controls generated declaration shape without changing runtime data type.

Supported catalog values include:

```text
csharp:record
csharp:class
csharp:struct
typescript:interface
typescript:type
unity:serializable_class
unity:serializable_struct
```

Example:

```jsonc
//@template
//@codegen csharp:record
//@codegen typescript:interface
//@name ItemConfig
{
  "GameName": ""
}
```

Multiple language-specific `@codegen` directives may coexist when they target different generation strategies.

## Sorting and formatting

Sorting directives define IDE formatting behavior. They do not change domain meaning.

### `//@sort_elements ["A","B"]`

Sorts array elements by property/dotted-path priority.

```jsonc
//@sort_elements ["Category", "Level", "GameName"]
//@config Items
[
  // ...
]
```

### `//@sort_config <Property>`

Legacy one-property form that maps to `sort_elements`.

```jsonc
//@sort_config GameName
```

Prefer `sort_elements` for new configs.

### `//@first [priority]`

Pins an array element before ordinary sorted elements.

```jsonc
//@first
{
  "GameName": "default"
}
```

An optional numeric priority is parsed for ordering multiple first elements:

```jsonc
//@first -10
```

The explicit template remains ahead of runtime `@first` elements during template-aware sorting.

### `//@sort_properties`

Enables object property sorting according to template and directive rules.

```jsonc
{
  //@template
  //@sort_properties

  //@order 0
  "GameName": "",

  "DisplayName": "",

  //@order -1
  "Notes": ""
}
```

### `//@order <number>`

Controls property placement:

- non-negative values form the front group, ascending;
- negative values form the tail group, ascending;
- `-1` is closest to the end;
- template order is the stable fallback for known properties.

## Ignore directives

### `//@ignore_property`

Excludes a template property from normal runtime schema/codegen properties and cleaned runtime export.

```jsonc
//@ignore_property
"DesignerNote": ""
```

Use this for authoring-only properties that should not exist in normal runtime output.

Do not confuse it with `@private`, which remains present in full export.

### `//@ignore_element`

Excludes an array element from normal runtime processing/output.

```jsonc
//@ignore_element
{
  "GameName": "debug_example"
}
```

Useful for examples or disabled authoring records that should remain visible in source.

## Parser aliases and canonical policy

The current parser recognizes these compatibility aliases:

| Canonical | Recognized alias |
|---|---|
| `one_of` | `oneOf` |
| `elements_name` | `elementsName` |
| `array_length` | `arrayLength`, misspelled legacy `array_lenght` |
| `uniqueBy` | `unique_by` |
| `choiceBetween` | `choice_between` |
| `sort_elements` | legacy `sort_config` for one key |
| `type bool` | accepts `boolean`, normalizes to `bool` |

Use canonical forms in migrated files. Avoid relying on catalog-only or undocumented spellings such as `@default_value`.

## Placement errors to avoid

### Template-only directive on runtime data

Incorrect:

```jsonc
{
  "GameName": "iron_sword",
  //@primary_key
  "Id": "item_1"
}
```

Correct: move the directive to the matching property in the explicit template.

### Property directive on the template element itself

Incorrect:

```jsonc
//@public
{
  //@template
  "DisplayName": ""
}
```

Correct:

```jsonc
{
  //@template
  //@public
  "DisplayName": ""
}
```

### Multiple templates

Incorrect:

```jsonc
[
  { //@template
    "Id": ""
  },
  { //@template
    "Id": ""
  }
]
```

Only one template is allowed per logical config.
