# Roadmap

## Phase 1: MVP (Core Engine)
- [ ] Define manifest.yaml schema (version 1).
- [ ] Implement `dotmgr` CLI scaffolding (Python, `click`, `questionary`).
- [ ] Implement `init` command (setup private repo connection).
- [ ] Implement basic `apply` command (stow functionality for one package).

## Phase 2: Orchestration & State
- [ ] Implement local state tracking (`installed.json`).
- [ ] Implement interactive setup wizard (asking about blocks to install).
- [ ] Implement dependency resolution for apt/pip packages.
- [ ] Implement `--dry-run` / `-n` for `apply` and `sync`.

## Phase 3: Synchronization & Host Management
- [ ] Implement `sync` command (drift detection + reconciliation).
- [ ] Implement `HostResolver` interface with `HierarchicalHostResolver`
  supporting a `hosts/<hostname>/` overlay directory and per-host
  variable injection.

## Future Plans (Research/Community)
- [ ] Support for MacOS (brew, etc.).
- [ ] Support for additional package managers (snap, flatpak).
- [ ] Automated CI/CD checks for manifest validity.
