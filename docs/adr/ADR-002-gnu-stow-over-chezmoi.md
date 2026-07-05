# ADR-002: GNU Stow over `chezmoi` / `yadm` / `Ansible`

- **Date:** 2026-01-01
- **Status:** accepted

## Context

Symlinking is the core mechanism for applying config files; writing
a custom, bug-free symlink orchestrator from scratch is error-prone.

## Decision

Orchestrate symlinks with **GNU Stow**.

## Consequences

### Pros

- A single-purpose symlink tool with no daemon, no imperative
  playbooks, and a trivial mental model.
- `chezmoi` and `yadm` couple encryption and templating to the manager,
  which adds surface area and makes the "public engine / private config"
  split harder to reason about.
- Ansible is a full orchestration framework — overkill for per-user
  dotfiles.
- Stow is already packaged on every target distro.

### Cons

- Adds `stow` as a required package in the bootstrap phase (easily
  installed via `apt install stow`).
