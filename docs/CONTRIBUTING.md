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
- **Agent Workflow:** The agent autonomously executes changes (e.g., git operations, file writes) after reaching a strategic consensus with the human. The agent is responsible for creating descriptive, high-quality commit messages and maintaining project documentation. The human maintains ultimate control by approving all non-read-only actions via the tool execution prompt.
