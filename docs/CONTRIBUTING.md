# Contributing Guidelines

## Documentation & Language
- All documentation, commit messages, and comments MUST be in English.
- This ensures consistency for AI agents and global contributors.

## Git Workflow
- **Branching:** Trunk-based development. Use short-lived branches:
  - `feat/<feature-name>`
  - `docs/<doc-name>`
  - `fix/<issue-name>`
- **Commits:** Conventional Commits specification (e.g., `feat: add cli boilerplate`).
- **Merging:** Use fast-forward (`--ff-only`) locally to maintain a clean linear history. Avoid merge commits.

## Versioning
- Use Semantic Versioning (SemVer) for the project.

## Agent Workflow

The policy for human↔agent collaboration (review cycle, roles, escalation,
Definition of Done for agent-authored PRs) lives in
[`docs/AGENT_WORKFLOW.md`](docs/AGENT_WORKFLOW.md).
