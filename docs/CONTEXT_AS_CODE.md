# Context-as-Code

This term is broader than Documentation-as-Code: it includes not only project
documentation but also meta-information for agents — intentions, constraints,
and interaction patterns. Context files are part of the Definition of Done for
any PR, and agents are obligated to read the entry-point files below before any
non-read-only operation.

## Definition

Project context (architecture, decisions, constraints, workflow policies) is
stored in the repository as first-class artifacts — as versioned, human-readable
files. Any participant (human or AI agent) entering the repository reads these
files to reconstruct the full context without external sources.

## Entry-point files (read in this order)

1. `README.md` — what this is, why it exists, how to get started.
2. [`ARCHITECTURE.md`](ARCHITECTURE.md) — system skeleton and Source
   of Truth.
3. [`ROADMAP.md`](ROADMAP.md) — current state and priorities.
4. [`CONTRIBUTING.md`](CONTRIBUTING.md) — development process.
5. [`AGENT_WORKFLOW.md`](AGENT_WORKFLOW.md) — agent collaboration
   policy and review cycle.
6. `docs/CONTEXT_AS_CODE.md` — this file.

## Agent workflow: review -> fix -> re-review

1. The **Author agent** opens a PR with changes.
2. The **Reviewer agent** conducts a structured review with inline comments,
   marked by axes (completeness / security / architecture / clarity).
   Review verdicts:
   - `COMMENT` — recommendation; the PR may be merged.
   - `REQUEST_CHANGES` — blocking remark; do not merge until resolved.
3. The **Author agent** reads the comments, applies fixes, and pushes new
   commits to the same branch (or a derived branch for A/B comparison).
4. The **Reviewer agent** re-reviews the updated diff.
5. The cycle repeats until all blocking remarks are resolved.

## Roles and escalation

- **Author agent** — opens the PR.
- **Reviewer agent** — issues `COMMENT` / `REQUEST_CHANGES`.
- **Human (MITM)** — final authority: accepts, rejects, or requests rework.
  May override any agent decision.
- **Orchestrator (future)** — manages the cycle automatically, but delegates
  final approval to the human.

## Principles

- Context lives in the repository, not in heads or chat histories.
- Context files are part of the Definition of Done for any PR.
- Agents are obligated to read the entry-point files before any non-read-only
  operation.
