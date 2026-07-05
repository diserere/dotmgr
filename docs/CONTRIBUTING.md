# Contributing Guidelines

## Documentation & Language

- All documentation, commit messages, and comments **MUST** be in English.
  This ensures consistency for AI agents and global contributors.

## Git Workflow

- **Branching:** trunk-based development with short-lived topic branches:
  - `feat/<feature-name>` — new behavior.
  - `docs/<doc-name>` — documentation changes.
  - `fix/<issue-name>` — bug fixes.
- **Commits:** follow the
  [Conventional Commits](https://www.conventionalcommits.org) specification
  (e.g. `feat(cli): add init command`).
- **Co-authorship:** every commit authored with agent assistance carries a
  `Co-Authored-By` trailer naming the model that assisted and its assigned role,
  e.g. `Co-Authored-By: Claude <noreply@anthropic.com> [Role: Author]`.
  This is required, not optional, so the provenance and responsibility of
  agent-assisted work is always auditable.
- **Merging:** use fast-forward (`--ff-only`) locally to maintain a clean
  linear history. Avoid merge commits.

## Versioning

- Use [Semantic Versioning](https://semver.org) (SemVer) for the project.

## Review Provenance
All PR review comments must be marked with a prefix (e.g., [agent-review], [human-review]).

## Agent Workflow

The agent collaboration policy (human-in-the-loop, review cycle, roles, and the
Definition of Done for agent-authored PRs) lives in
[`docs/AGENT_WORKFLOW.md`](docs/AGENT_WORKFLOW.md). In short: the agent
autonomously executes read-only operations; any non-read-only operation
requires explicit human approval via the tool-execution prompt. The human
maintains ultimate authority and may override any agent decision.

## Agent Roles

To ensure systematic development, assign roles to agents at the start of a session:

- **Architect:** Responsible for `ARCHITECTURE.md`, `ADRs`, and high-level design consistency.
- **Author:** Implements code/docs and creates PRs.
- **Reviewer:** Conducts structured reviews (Security, Architecture, Clarity) using `COMMENT`/`REQUEST_CHANGES`.
- **QA/Tester:** Develops and executes integration tests (e.g., in Docker), validates `dry-run` results.
- **MITM (Human):** Final gatekeeper. Approves/Rejects PRs and resolves conflicts between agent roles.
