# Global Fixes

Global Fixes are project-wide or multi-file automated actions.

They are designed for operations that are too large or too risky for a single local Quick Fix.

## What Global Fixes can do

Examples:

- add missing properties across many files;
- remove repeated additional properties;
- clamp all out-of-range numeric values;
- convert numeric strings to numbers;
- rename config keys;
- rename config properties;
- rename primary key properties;
- rename primary key values and cascade references;
- fix broken foreign keys;
- split a config file by property;
- extract templates to a separate file.

## Preview-first model

Global Fixes should be previewed before they are applied.

A preview should show:

- affected files;
- created files;
- deleted files;
- text edits;
- warnings;
- possible conflicts.

## Example: rename primary key value

Before:

```jsonc
//@config Items
[
  {
    //@template
    //@primary_key
    "Id": ""
  },
  {
    "Id": "wood_old"
  }
]
```

References:

```jsonc
//@foreign_key Items
"RequiredItem": "wood_old"
```

Global Fix can rename:

```txt
wood_old -> wood
```

and update all foreign-key references.

## Example: split file by property

A file with mixed groups:

```jsonc
[
  {
    "Type": "Weapon",
    "Id": "sword"
  },
  {
    "Type": "Food",
    "Id": "apple"
  }
]
```

Global Fix can create separate files:

```txt
weapons.jsonc
food.jsonc
```

## Safety rules

Global Fixes should:

- never apply without a preview;
- use deterministic file names;
- avoid destructive edits unless explicitly confirmed;
- keep project files valid after apply;
- report conflicts instead of guessing;
- keep enough metadata for undo/review workflows.

## Difference between Global Fix and Import / Export

Global Fix changes the current IDE project.

Import / Export moves data into or out of the IDE.
