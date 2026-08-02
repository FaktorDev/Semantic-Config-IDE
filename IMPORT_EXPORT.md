# Import and Export

Semantic Config IDE separates authoring-project backup from production-ready file export.

## 1. Project Backup / Restore

Project backup preserves IDE project state in a special JSON bundle.

Default filename pattern:

```text
<project-name>.semantic-config-project.json
```

Use it to:

- back up an IDE project;
- move it between browsers/devices;
- preserve IDE metadata and project structure;
- restore files as an IDE project.

This is not the same as ready-files export and should not be consumed by the game/application runtime.

## 2. Raw Files Import

Raw import accepts a ZIP archive containing `.json` and `.jsonc` files and creates editable IDE project files.

The importer can preserve archive paths under an optional target root.

### Conflict strategies

| Strategy | Behavior |
|---|---|
| `Skip existing` | Existing project files are kept; conflicting imports are skipped. |
| `Rename imported` | Imported paths are renamed deterministically to avoid collisions. |
| `Overwrite existing` | Existing files are replaced after explicit confirmation. |

### Import diagnostics

The import plan can report:

- existing-file conflicts;
- invalid paths;
- unsupported extensions;
- invalid JSONC;
- empty files;
- duplicate import paths;
- archive read failures.

Review the plan before applying overwrite operations.

### Recommended adoption workflow

1. create/select an empty IDE project;
2. import the raw ZIP using `Rename imported` or `Skip existing` for the first inspection;
3. inspect parse diagnostics;
4. add config/template semantics through a controlled migration;
5. only use overwrite after the target structure is verified.

## 3. Ready Files Export

Ready-files export creates runtime/application artifacts without IDE project metadata.

### Export modes

| Mode | Purpose |
|---|---|
| `Raw project files` | Original source JSONC, including comments/directives when preservation options allow it. Full visibility only. |
| `Cleaned runtime JSON` | Parsed JSON output with authoring-only elements removed according to options. Recommended runtime mode. |
| `Selected configs` | Exports only selected logical configs/files according to the export plan. |

### Structure modes

#### Keep project structure

Preserves relative folders:

```text
configs/items.json
configs/recipes.json
configs/units.json
```

Use when runtime loaders depend on project paths.

#### Flat folder

Writes ready files into one folder. Name collisions are resolved deterministically and reported.

Use only when path-based identity is not required.

#### Single combined files

Creates a small set of combined artifacts, commonly:

```text
all-configs.json
all-templates.json
all-schemas.json
generated-code.<language extension>
```

Combined configs are ordinary JSON data organized by config key, not an IDE project-backup wrapper.

## 4. Visibility modes

### Full — public + private

Exports all runtime properties. `@public` and `@private` do not filter full output.

### Public only

Exports only effective public properties:

- explicit `@public`;
- PK/FK unless overridden by `@private`;
- structural parent containers required by public descendants.

Unmarked properties are private.

Public-only mode forces clean generated output:

- raw source mode is disabled/normalized;
- comments are not preserved;
- directives are not preserved;
- runtime JSON, templates, schemas, and generated code are projected.

See [PUBLIC_EXPORT.md](./PUBLIC_EXPORT.md).

## 5. Artifact options

Depending on structure and mode, export can include:

- runtime config JSON;
- template files;
- JSON Schemas;
- generated code;
- manifests;
- selected files/configs;
- an optional archive root folder;
- configurable combined artifact names;
- JSON indentation from 0 to 8 spaces.

Do not include full schemas or full generated code in a public client package. Public-only mode must project them.

## 6. Templates and ignored data

Cleaned runtime export normally removes:

- explicit template elements;
- `@ignore_element` records;
- `@ignore_property` properties;
- comments/directives when not preserved.

`@private` is different:

- retained in full cleaned export;
- removed only in public-only export.

## 7. Export warnings

Important warnings/errors include:

- selected config has no files;
- selected config set is empty;
- selected file contains other configs;
- invalid/non-JSON file skipped;
- schema/codegen missing or failed;
- output path collision resolved;
- template export empty/failed;
- public export missing IR;
- public config skipped;
- no public properties;
- hidden PK/FK;
- raw mode normalized for public export;
- unfiltered artifact blocked.

Treat public-export errors as security-relevant. Do not distribute an archive with unexplained public-export errors.

## 8. Recommended export profiles

### Server runtime

```text
Mode: Cleaned runtime JSON
Structure: Keep project structure (or consumer-required structure)
Visibility: Full
Preserve comments: No
Preserve directives: No
Templates: No, unless server tooling needs them
Schema/code: optional build artifacts
```

### Client runtime

```text
Mode: Cleaned runtime JSON
Structure: consumer-required structure
Visibility: Public only
Preserve comments: No
Preserve directives: No
Templates/schema/code: only public-projected artifacts when needed
```

### IDE source handoff

```text
Use Project Backup / Restore
or provide the original JSONC source archive
```

Do not use cleaned runtime export as the only source backup because it may omit templates, comments, directives, and ignored authoring data.

## 9. Migration verification

After adapting existing configs:

1. export `Full` cleaned runtime JSON;
2. compare it semantically to the original runtime data;
3. export `Public only` when applicable;
4. search the entire archive for private sentinel names/values;
5. inspect warnings and artifact counts;
6. test the real runtime consumer with the exported files.
