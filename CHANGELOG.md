# Factor Config IDE 0.1.2

## Project Evolution, semantic history, scalable exports and major IDE reliability improvements

This release significantly expands Factor Config IDE beyond editing individual configuration projects.

Version `0.1.2` introduces project version families, semantic change analysis, activity history, immutable releases, public/private export contracts, major Context Worker scalability improvements, and a broad set of editor, code-generation and import/export UX upgrades.

## Highlights

### Project Groups & Version History

Projects can now be organized into version families using **Project Groups**.

The IDE can:

- organize independent Projects as ordered versions;
- compare `FROM` and `TO` versions;
- detect added, removed, modified and renamed files;
- visualize project growth between versions;
- analyze historical projects without activating them in the editor;
- drag Projects between groups and reorder versions;
- create new versions from existing Projects.

Project Group analysis is available through the new **Analyze** workflow.

---

### Semantic Change Analysis

Version comparison now goes beyond file diffs.

The IDE can analyze changes at several semantic levels:

#### Config-level changes

Detects:

- added configs;
- removed configs;
- changed configs;
- unchanged configs.

Formatting-only object property reordering does not count as a semantic change.

#### Entity-level changes

For configs with a reliable primary key, the IDE detects:

- added entities;
- removed entities;
- changed entities.

Entity identity is resolved through the real `//@primary_key` contract rather than array positions or heuristics.

#### Property-level changes

Changed entities can be inspected down to individual properties:

```text
item_sword

Damage        10 → 13
Weight         5 → 4.5
Durability   100 → 120
Visual.Icon  old → new
```

Nested property paths are supported.

Array values are intentionally treated atomically when no explicit element identity exists, avoiding misleading index-based diffs.

---

### Standalone Project Analytics

The **Analyze** panel now works for individual Projects even when they are not part of a Project Group.

Available views include:

- **Overview**
- **Configs**
- **Quality**
- **Activity**

Analytics includes file, config, entity, line, PK/FK and validation metrics.

Expensive analysis is cached and can be refreshed explicitly instead of running after every keystroke.

---

### Project Activity History

Factor Config IDE now records useful local project activity without storing a keypress event log.

Activity includes:

- edit sessions;
- edited files;
- files created;
- files deleted;
- files renamed;
- files moved;
- imports;
- project/version creation;
- Project Group membership changes.

Editing activity is aggregated into sessions rather than writing an event for every text change.

A 52-week activity heatmap and per-day drilldown are available for both individual Projects and Project Groups.

The IDE never fabricates activity from before tracking was enabled.

---

### Draft / Released / Archived Versions

Project versions now have an explicit lifecycle:

```text
Draft
  ↓
Released
  ↓
Archived
```

Released versions are sealed and receive an immutable Project snapshot with a canonical **SHA-256 release digest**.

A release snapshot includes configuration files, config settings and source-tree structure while excluding volatile IDE metadata.

Released and Archived versions:

- are read-only;
- reject content mutations at persistence boundaries;
- remain available for historical analysis;
- preserve deterministic historical comparisons;
- can be used to create the next editable Draft.

Example workflow:

```text
0.3.5 Released
      ↓
Create next Draft
      ↓
0.3.6 Draft
```

Historical analysis and cloning use the frozen release snapshot rather than mutable live IndexedDB state.

---

## Public / Private Export

A new visibility model allows generating client-safe configuration packages.

New directives:

```jsonc
//@public
//@private
```

Rules include:

- `//@private` removes a property and its subtree from public exports;
- `//@public` exposes a property or subtree;
- `//@primary_key` and `//@foreign_key` are implicitly public;
- unmarked properties remain private in `publicOnly` mode.

Public filtering applies consistently to:

- runtime JSON;
- templates;
- JSON Schema;
- generated C#;
- generated TypeScript.

Private logical configs can be removed completely from public exports.

Public export is fail-closed when the IDE cannot safely determine the projected schema or runtime shape.

---

### Deterministic Export Manifest

Ready Files exports now support a deterministic manifest containing:

- export visibility mode;
- source SHA-256;
- artifact-set SHA-256;
- content revision;
- SHA-256 for every generated artifact.

This makes exported configuration packages easier to validate and integrate into build pipelines.

---

## Large Project Export Improvements

Large Ready Files ZIP exports received substantial reliability and performance improvements.

The export worker now uses progress-aware stall detection instead of a fixed total runtime timeout.

Export progress is reported across stages such as:

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

Other improvements include:

- hard cancellation by terminating the worker;
- parse-once prepared sources;
- indexed directive lookups;
- reuse of structured runtime data;
- cached public IR projections;
- reduced ZIP memory copying;
- configurable ZIP compression presets:
  - Fast
  - Balanced
  - Smaller ZIP

