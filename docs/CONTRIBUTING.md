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
- **Merging (local merge workflow):**
  1. Open a Pull Request on GitHub for review and tracking.
  2. After approval, merge locally into `main` — this preserves the full
     commit history from the topic branch:
     ```
     git checkout main
     git merge <branch-name>
     git push
     ```
  3. GitHub automatically detects the merge and closes the PR.
  4. Delete the topic branch: `git branch -d <branch-name>`.
  - The merge strategy is **default `git merge`** (no `--ff-only`, no
    `--squash`). This produces a fast-forward when `main` has not diverged,
    or a merge commit when it has. The goal is to **preserve history**, not
    to squash it into a single commit.

## Versioning

- Use [Semantic Versioning](https://semver.org) (SemVer) for the project.

## Agent Workflow

The agent collaboration policy (human-in-the-loop, review cycle, roles, and the
Definition of Done for agent-authored PRs) lives in
[`docs/AGENT_WORKFLOW.md`](AGENT_WORKFLOW.md).
