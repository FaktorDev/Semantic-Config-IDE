# Migration Report Template

Use this structure when adapting a config project to Semantic Config IDE.

## 1. Scope

- Source archive/root:
- Target root:
- Migration date:
- IDE documentation version/snapshot:
- Runtime compatibility requirement:
- Client/public export requirement:
- Generated languages:

## 2. Summary

Describe:

- number of source files inspected;
- number of files changed/created/removed;
- number of logical configs;
- number of templates;
- number of reusable types/enums;
- number of PK/FK relationships;
- whether runtime semantics changed.

## 3. File operations

| Operation | Path | Reason |
|---|---|---|
| Modified |  |  |
| Created |  |  |
| Removed |  |  |
| Unchanged intentionally |  |  |

## 4. Config mapping

| Source collection/path | IDE config name | Physical parts | Template file | Notes |
|---|---|---|---|---|
|  |  |  |  |  |

## 5. Template decisions

For each logical config:

### `<ConfigName>`

- Template source/evidence:
- Required properties:
- Optional properties:
- Nullable properties:
- Type conflicts and resolution:
- Union/polymorphism decisions:
- Generated name/codegen directives:

## 6. Primary keys

| Config | Property | Type | Uniqueness result | Stability evidence | Public override |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

List duplicate or missing keys separately.

## 7. Foreign keys

| Source config | Property | Target config | Scalar/array | Broken references | Resolution |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

## 8. Reusable definitions

### Types

| Name | Scope | Used by | Reason extracted |
|---|---|---|---|
|  | global/local |  |  |

### Enums

| Name | Scope | Values/evidence | Used by |
|---|---|---|---|
|  | global/local |  |  |

## 9. Constraints added

| Config/type | Property | Directive | Evidence | Risk |
|---|---|---|---|---|
|  |  |  |  |  |

Explicitly identify constraints inferred only from user approval versus code/schema evidence.

## 10. Public/private export

| Config/type | Property/subtree | Effective visibility | Reason |
|---|---|---|---|
|  |  | public/private/implicit PK/FK |  |

- Private sentinel used:
- ZIP-wide sentinel search result:
- Public schemas/code inspected:
- Hidden PK/FK decisions:
- Configs skipped due to missing IR:

## 11. Compatibility comparison

- Original runtime entity counts:
- Full cleaned export entity counts:
- Property/value differences:
- Order differences:
- Filename/path differences:
- Consumer changes required:

List every intentional semantic change.

## 12. Validation performed

Mark each item:

- [ ] All JSON/JSONC files parse.
- [ ] Every intended config has explicit `@config`.
- [ ] Every logical config has exactly one explicit `@template`.
- [ ] Template-only directive placement passes.
- [ ] Reusable type/enum references resolve.
- [ ] Union members resolve.
- [ ] Primary keys are present and unique.
- [ ] Foreign keys resolve or are reported.
- [ ] Required/optional/nullability checks pass.
- [ ] Enum/regex/clamp/array-length checks pass.
- [ ] Full cleaned export compared with source runtime data.
- [ ] Public export searched for private names and sentinel values.
- [ ] Generated schema/code reviewed.
- [ ] Archive structure verified.

Include commands/tooling used and results.

## 13. Unresolved questions

| Question | Why unresolved | Safe current behavior | Required owner decision |
|---|---|---|---|
|  |  |  |  |

## 14. Known risks

Include:

- ambiguous data shapes;
- heuristic decisions;
- consumer assumptions;
- union ambiguity;
- legacy implicit configs;
- destructive normalization;
- public contract expansion risks;
- missing tests.

## 15. Recommended next steps

Prioritize concrete steps such as:

1. run IDE validation on the full project;
2. resolve remaining PK/FK diagnostics;
3. test full export in the real runtime;
4. add public sentinel regression tests;
5. replace legacy aliases with canonical directives;
6. add CI verification for configs and generated artifacts.
