# Documentation Update Report

## Scope

This refresh updates the documentation repository to the `ide-frontend` `0.1.2` release scope and makes the product changelog visible as part of the primary documentation navigation.

The repository now documents both:

- the semantic-config authoring contract; and
- the user-visible Project Evolution workflows introduced across the current IDE.

## Main corrections and additions

- updated the documented application version from `0.1.1` to `0.1.2`;
- added `CHANGELOG.md` with the current product release notes and linked it prominently from `README.md`;
- added `PROJECT_EVOLUTION.md` for Project Groups, version comparison, semantic history, Activity, and release lifecycle;
- documented standalone Project Analyze and Project Group Analyze;
- documented config-level, PK entity-level, and on-demand property-level semantic change analysis;
- documented local Activity heatmaps and tracking-start limitations;
- documented `Draft`, `Released`, and `Archived` semantics plus immutable release snapshots/SHA-256 identity;
- updated Import/Export with direct Project transfer, deterministic Ready Files manifests, progress-aware exports, cancellation, and ZIP compression presets;
- documented Markdown/non-config Project files as non-semantic content;
- updated AI migration/handoff guidance so sealed releases are not modified in place;
- updated the machine-readable manifest and reading order.

## Existing config-contract corrections retained

The earlier `0.1.1` documentation rewrite remains authoritative for:

- explicit `@config` as the migration standard while legacy implicit discovery exists;
- explicit-template-only semantic processing;
- reusable type/enum declaration patterns;
- canonical directive names and compatibility aliases;
- `@public` / `@private` fail-closed export semantics;
- Quick Fix and Global Fix behavior;
- migration/report templates and JSONC examples.

## Validation performed

- all documentation-local Markdown links resolve;
- all Markdown code fences are balanced;
- `DOCUMENTATION_MANIFEST.json` parses as JSON;
- all example `.jsonc` files still parse after JSONC comment removal;
- `CHANGELOG.md` is linked from the primary README;
- `PROJECT_EVOLUTION.md` is linked from README, Import/Export, START_HERE, and the AI handoff prompt;
- documentation version markers no longer claim the old `0.1.1` baseline as current;
- no unfinished draft markers remain.

## Known documentation risks

1. Exact UI labels can evolve faster than semantic contracts. When UI wording changes, keep the workflow description accurate even if a button label changes.
2. Activity is local browser history; documentation must not imply cross-device synchronization.
3. Release snapshots consume local IndexedDB storage; exact storage size depends on Project size and number of releases.
4. Property-level semantic diff intentionally treats arrays atomically when safe element identity is unavailable.
5. The documentation repository does not replace real runtime integration tests with Full/Public Ready Files.
6. Future IDE releases should update both `CHANGELOG.md` and `DOCUMENTATION_MANIFEST.json`.

## Recommended maintenance

- update `CHANGELOG.md` for every public IDE release;
- update the manifest version/date whenever release behavior changes;
- add a documentation regression example for each new directive;
- keep `START_HERE_FOR_AI.md` concise enough to remain the primary AI entry point;
- validate examples in CI using the same JSONC parser as the IDE;
- periodically compare `DIRECTIVES.md` with the source directive catalog/parser;
- periodically compare `GLOBAL_FIXES.md` and `QUICK_FIXES.md` with registered actions;
- keep `PROJECT_EVOLUTION.md` synchronized with lifecycle and Analyze behavior.
