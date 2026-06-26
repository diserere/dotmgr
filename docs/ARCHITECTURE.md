# Architecture: Dotfile Manager (Context-as-Code)

## Overview
This project implements a declarative, idempotent system for managing user dotfiles, packages, and custom scripts across Linux environments. It uses a "Public Engine / Private Configuration" model to balance usability with security.

## Core Philosophy: Context-as-Code
The project maintains its architecture, design decisions, and roadmap as first-class citizens within the repository. AI agents interacting with this repo are expected to read these files to understand the project's constraints and intent.

## Source of Truth
1. **Desired State (Manifest):** `manifest.yaml` in the private configuration repository is the **absolute source of truth**. It defines what should be installed and configured.
2. **Observed/Local State (`installed.json`):** This is a performance cache, not a source of truth. The CLI uses it to detect drifts and optimize performance, but the manifest always overrides it during `sync` operations.

## Glossary

| Term | Meaning |
|------|---------|
| **Manifest** | `manifest.yaml` in the private repository — the **absolute source of truth** for desired state. |
| **Block** | An atomic, named configuration unit inside a manifest. A block maps to a Stow package (directory). Used interchangeably with *section* in this document. **Preferred term: block.** |
| **Observed State** | `installed.json` — a local performance cache of what was last applied. Not a source of truth. |
| **`init`** | Bootstrap operation: clone the private configuration repository and establish the connection. No Stow operations happen at `init` time. |
| **`apply`** | Idempotent reconciliation of a single block: make the local system match the manifest for that block, then update `installed.json`. Multiple `apply` calls with the same manifest produce the same result. |
| **`sync`** | Drift-detection + reconciliation across **all** blocks: compares `installed.json` against the system and against the manifest, reports divergences, then runs `apply` for any out-of-date blocks. **Never silently overwrites local-only changes** — see "Dry-run" below. |
| **Drift** | Any divergence between the manifest (desired), `installed.json` (recorded), and the actual system state. |
| **Dry-run** | A read-only preview of what an `apply`/`sync` would change, without touching the filesystem. Mandatory before the first `sync` on a host (see "Failure Model"). |
| **Schema version** | The `version:` field at the top of `manifest.yaml`. Independent of the project SemVer; bumped when the manifest format changes in a backward-incompatible way. |

## Design Decisions
- **Modularity:** Configurations are organized into atomic **blocks**. Each block is an independently `apply`-able unit.
- **Tooling:** Uses `GNU Stow` for symlink management and `python` with `click`/`questionary` for the interactive CLI.
- **Bootstrapping:** `init` clones the private configuration repository (plain `git clone`, not a submodule) to a configured path and writes its location to `installed.json["private_repo"]`. No Stow operations run during `init`.
- **Platform Strategy:** Linux/Ubuntu is the primary target. MacOS support is planned for the future, to be documented in `ROADMAP.md`.
- **Versioning:** The manifest carries a `schema_version` field at its top level, bumped on incompatible format changes. This is independent of the project SemVer (which tags releases of the `dotmgr` itself). Additions to the schema are backward-compatible; removals or type changes require a `schema_version` bump.

## Security

**Secrets must never be stored in any git repository used by this project —
public or private.** This is a hard constraint, not a guideline.

- Configuration (e.g. which packages to install, what dotfiles to link) may live
  in the private repository.
- Secrets (API tokens, SSH private keys, passwords) must be injected at
  runtime via one of:
  - gitignored files: `manifest.local.yaml` overlaid on top of
    `manifest.yaml`, plus a `.env` file that Stow or the engine reads from
    `$XDG_CONFIG_HOME/dotmgr/`.
  - A local secrets manager (e.g. system keyring, HashiCorp Vault) the
    engine queries via a provider interface.
- `init` must refuse to run with an uncommitted secret file present in the
  working tree of the private repository and warn the user.

**Stow safety:** GNU Stow refuses to overwrite an existing non-symlink file.
The CLI must adopt an explicit first-apply policy (chosen by the user during
`init`): **backup + adopt**, **explicit abort with instructions**, or
**skip**. The default is **abort** — never silently clobber a file that is
not a Stow-managed symlink. Manifest paths and Stow target paths are treated
as **trusted input**: a manifest may not retarget symlinks outside the user's
home directory without explicit opt-in.

## Failure Model & Dry-run

- **`apply` per-block atomicity:** a failed `apply` for one block must not
  halt the entire run — record the failure, continue with remaining blocks,
  then exit non-zero with a summary. The block that failed is **not** marked
  as applied in `installed.json`, so the next `sync` retries it.
- **`installed.json` corruption:** treat a missing or malformed file as an empty
  observed state. The engine logs a warning and continues, re-computing drift
  against the live system rather than trusting stale data.
- **Dry-run requirement:** `apply` and `sync` both support `--dry-run` (short:
  `-n`), which prints the planned filesystem diff without writing anything.
  On a fresh host, a dry-run of `sync` is **mandatory** before the first real
  `apply`, so the operator can review Stow's intended symlink targets.

## Extension Points

The following interfaces are declared now so future phases can extend
behavior without rewriting the Phase 1 engine into `if/elif` chains.

- **`PackageProvider`** — interface for installing/removing/querying packages.
  Phase 1 ships `AptProvider` and `PipProvider`. Phase 3 slots in
  `BrewProvider`, `SnapProvider`, `FlatpakProvider` behind the same
  interface. Each provider exposes: `is_available()`, `install(spec)`,
  `remove(spec)`, `list_installed()`.
- **`HostResolver`** — interface for turning an hostname/context into an
  effective manifest: given `manifest.yaml` and a hostname, resolve which
  blocks apply and with which overrides. Phase 1 ships a
  `DefaultHostResolver` (applies all blocks unconditionally). Phase 3 adds
  `HierarchicalHostResolver` supporting a `hosts/<hostname>/` overlay
  directory and per-host variable injection.

## Alternatives Considered (ADR)

- **Why GNU Stow over `chezmoi` / `yadm` / `Ansible`?**
  - Stow is a single-purpose symlink tool with no daemon, no imperative
    playbooks, and a trivial mental model. `chezmoi` and `yadm` couple
    encryption and templating to the manager, which adds surface area and
    makes the "public engine / private config" split harder to reason about.
    Ansible is a full orchestration framework — overkill for per-user
    dotfiles. Stow's symlink model is the narrowest tool that satisfies the
    requirement, and it is already packaged on every target distro.
- **Why Python over Rust / Go?**
  - The MVP's bottleneck is operator judgment and manifest readability, not
    runtime performance. Python keeps the engine hackable by non-systems
    developers, integrates naturally with YAML/JSON manifests, and provides
    `click` + `questionary` for the interactive CLI with minimal ceremony.
    Moving hot paths to Rust later is possible via PyO3 if profiling ever
    shows it is needed; the manifest schema and Stow semantics are portable
    either way.
