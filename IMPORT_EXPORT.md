# Import / Export

Semantic Config IDE separates two different workflows:

1. Project Backup / Restore.
2. Ready Files Export / Import.

They should not be confused.

## Project Backup / Restore

Project export is for moving or backing up the full IDE project.

It includes IDE-specific information such as:

- project metadata;
- files;
- folders;
- tree structure;
- colors;
- settings;
- layout/session data if supported.

### Export Project

Creates one special JSON file:

```txt
<project-name>.semantic-config-project.json
```

Use this when you want to:

- back up an IDE project;
- move a project to another browser/device;
- preserve IDE metadata;
- restore project structure later.

### Import Project

Imports one special project JSON file.

This is not the same as importing raw `.json` files.

The importer should validate that the file is a Semantic Config IDE project bundle before restoring it.

## Ready Files Export

Ready files are production-oriented files intended for use in a game/application.

They should not contain IDE-only project metadata.

### Export Files: Project Structure

Keeps the project folder structure.

Example output:

```txt
configs/items.json
configs/units.json
configs/recipes.json
```

### Export Files: Flat Folder

Exports all ready files into one folder.

Example output:

```txt
items.json
units.json
recipes.json
```

If two files have the same name, the exporter should resolve the collision deterministically.

### Export Files: Single Combined Files

Creates a small set of combined files.

Example output:

```txt
all-configs.json
all-templates.json
all-schemas.json
generated-code.cs
generated-code.ts
```

`all-configs.json` should be a valid JSON object.

Example:

```json
{
  "Items": [],
  "Units": [],
  "Recipes": []
}
```

It must not be a custom metadata wrapper.

## Raw Files Import

Raw file import is for users who already have `.json` or `.jsonc` config files and want to start using the IDE.

### Import From Raw Files Archive

Input:

```txt
.zip archive with .json/.jsonc files
```

Expected behavior:

- read all supported files;
- preserve folder structure when possible;
- ignore unsupported files or report warnings;
- create editable IDE project files;
- do not require project bundle metadata.

## Recommended workflow

For IDE backup:

```txt
Export Project -> Import Project
```

For game/application usage:

```txt
Export Ready Files
```

For adopting existing configs:

```txt
Import From Raw Files Archive
```
