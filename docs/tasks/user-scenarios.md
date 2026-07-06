# Task: User Scenarios (Feature Stories)

## Status

Planned

## Motivation

The project currently has architecture and a manifest schema, but lacks
explicit descriptions of **who uses dotmgr, how, and in what situations**.
User scenarios (Feature Stories) serve as the bridge between architecture and
implementation — they define the business logic that drives CLI design and QA.

See [docs/CONTEXT_AS_CODE.md](../CONTEXT_AS_CODE.md) for the motivation behind
keeping intent in the repository.

## Expected outcome

A single document `docs/USER_SCENARIOS.md` containing 5–8 Feature Stories
written in Gherkin-style (Given/When/Then). Each story describes one
end-to-end user workflow from the perspective of a person managing their
dotfiles.

## Format

Use Gherkin (Given/When/Then) for each scenario:

```gherkin
Feature: <feature name>
  As a <role>
  I want to <goal>
  So that <benefit>

  Scenario: <scenario name>
    Given <precondition>
    When <action>
    Then <expected result>
```

The `Feature:` header is a short title. The optional narrative line (As a / I
want to / So that) adds user context. Each scenario should be independent and
self-contained.

## Scenarios to cover

At minimum, the following flows should be documented. The Author agent may
add more if they identify gaps.

| # | Flow | Key interactions |
|---|------|-----------------|
| 1 | **Fresh system bootstrap** | User installs Ubuntu, clones dotmgr, runs `init`, then `apply` for the first time. Covers `init` + `apply` without `installed.json`. |
| 2 | **Incremental config update** | User adds a new block to `manifest.yaml`, runs `apply <block>`. Covers idempotency and selective apply. |
| 3 | **Package installation** | Block depends on apt/pip packages not present on the system. Covers `PackageProvider.resolve()` before Stow. |
| 4 | **Custom script deployment** | Block with custom scripts and `file_permissions`. Covers chmod after Stow. |
| 5 | **Dry-run preview** | User runs `sync --dry-run` to see what would change before applying. Covers read-only diff. |
| 6 | **Drift detection** | User modifies a symlink target outside dotmgr, then runs `sync`. Covers `installed.json` comparison. |
| 7 | **Block rollback / disable** | User disables a block (`enabled: false`) and re-applies. Covers cleanup of symlinks and packages. |
| 8 | **Multi-host conditions** (Phase 3 placeholder) | User has desktop + laptop, each with host-specific blocks. Covers `condition` field (structure only, no implementation). |

## References

- [docs/ARCHITECTURE.md](../ARCHITECTURE.md) — glossary, terminology, Source of Truth model
- [docs/manifest-schema-v1.md](../manifest-schema-v1.md) — block structure with stow, packages, tasks, file_permissions
- [schemas/manifest-v1.json](../../schemas/manifest-v1.json) — JSON Schema for validation
- [docs/QA_STRATEGY.md](../QA_STRATEGY.md) — testing layers; scenarios will later become integration tests
- [docs/ROADMAP.md](../ROADMAP.md) — Phase 1 / Phase 2 scope
- [docs/CONTRIBUTING.md](../CONTRIBUTING.md) — branch naming, commit conventions, merge workflow

## Acceptance criteria (DoD)

- [ ] `docs/USER_SCENARIOS.md` exists with 5–8 Gherkin-style scenarios
- [ ] Each scenario is independent and self-contained
- [ ] All referenced terms (block, apply, sync, stow, manifest) are hyperlinked to definitions in `docs/ARCHITECTURE.md` or `docs/manifest-schema-v1.md`
- [ ] Scenarios cover at least flows 1–6 from the table above
- [ ] `docs/ROADMAP.md` is updated to mark the checkpoint as completed
- [ ] Commit messages follow Conventional Commits with `Co-Authored-By` trailer
- [ ] Documentation is in English
