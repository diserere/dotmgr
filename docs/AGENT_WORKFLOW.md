# Agent Workflow

This document defines the policy for how AI agents collaborate with humans on
this repository. It is **not** a general contribution guide — for branching,
commits, and language rules see [`docs/CONTRIBUTING.md`](docs/CONTRIBUTING.md).

## Human-in-the-loop

The agent autonomously executes **read-only** operations (searching, reading,
running read-only shell commands) without approval. Any **non-read-only**
operation (writing files, running mutating commands, pushing to a remote)
requires explicit human approval via the tool-execution prompt.

The human maintains ultimate authority and may override any agent decision.

## Agent review cycle

1. **Author agent** opens a PR with changes.
2. **Reviewer agent** performs a structured review with inline comments,
   tagged by axis:
   - `[1. Completeness]` — requirements coverage and consistency.
   - `[2. Security]` — Source of Truth, secrets, injection, trust boundaries.
   - `[3. Architecture / scaling]` — extensibility, failure model, drift.
   - `[4. Clarity]` — developer/agent readiness of documentation.
   Review verdicts:
   - `COMMENT` — recommendation; the PR may be merged.
   - `REQUEST_CHANGES` — blocking issue; do not merge until resolved.
3. **Author agent** reads comments, applies fixes, and pushes new commits to
   the same branch (or a derived branch for A/B comparison).
4. **Reviewer agent** re-reviews the updated diff.
5. The cycle repeats until all blocking items are resolved.

## Roles and escalation

- **Author agent** — opens the PR.
- **Reviewer agent** — issues `COMMENT` / `REQUEST_CHANGES`.
- **Human (MITM)** — final authority: accepts, rejects, or requests rework.
  May override any agent decision.
- **Orchestrator (future)** — manages the cycle automatically, but delegates
  final approval to the human.

## Definition of Done for agent-authored PRs

- All entry-point files are updated if the change affects project context:
  - `README.md`
  - `docs/ARCHITECTURE.md`
  - `docs/ROADMAP.md`
  - `docs/CONTRIBUTING.md`
  - `docs/AGENT_WORKFLOW.md`
  - `docs/CONTEXT_AS_CODE.md`
- Commit messages follow Conventional Commits and carry a
  `Co-Authored-By` trailer.
- Documentation is in English.
- A dry-run of any new `apply`/`sync` behavior is included in the change
  description.

## Review Provenance
All PR review comments (from AI or Human) MUST be marked with a prefix:
- `[agent-review]`
- `[human-review]`

