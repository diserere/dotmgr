# Roadmap

## Phase 1: MVP (Core Engine & Core Abstractions)
- [ ] Define `manifest.yaml` schema (version 1) using **Modules** as atomic units.
- [ ] Implement core extensible interfaces:
  - `BasePackageProvider` with default `AptProvider` implementation.
  - `HostResolver` for condition checking and hostname resolving.
- [ ] Implement `dotmgr` CLI scaffolding (Python 3, `click`, `questionary`).
- [ ] Implement `init` command (sets up private repository connection and local configuration directory).
- [ ] Implement basic `apply` command (orchestrates `GNU Stow` symlinks for a specific Module).
- [ ] Implement `--dry-run` flag support across package, symlink, and task actions.

## Phase 2: Orchestration & State Cache
- [ ] Implement local State Cache (`installed.json`) to track applied modules.
- [ ] Implement interactive setup wizard (prompts user to select which modules to install).
- [ ] Implement task execution and system package dependency resolution.
- [ ] Integrate conflict resolution and backup strategy (renaming pre-existing files to `.bak`).

## Phase 3: Synchronization, Host Management & Secrets
- [ ] Implement `sync` command (drift detection between Manifest Desired State and local Cache/System).
- [ ] Implement Host-specific Module filtering based on `HostResolver` conditions.
- [ ] Implement `.env` secrets injection and decryption tasks for private credentials.

## Future Plans (Research & Community)
- [ ] Support for MacOS via `BrewProvider` implementation.
- [ ] Support for additional package managers (Snap, Flatpak, Pip).
- [ ] Automated CI/CD validation tests for manifest files.
