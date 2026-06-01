# Quick Fixes

Quick Fixes are local editor actions shown near diagnostics in the JSONC editor.

They are intended for small, safe, focused changes inside the current file.

## What Quick Fixes can do

Examples:

- add a missing required property;
- remove an unsupported property;
- convert a value type;
- replace an invalid enum value;
- add a value to an enum;
- insert a directive;
- move a directive to the correct location;
- remove an invalid directive;
- reveal the issue location.

## Example: add missing property

Template:

```jsonc
[
  {
    //@template
    //@primary_key
    "Id": "",
    "Name": ""
  },
  {
    "Id": "sword"
  }
]
```

If `"Name"` is required, Quick Fix can add:

```jsonc
{
  "Id": "sword",
  "Name": ""
}
```

## Example: remove additional property

Template:

```jsonc
{
  //@template
  "Id": "",
  "Name": ""
}
```

Runtime entity:

```jsonc
{
  "Id": "sword",
  "Name": "Sword",
  "Unknown": 123
}
```

Quick Fix can remove `"Unknown"`.

## Example: enum value replacement

Template:

```jsonc
//@enum[Common,Rare,Epic]
"Rarity": "Common"
```

Runtime entity:

```jsonc
"Rarity": "Legendary"
```

Quick Fix can replace it with one of the allowed values.

## Safety rules

Quick Fixes should:

- keep edits minimal;
- preserve JSONC validity;
- preserve comments/directives when possible;
- not modify unrelated files;
- not silently change project structure;
- keep issue codes stable for diagnostics and automation.

## Difference between Quick Fix and Global Fix

Quick Fix:

```txt
Small local edit.
Usually affects one file and one issue.
```

Global Fix:

```txt
Project-wide or multi-file operation.
Requires preview before applying.
```
