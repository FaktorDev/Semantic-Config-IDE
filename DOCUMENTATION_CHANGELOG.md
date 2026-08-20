# Documentation Changelog

## 2026-08-21 — Documentation refresh for IDE 0.1.2

Updated the documentation baseline from `ide-frontend` `0.1.1` to the `0.1.2` release scope.

Major changes:

- added a root [CHANGELOG.md](./CHANGELOG.md) with the user-visible `0.1.2` release notes;
- made the product changelog directly discoverable from `README.md`;
- added [PROJECT_EVOLUTION.md](./PROJECT_EVOLUTION.md) covering Project Groups, standalone/group Analyze, semantic config/entity/property history, Activity, and `Draft` / `Released` / `Archived`;
- documented immutable release snapshots and canonical SHA-256 release identity;
- documented the recommended `Released → Create next Draft` workflow;
- updated import/export docs with Project-panel transfer, Project Group metadata separation, deterministic manifests, progress-aware large exports, cancellation, and compression presets;
- documented Markdown/non-config project files as opaque to the semantic config pipeline;
- updated AI handoff/migration guidance to avoid mutating sealed releases;
- refreshed the machine-readable documentation manifest to `0.1.2`;
- kept `DOCUMENTATION_CHANGELOG.md` as documentation-maintenance history, separate from the product `CHANGELOG.md`.

## 2026-08-02 — Implementation-aligned documentation rewrite

Reconciled the documentation with the supplied `ide-frontend` version `0.1.1` source snapshot.

Major changes:

- added `START_HERE_FOR_AI.md` as the primary adaptation contract;
- added a full config migration workflow;
- documented current legacy implicit config discovery and made explicit `@config` the migration standard;
- removed obsolete “Target Stage 7” roadmap wording;
- documented explicit-template-only behavior;
- added exact public/private export semantics and fail-closed security guidance;
- documented current export modes, structures, warnings, and public artifact projection;
- corrected reusable type/enum examples to use valid single-root JSONC declarations;
- added current directive table, canonical spellings, parser aliases, placement rules, and codegen values;
- updated Quick Fix and Global Fix documentation to match implemented action families;
- added complete JSONC examples;
- added migration report and new-chat handoff templates;
- added a machine-readable documentation manifest.
