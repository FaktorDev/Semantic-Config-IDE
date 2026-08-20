# Project Evolution

Project Evolution is the IDE subsystem for organizing configuration snapshots as versions, comparing them semantically, tracking local work, and sealing reproducible releases.

It complements the config model described in [CONFIG_MODEL.md](./CONFIG_MODEL.md). A Project remains an independent editable snapshot; a Project Group organizes related Projects as a version family.

## Project Groups and versions

A Project Group contains ordered Project versions.

Typical example:

```text
CoS
├── 0.3.4
├── 0.3.5
└── 0.3.6
```

Important rules:

- each Project remains independent;
- a Project belongs to at most one group in the current model;
- version labels are explicit metadata, not inferred from filenames once assigned;
- group operations do not change the contents of Project export bundles;
- Project transfer remains usable outside a Project Group.

## Analyze

### Standalone Project analysis

Open a Project and choose **Analyze** to inspect the current Project without requiring a group.

Available views include:

- **Overview** — files, configs, entities, lines, PK/FK and other aggregate metrics;
- **Configs** — logical config inventory and semantic metrics;
- **Quality** — error/warning/diagnostic summaries;
- **Activity** — local activity calendar and day drilldown.

Expensive semantic analytics are cached. If the Project changes after the cached snapshot, the UI can mark analysis as outdated until refreshed.

### Project Group analysis

Use **Analyze** on a Project Group to compare versions.

The comparison selector uses:

```text
FROM <version>
     ⇄
TO   <version>
```

The selected pair drives file and semantic comparison.

Group analysis views include:

- **Changes** — added, removed, modified, and renamed files with source diff;
- **Overview** — aggregate comparison metrics;
- **Growth** — change over versions;
- **Configs** — semantic config/entity/property analysis;
- **Quality** — diagnostic trends;
- **Activity** — aggregated activity across current group members.

## Semantic change levels

### Config-level

The IDE can classify logical configs as:

- added;
- removed;
- changed;
- unchanged.

Semantic digests use canonical object serialization so object-property ordering alone does not create a false semantic change.

### Entity-level

When a config has a reliable `//@primary_key`, entities can be matched by their actual domain identity.

The IDE reports:

- added entities;
- removed entities;
- changed entities.

Identity is fail-closed. The IDE does not guess entity identity from array position when the PK is missing, duplicated, invalid, or incompatible between the compared versions.

### Property-level

A changed primary-key entity can be inspected on demand:

```text
item_sword

Damage        10 → 13
Weight         5 → 4.5
Durability   100 → 120
Visual.Icon  old → new
```

Nested object paths are supported.

Arrays are compared atomically unless the IDE has an explicit safe identity contract for their elements. This avoids misleading index-based or fuzzy diffs.

## Activity history

Activity is a local, aggregated history designed for project insight rather than keystroke recording.

Tracked categories include:

- edit sessions;
- edited files;
- files created/deleted/renamed/moved;
- imports;
- Project/version creation;
- Project Group membership changes;
- release/archive/restore operations when lifecycle support is active.

Text editing is aggregated into sessions rather than persisted per keypress.

The Activity view provides a 52-week heatmap and per-day drilldown.

Important limitations:

- tracking starts when Activity support is initialized for that Project;
- the IDE does not fabricate older activity;
- history is local to the current browser/IndexedDB unless exported or synchronized by a future external system;
- Group Activity is aggregated from its current member Projects instead of being duplicated as a separate event history.

## Version lifecycle

Project Group versions can have one of three states:

```text
Draft
Released
Archived
```

### Draft

A Draft is editable and is the normal state for active work.

Use a Draft for config changes, migration, validation, and preparation of a release.

### Released

Releasing a Draft creates an immutable release snapshot and a canonical SHA-256 release digest.

A Released version is sealed:

- JSON editors become read-only;
- persistence guards reject content mutation;
- rename/move/delete/version-changing operations that would mutate historical state are blocked;
- history and analysis use the frozen release snapshot rather than mutable live rows.

Before creating the release snapshot, pending editor persistence is flushed so unsaved debounced text cannot be silently excluded from the release.

### Archived

An Archived release remains immutable and available for historical analysis. Archiving changes lifecycle state but does not change the release snapshot or digest.

An Archived release can be restored to `Released` without changing its content identity.

## Create next Draft

Do not edit a Released version in place.

Preferred workflow:

```text
0.3.5 Released
      ↓
Create next Draft
      ↓
0.3.6 Draft
```

The next Draft is cloned from the frozen release snapshot, not from potentially diverged live IndexedDB rows.

This makes the release history reproducible even if external tools or browser DevTools modify live storage outside normal IDE guards.

## Release identity

The release SHA-256 represents canonical Project content rather than volatile UI metadata.

Release identity includes configuration source and semantic project structure required to reproduce the version. Volatile metadata such as timestamps and workspace layout does not define content identity.

The IDE can detect when live Project state diverges from the frozen release snapshot. Historical Analyze/export/clone workflows continue to rely on the immutable release snapshot.

## Project transfer and releases

Project-file import/export remains a standalone Project transfer format.

Project Group metadata such as:

- group ID;
- version order;
- lifecycle state;
- release digest;

is not required to make a Project bundle portable.

For sealed releases, historical operations should use the immutable release source where supported.

See [IMPORT_EXPORT.md](./IMPORT_EXPORT.md) for artifact/export rules.

## Recommended workflow

1. Work in a `Draft` Project.
2. Run validation and standalone **Analyze**.
3. Export Full/Public Ready Files as required and test real consumers.
4. Add the Project to the appropriate Project Group/version family.
5. Compare it with the previous version using Group **Analyze**.
6. Resolve unexpected file/config/entity/property changes.
7. Release the Draft.
8. Record or publish the release SHA-256 where reproducibility matters.
9. Create the next Draft for continued development.
10. Archive older releases only when they are no longer active, not as a substitute for deletion.
