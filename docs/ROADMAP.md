# Roadmap

This roadmap tracks the path from documentation-only to a working `dotmgr`
engine. Each phase is a self-contained milestone; phases build on the previous
one. Terminology follows the glossary in
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — "block" is the canonical term
for an atomic configuration unit.

## Phase 1: MVP (Core Engine)

- [ ] Define `manifest.yaml` schema (version 1).
- [ ] Implement `dotmgr` CLI scaffolding (Python, `click`, `questionary`).
- [ ] Implement `init` command (clone and record the private repo connection).
- [ ] Implement basic `apply` command (Stow functionality for one block).

## Phase 2: Orchestration & State

- [ ] Implement local state tracking (`installed.json`).
- [ ] Implement interactive setup wizard (ask which blocks to install).
- [ ] Implement dependency resolution for apt/pip packages.
- [ ] Implement `--dry-run` / `-n` for `apply` and `sync`.

## Phase 3: Synchronization & Host Management

- [ ] Implement `sync` command (drift detection + reconciliation).
- [ ] Implement `HostResolver` interface with `HierarchicalHostResolver`
  supporting a `hosts/<hostname>/` overlay directory and per-host variable
  injection.

## Future Plans (Research / Community)

- [ ] Support for MacOS (brew, etc.).
- [ ] Support for additional package managers (snap, flatpak).
- [ ] Automated CI/CD checks for manifest validity.
