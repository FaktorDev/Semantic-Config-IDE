# Global Fixes

Global Fixes are preview-first operations for repeated, config-wide, project-wide, or multi-file changes.

They build a plan before application. The plan can include text edits and file operations.

## Implemented actions

### Schema and constraint cleanup

| Action | Scope | Behavior |
|---|---|---|
| Remove all additional properties | File | Removes properties reported as unknown/additional by schema validation. Destructive. |
| Add additional property to template as optional | File | Adds repeated unknown properties to the explicit template with `//@optional` instead of removing data. |
| Add all missing properties with defaults | File | Inserts required properties using schema default, template value, or type fallback. |
| Add all missing properties with null | File | Inserts missing required properties with `null`; consumer compatibility must be reviewed. |
| Convert numeric strings to numbers | File | Converts values such as `"12"` when diagnostics establish a numeric target. |
| Clamp all out-of-range values | File | Moves numeric values to the nearest valid schema/directive bound. |

### Files and templates

| Action | Scope | Behavior |
|---|---|---|
| Extract template to separate file | File | Creates a sibling `*.template.jsonc` config part and removes the explicit template element from the original source file. |
| Split file by property | File | Splits runtime entities into sibling files by a selected string or string-array property; the template stays in the source file. |

For string arrays, split-file logic uses the first array value as the grouping target. Review whether this matches domain semantics.

### Rename and reference-safe refactoring

| Action | Scope | Behavior |
|---|---|---|
| Rename config key | Project | Renames `@config` and updates foreign-key target declarations project-wide. |
| Rename property in config | Logical config | Renames one property in the template and runtime entities across all physical config parts. |
| Rename primary key property | Logical config | Renames the PK property name; PK values remain unchanged. |
| Rename primary key value and update FK references | Project | Renames one PK value and cascades only foreign keys that reference that config. |

Reference-aware rename does not replace random matching strings outside the semantic reference graph.

### Broken foreign keys

| Action | Behavior |
|---|---|
| Set broken FK values to null | Replaces unresolved FK values with null where applicable. |
| Remove optional broken FK properties | Removes broken reference properties when the schema allows omission. |
| Replace broken FK with closest existing PK | Uses a closest-match resolver and must be manually reviewed. |

The closest-match action is heuristic. Never apply it without inspecting every replacement.

### Diagnostic preview

A no-op issue-summary action exists to exercise the preview workflow without changing files.

## Preview contents

A Global Fix preview can show:

- action title and scope;
- affected files;
- created/deleted files;
- per-file text changes;
- warnings and conflicts;
- destructive status;
- estimated operation cost.

All implemented default actions require preview.

## Safety rules

1. Read the complete preview.
2. Review created and deleted paths.
3. Check whether edits touch template, runtime data, or both.
4. Verify PK/FK cascades by semantic target, not only text count.
5. Do not accept guessed fuzzy FK replacements without domain confirmation.
6. Re-run validation after apply.
7. Rebuild/export and compare runtime data after structural operations.
8. Keep a project backup before destructive operations.

## Multi-file atomicity caution

A preview is not the same as a database transaction. Treat large file operations as changes that require backup and post-apply verification.

Recommended sequence:

```text
Export Project backup
→ build Global Fix preview
→ inspect every file operation
→ apply
→ rebuild semantic context
→ resolve remaining diagnostics
→ export full runtime files
→ compare with intended result
```

## Choosing between competing fixes

### Unknown property

- property is invalid/stale → remove it;
- property is legitimate and part of the model → add it to template as optional/required after review.

### Missing property

- template/default has a valid domain value → add with defaults;
- null is valid and expected by consumer → add with null;
- property should actually be optional → change the template instead of bulk-filling runtime data.

### Broken FK

- value is obsolete and optional → remove property;
- null is a defined state → set null;
- obvious typo with confirmed target → replace;
- target entity is missing → restore/add target rather than mutating the reference.

## Difference from import/export

Global Fix modifies the active IDE project.

Import/export moves project or runtime artifacts into/out of the IDE. Export does not replace the need for semantic refactoring, and Global Fix is not a backup mechanism.