Large exports are no longer considered failed simply because they take longer than the previous fixed timeout.

---

## Context Worker Reliability & Performance

The semantic Context pipeline received major scalability work for large projects.

Improvements include:

- hard worker restart when switching Projects;
- safe cancellation of stale context builds;
- worker-generation ownership for async results;
- progress reporting through expensive Context stages;
- stall watchdog recovery;
- compact semantic signatures;
- JSON/JSONC-only semantic processing;
- opaque handling for non-config files;
- lazy handling of very large string payloads;
- incremental context transport;
- context warm cache support;
- reduced redundant parsing and cloning;
- lighter Context snapshots while keeping authoritative computed data inside workers.

These changes specifically improve behavior on projects containing hundreds of config files and multi-megabyte generated/runtime payloads.

---

## Reusable Configuration Types

Reusable configuration definitions received several improvements.

The IDE now provides better support for:

```text
//@global_type
//@local_type
//@global_enum
//@local_enum
//@ref_type
//@ref_enum
//@union
```

Enhancements include:

- object-based reusable templates;
- reusable type references on object arrays;
- reusable enum references;
- improved completion suggestions;
- better directive documentation and examples;
- clearer separation between reusable definitions and logical configs.

---

## Code Generation

Generated code workflows were improved for both C# and TypeScript.

Changes include:

- improved generated type naming;
- better shared type resolution;
- structural type reuse;
- improved Generated Code window UX;
- responsive code-generation controls;
- stronger schema/code generation integration with export visibility.

---

## Import / Export UX

Import and export workflows received a substantial UI refresh.

Improvements include:

- improved Ready Files workflow;
- modernized export preview and options;
- direct Project import/export from the Project panel;
- project bundle support;
- clearer export progress;
- better cancellation behavior;
- improved error handling;
- Project-file transfer remains independent from Project Group metadata.

---

## Editor & Source Tree

Editor workflow improvements include:

- Markdown file support;
- configurable tab size;
- copying selected files as prompt-ready clipboard content;
- Source Tree dialogs instead of browser `prompt` / `confirm`;
- simplified Source Tree toolbar;
- better responsive window header controls;
- preservation of editor windows when files are moved;
- improved file icons and source-tree behavior;
- improved JSON indentation persistence.

---

## Monaco, Diagnostics & Quick Fixes

A large number of correctness improvements were made across the semantic editor.

Highlights include:

- improved directive hints and documentation;
- stabilized entity error marker ranges;
- improved PK/FK diagnostics;
- better enum/type scanning;
- corrected template fallback behavior;
- improved reusable-reference completions;
- improved navigation and completion indexes;
- `Remove all additional properties` Quick Fix availability;
- additional Quick Fix correctness improvements.

---

## Sorting Fixes

Fixed `//@sort_elements` behavior where the first array element could previously be skipped.

Sorting and directive-driven formatting received additional correctness improvements.

---

## Runtime & Infrastructure

Additional platform improvements include:

- request logging middleware;
- improved local diagnostics;
- Webpack-based Next.js dev and production builds for more reliable Worker bundling;
- proper standalone production startup;
- improved IDE snapshot tooling for reproducible debugging and handoff.

---

## Testing & Reliability

The test surface has grown substantially since `0.1.1`.

Compared with the `0.1.1` source archive:

- test files increased from roughly **96 to 145**;
- new regression coverage was added for:
  - Project Groups and versioning;
  - historical project diffs;
  - analytics;
  - entity/property semantic changes;
  - activity tracking;
  - release snapshots;
  - public export;
  - worker reliability;
  - context scalability;
  - large ZIP export behavior.

---

## Bug Fixes

This release also includes numerous fixes across existing functionality:

- export correctness and directive-aware exports;
- JSON indentation and persistence;
- issue marker positioning;
- PK/FK validation;
- enum and type scanning;
- Source Tree behavior;
- editor-window persistence during file moves;
- responsive header layouts;
- logging;
- Quick Fix availability;
- sorting;
- import/export stability;
- worker lifecycle and stale-result handling.

---

## Upgrade Notes

Projects created with earlier versions remain usable.

Project Group members that predate lifecycle support are treated as **Draft** versions.

Activity history starts only after activity tracking is initialized; previous activity is intentionally not reconstructed.

Released versions use immutable local snapshots, so they can consume additional IndexedDB storage compared with ordinary Draft projects.

---

## Full comparison

[Compare ide-frontend-v0.1.1...ide-frontend-v0.1.2](https://github.com/FaktorDev/config-ide/compare/ide-frontend-v0.1.1...ide-frontend-v0.1.2)
