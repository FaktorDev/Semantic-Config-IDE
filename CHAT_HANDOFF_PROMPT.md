# Prompt for a New AI Chat

Upload this documentation archive and the project config archive, then send the following prompt.

```text
Analyze the uploaded Semantic Config IDE documentation first. Treat START_HERE_FOR_AI.md as the primary workflow contract, DIRECTIVES.md as the canonical directive reference, PUBLIC_EXPORT.md as the client/server visibility contract, and PROJECT_EVOLUTION.md as the Project Group/version/release contract when version history is involved.

Then analyze the uploaded project configs and adapt them to Semantic Config IDE.

Requirements:
- preserve runtime semantics, property names, identifier values, paths, and value types unless a change is explicitly justified;
- add explicit //@config to every intended logical config even if legacy implicit discovery would work;
- create exactly one explicit //@template per logical config;
- do not convert the first real runtime entity into a template;
- add PK/FK only when supported by data, code, schemas, or domain evidence;
- use reusable types/enums only for stable repeated concepts;
- add constraints conservatively and report the evidence for each one;
- use canonical directive spellings;
- when client export is needed, implement //@public and //@private using fail-closed rules and verify JSON, templates, schemas, and generated code;
- do not silently guess unresolved domain decisions;
- preserve comments where possible;
- produce files at their real target paths in a ready replacement archive;
- include MIGRATION_REPORT.md based on MIGRATION_REPORT_TEMPLATE.md;
- validate JSONC, templates, PK uniqueness, FK resolution, full export compatibility, and public sentinel absence;
- if the target Project is Released/Archived, do not mutate it in place; work from a new Draft;
- finish with an analysis of completed work, possible errors, unresolved questions, and next steps.

Before modifying files, build a source inventory and config-mapping table. When information is genuinely missing, make the safest compatibility-preserving choice and list the unresolved issue rather than inventing semantics.
```

## Optional project-specific additions

Append details such as:

```text
Runtime language: C#/.NET
Generated target: C# records
Full/server export path: ...
Public/client export path: ...
Properties that must remain private: ...
Existing primary keys: ...
Files/configs that must not be changed: ...
Archive root expected for replacement: ...
```

## Expected AI deliverables

The AI should return:

1. a ready archive;
2. `MIGRATION_REPORT.md` inside the archive;
3. a concise completion summary;
4. validation results;
5. unresolved questions and risks;
6. recommended next steps.
