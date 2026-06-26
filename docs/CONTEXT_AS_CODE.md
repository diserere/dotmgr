# Context-as-Code

## Definition
Project context (architecture, decisions, constraints, workflows) is stored in the repository as first-class artifacts — as versioned, human-readable files. Any participant in the process (human or AI agent) entering the repository reads these files to reconstruct the full context without external sources.

This term is broader than Documentation-as-Code: it includes not only project documentation but also meta-information for agents — intentions, constraints, and interaction patterns.

## Entry-point files (reading order)
1. `README.md` — What is this, why does it exist, how to run it.
2. `docs/ARCHITECTURE.md` — System skeleton and Source of Truth.
3. `docs/ROADMAP.md` — Current state and priorities.
4. `docs/CONTRIBUTING.md` — Working process.
5. `docs/CONTEXT_AS_CODE.md` — This file, philosophy, and agent workflow.

## Agent Workflow: review → fix → re-review
1. The Author Agent creates a PR with changes.
2. The Reviewer Agent conducts a structured review with inline comments, marked by axes (completeness / security / architecture / clarity).
   Review types:
   - `COMMENT` — recommendation, PR can be merged.
   - `REQUEST_CHANGES` — blocking remark, do not merge until fixed.
3. The Author Agent reads the comments, generates fixes, and pushes new commits to the same branch (or a derivative branch for A/B comparison).
4. The Reviewer Agent conducts a re-review of the updated diff.
5. The cycle repeats until all blocking remarks are resolved.

## Roles and Escalation
- **Author Agent** — creates the PR.
- **Reviewer Agent** — sets COMMENT / REQUEST_CHANGES.
- **MITM Human** — final word: approves PR, rejects, or asks to redo. Can override any agent's decision.
- **Orchestrator (future)** — manages the cycle automatically but delegates final approval to the human.

## Principles
- Context in the repository, not in heads or chat histories.
- Context files are part of the Definition of Done for any PR.
- Agents are obligated to read entry-point files before any non-read-only operation.
