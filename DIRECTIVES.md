# Semantic Config IDE Directives

Directives are JSONC comments that start with `//@`.

They add semantic meaning to plain JSONC files without changing the runtime JSON data model.

```jsonc
//@config Items
[
  {
    //@template
    //@primary_key
    "Id": "",
    "Name": ""
  },
  {
    "Id": "sword",
    "Name": "Sword"
  }
]
```

## Basic syntax

```jsonc
//@directive_name
//@directive_name value
//@directive_name(value)
//@directive_name[value1,value2,value3]
```

Directives must be placed immediately before the JSON node they describe.

## Core concepts

### Config

A config is a logical collection of entities.

```jsonc
//@config Items
[
  {
    //@template
    //@primary_key
    "Id": "",
    "Name": ""
  }
]
```

Only `//@config` creates a config.

These directives do not create configs:

```jsonc
//@global_type
//@local_type
//@ref_type
//@global_enum
//@local_enum
//@ref_enum
```

### Template

A template defines the structure of runtime entities.

```jsonc
[
  {
    //@template
    //@primary_key
    "Id": "",
    "Name": "",
    "Weight": 1
  },
  {
    "Id": "sword",
    "Name": "Sword",
    "Weight": 3
  }
]
```

The template is not runtime data. It is used for validation, schema generation, code generation, completions and quick fixes.

Target Stage 7 behavior:

```txt
Only explicit //@template creates a template.
If a config has no //@template, the first array element must not be treated as a template.
```

## Config directives

### `//@config <ConfigName>`

Creates or joins a logical config.

```jsonc
//@config Items
[
  {
    //@template
    //@primary_key
    "Id": "",
    "Name": ""
  }
]
```

Multiple files may contribute to the same config by using the same config name.

### `//@template`

Marks one array element as the template.

```jsonc
{
  //@template
  //@primary_key
  "Id": "",
  "Name": ""
}
```

### `//@primary_key`

Marks a property as the unique identifier of an entity.

```jsonc
{
  //@template
  //@primary_key
  "Id": ""
}
```

### `//@foreign_key <ConfigName>`

Marks a property as a reference to another config.

```jsonc
//@config Recipes
[
  {
    //@template
    //@primary_key
    "Id": "",

    //@foreign_key Items
    "ResultItemId": ""
  }
]
```

For arrays:

```jsonc
//@foreign_key Items
"RequiredItems": ["wood", "iron"]
```

Each array item should reference an existing primary key in the target config.

## Property validation directives

### `//@type <type>`

Forces a property type.

```jsonc
//@type int
"Level": 1
```

Common primitive types:

```txt
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

### `//@optional`

Marks a property as not required.

```jsonc
//@optional
"Description": ""
```

### `//@nullable`

Allows `null`.

```jsonc
//@nullable
"Icon": null
```

### `//@defaultValue <value>`

Declares a default value.

```jsonc
//@defaultValue 1
"Weight": 1
```

### `//@enum[...]`

Declares inline enum values.

```jsonc
//@enum[Common,Rare,Epic,Legendary]
"Rarity": "Common"
```

### `//@clamp(min,max)`

Constrains a number.

```jsonc
//@clamp(0,100)
"Health": 100
```

### `//@regex <expression>`

Validates a string with a regular expression.

```jsonc
//@regex ^[a-z0-9_]+$
"Id": "iron_sword"
```

### `//@desc <text>`

Adds a human-readable description.

```jsonc
//@desc Item display name shown in UI.
"Name": ""
```

## Reusable types

Reusable types allow shared object shapes.

### `//@global_type <TypeName>`

Declares a type available across the project.

```jsonc
//@global_type Price
{
  "Amount": 0,
  "Currency": "gold"
}
```

### `//@local_type <TypeName>`

Declares a type available only within the current config or file scope.

```jsonc
//@local_type DropRange
{
  "Min": 0,
  "Max": 1
}
```

### `//@ref_type <TypeName>`

References a reusable type.

Scalar object usage:

```jsonc
//@global_type Price
{
  "Amount": 0,
  "Currency": "gold"
}

//@config Items
[
  {
    //@template
    //@primary_key
    "Id": "",

    //@ref_type Price
    "Price": {
      "Amount": 10,
      "Currency": "gold"
    }
  }
]
```

Array usage:

```jsonc
//@global_type Modifier
{
  "Type": "",
  "Value": 0
}

//@ref_type Modifier
"Modifiers": [
  {
    "Type": "tax",
    "Value": 0.2
  }
]
```

Target Stage 7 behavior:

```txt
If //@ref_type is placed on an array property, the referenced type applies to array items.
```

## Reusable enums

Reusable enums allow shared value sets.

### `//@global_enum <EnumName> [A,B,C]`

Declares an enum available across the project.

```jsonc
//@global_enum Rarity [Common,Rare,Epic,Legendary]
```

### `//@local_enum <EnumName> [A,B,C]`

Declares an enum available only within the current config or file scope.

```jsonc
//@local_enum DamageType [Physical,Fire,Ice,Poison]
```

### `//@ref_enum <EnumName>`

References a reusable enum.

Scalar usage:

```jsonc
//@global_enum Rarity [Common,Rare,Epic,Legendary]

//@ref_enum Rarity
"Rarity": "Common"
```

Array usage:

```jsonc
//@global_enum Tag [Weapon,Material,Food]

//@ref_enum Tag
"Tags": ["Weapon", "Material"]
```

Target Stage 7 behavior:

```txt
If //@ref_enum is placed on an array property, every array item must be a string from the referenced enum.
```

## Union

### `//@union[...]`

Allows a property to match one of several alternatives.

```jsonc
//@union[string,int]
"Value": "10"
```

For reusable types:

```jsonc
//@union[FlatPrice,PercentPrice]
"PriceRule": {
  "Amount": 10
}
```

## Sorting directives

### `//@sort_elements [...]`

Sorts array elements by one or more keys.

```jsonc
//@config Environments
//@sort_elements ["GameName"]
[
  { "GameName": "forest" },
  { "GameName": "desert" }
]
```

### `//@first`

Pins an element before regular sorted elements.

```jsonc
//@first
{
  "GameName": "default"
}
```

## Ignore directives

### `//@ignore_element`

Excludes an array element from generated/runtime output.

```jsonc
//@ignore_element
{
  "Id": "debug_only"
}
```

### `//@ignore_property`

Excludes a property from generated/runtime output.

```jsonc
//@ignore_property
"DebugNote": "Only for designers"
```

## Recommended config structure

```jsonc
//@global_enum Rarity [Common,Rare,Epic]

//@global_type Price
{
  "Amount": 0,
  "Currency": "gold"
}

//@config Items
[
  {
    //@template
    //@primary_key
    "Id": "",

    "Name": "",

    //@ref_enum Rarity
    "Rarity": "Common",

    //@ref_type Price
    "Price": {
      "Amount": 1,
      "Currency": "gold"
    }
  },
  {
    "Id": "sword",
    "Name": "Sword",
    "Rarity": "Common",
    "Price": {
      "Amount": 10,
      "Currency": "gold"
    }
  }
]
```

## Known limitations

Stage 7 is intended to stabilize the behavior of reusable types/enums and remove legacy implicit template fallback.

Until all Stage 7 implementation steps are complete, some docs may describe target behavior that is being aligned in the next implementation stages.
