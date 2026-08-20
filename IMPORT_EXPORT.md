# Import and Export

Semantic Config IDE separates editable Project transfer from production-ready file export.

## 1. Project Import / Export

Project transfer preserves IDE Project state in a special JSON bundle.

Default filename pattern:

```text
<project-name>.semantic-config-project.json
```

Use it to:

- back up an IDE Project;
- move it between browsers/devices;
- preserve IDE metadata and source-tree structure;
- restore files as an editable IDE Project.

Project Import/Export is available from the Project management workflow in addition to the dedicated Import/Export area.

This format is not the same as Ready Files export and should not be consumed directly by the game/application runtime.

### Project Group metadata

Project transfer is intentionally independent from Project Groups.

A Project bundle does not need to carry group-family metadata such as:

- Project Group ID;
- version order;
- lifecycle state;
- release digest.

This keeps a Project portable as a standalone authoring unit.

For `Released`/`Archived` versions, immutable historical workflows use the frozen release snapshot where supported. See [PROJECT_EVOLUTION.md](./PROJECT_EVOLUTION.md).

## 2. Raw Files Import

Raw import accepts a ZIP archive containing supported project files. JSON/JSONC files can participate in semantic config analysis; Markdown can be kept as supporting documentation without being parsed as a config.

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

1. create/select an empty IDE Project;
2. import the raw ZIP using `Rename imported` or `Skip existing` for first inspection;
3. inspect parse diagnostics;
4. add config/template semantics through a controlled migration;
5. only use overwrite after the target structure is verified.

## 3. Ready Files Export

Ready Files export creates runtime/application artifacts without Project Group metadata.

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

Combined configs are ordinary JSON data organized by config key, not an IDE Project-backup wrapper.

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
- deterministic export manifest;
- selected files/configs;
- an optional archive root folder;
- configurable combined artifact names;
- JSON indentation from 0 to 8 spaces.

Do not include full schemas or full generated code in a public client package. Public-only mode must project them.

### Deterministic manifest

Ready Files exports can include a deterministic manifest describing the generated artifact set.

The manifest can include:

- export visibility mode;
- source SHA-256;
- artifact-set SHA-256;
- content revision;
- SHA-256 for individual generated artifacts.

Use these digests in integration/CI pipelines when reproducibility matters.

## 6. Templates and ignored data

Cleaned runtime export normally removes:

- explicit template elements;
- `@ignore_element` records;
- `@ignore_property` properties;
- comments/directives when not preserved.

`@private` is different:

- retained in full cleaned export;
- removed only in public-only export.

## 7. Large export behavior

Large exports are progress-aware and cancellable. The export pipeline reports meaningful stages instead of relying only on a fixed total timeout.

Stages can include:

```text
Preparing project
Runtime configs
Templates
Schemas
Code generation
Combined artifacts
Manifest
ZIP compression
```

The implementation uses stall detection so a large export is not considered failed merely because total runtime exceeds an older fixed timeout.

ZIP creation can use size/speed-oriented compression presets such as:

- Fast;
- Balanced;
- Smaller ZIP.

If an export is cancelled, the worker is terminated rather than allowing stale background completion to mutate UI state.

## 8. Export warnings

Important warnings/errors include:

- selected config has no files;
- selected config set is empty;
- selected file contains other configs;
- invalid/non-JSON file skipped for semantic export;
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

## 9. Recommended export profiles

### Server runtime

```text
Mode: Cleaned runtime JSON
Structure: Keep project structure (or consumer-required structure)
Visibility: Full
Preserve comments: No
Preserve directives: No
Templates: No, unless server tooling needs them
Schema/code: optional build artifacts
Manifest: recommended for CI/reproducibility
```

### Client runtime

```text
Mode: Cleaned runtime JSON
Structure: consumer-required structure
Visibility: Public only
Preserve comments: No
Preserve directives: No
Templates/schema/code: only public-projected artifacts when needed
Manifest: recommended
```

### IDE source handoff

```text
Use Project Import / Export
or provide the original source archive.
```

Do not use cleaned runtime export as the only source backup because it may omit templates, comments, directives, ignored authoring data, and non-config documentation.

## 10. Migration/release verification

After adapting existing configs:

1. export `Full` cleaned runtime JSON;
2. compare it semantically to the original runtime data;
3. export `Public only` when applicable;
4. search the entire archive for private sentinel names/values;
5. inspect warnings, artifact counts, and manifest digests;
6. test the real runtime consumer with the exported files;
7. if the Project is versioned, use Group **Analyze** to inspect unexpected changes before release;
8. create a `Released` version only after the intended source state has persisted and validation/export checks pass.
