# Architecture: Dotfile Manager (Context-as-Code)

This document is the architectural backbone of the project. It defines the
terminology, the Source of Truth, the failure model, the hard security
constraints, and the extension points that keep `able. Read it
before extending the engine or writing manifests.

## Overview

This project implements a declarative, idempotent system for managing user
dotfiles, packages, and custom scripts across Linux environments. It uses a
**Public Engine / Private Configuration** split to balance reusability with
security: the engine and docmentation live in a public repository, while
host-specific and user-specific configuration lives in a separate private
repository.

## Core Philosophy: Context-as-Code

The project maintains its architecture, design decisions, and roadmap as
first-class citizens inside the repository. AI agents interacting with this
repo are expected to read these files to understand the project's constraints
and intent before performing any non-read-only operation. Context files are
part of the Definition of Done for any PR.

## Source of Truth

1. **Desired State (Manifest):** `manifest.yaml` in the private configuration
   repository is the **absolute source of truth**. It defines what should be
   installed and configured.
2. **Observed/Local State (`installed.json`):** This is a performance cache,
   not a source of truth. The CLI uses it to detect drift and optimize
   performance, but the manifest always overrides it during `sync`
   operations.

## Glossary

| Term | Meaning |
|------|---------|
| **Manifest** | `manifest.yaml` in the private repository — the absolute source of truth for desired state. |
| **Block** | An atomic, named configuration unit inside a manifest. A block maps to a Stow package (directory). *Legacy synonyms: "section", "module" — prefer "block" everywhere.* |
| **Observed State** | `installed.json` — a local performance cache of last-applied state. Not a source of truth. |
| **`init`** | Bootstrap operation: clone the private configuration repository and record its location. No Stow operations happen at `init` time. |
| **`apply`** | Idempotent reconciliation of a **single block**: make the local system match the manifest for that block, then update `installed.json`. Re-running `apply` with the same result. |
| **`sync`** | Drift-detection and reconciliation across **all** blocks: compare `installed.json` against the live system and against the manifest, report divergences, then run `apply` for out-of-date blocks. Never silently overwrites local-only changes. |
| **Drift** | Any divergence between desired state (manifest), recorded state (`installed.json`), and actual system state. |
| **Dry-run** | A read-only preview of what an `apply`/`sync` would change, without touching the filesystem. Mandatory before the first real `sync` on a host (see Failure Model). |
| **Schema version** | The `version:` field at the top of `manifest.yaml`. Independent of the project SemVer; bumped when the manifest format changes in a backward-incompatible way. |

## Design Decisions

- **Tooling:** `GNU Stow` for symlink management; Python 3.10+ with
  `click`/`questionary` for the interactive CLI (see ADR-001).
- **Bootstrapping:** `init` clones the private configuration repository
  (plain `git clone`, not a submodule) to a configured path and records it
  under `installed.json["private_repo"]`. No Stow operations run during
  `init`.
- **Platform Strategy:** Linux/Ubuntu is the primary target. MacOS support is
  planned for the future and tracked in `ROADMAP.md`.
- **Versioning:** The manifest carries a `schema_version` field at its top
  level, bumped on incompatible format changes. This is independent of the
  project SemVer (which tags releases of `dotmgr` itself). Additions to the
  schema are backward-compatible; removals or type changes require a
  `schema_version` bump.

## Security

**Secrets must never be stored in any git repository used by this project —
public or private.** This is a hard constraint, not a guideline.

- Configuration (which packages to install, which dotfiles to link) may live
  in the private repository. Secrets (API tokens, SSH private keys, passwords)
  must be injected at runtime via one of:
  - **Git-ignored local `.env`** at `~/.config/dotmgr/.env`, loaded into the
    environment before applying configurations; overlaid on top of
    `manifest.yaml` via an optional `manifest.local.yaml`.
  - **Local GPG / Age decryption** (advanced): the private repo may carry an
    encrypted file (e.g. `id_rsa.age`); a task runs a decryption command using
    a key already present on the host.
  - **Interactive secure prompting**: for bootstrap or on-demand setups the
    CLI prompts the user in the terminal and injects the input directly into
    the required task without persisting it to disk.
- `init` must refuse to run with an uncommitted secret file present in the
  working tree of the private repository and warn the user.

**Stow safety:** GNU Stow refuses to overwrite an existing non-symlink file.
The CLI must adopt an explicit first-apply policy (chosen by the user during
`init`): **backup + adopt**, **explicit abort with instructions**, or
**skip**. The default is **abort** — never silently clobber a file that is
not a Stow-managed symlink.

**Trust boundaries:** manifest paths and Stow target paths are treated as
trusted input. A manifest may not retarget symlinks outside the user's home
directory without explicit opt-in.

## Failure Model & Dry-run

- **Per-block atomicity:** a failed `apply` for one block halts that block,
  records the failure, and continues with the remaining blocks. The engine
  then exits non-zero with a summary. The failing block is **not** marked as
  applied in `installed.json`, so the next `sync` retries it. This keeps a
  single broken block from taking the whole reconciliation offline.
- **`installed.json` corruption:** treat a missing or malformed file as an
  empty observed state. The engine logs a warning and re-computes drift
  against the live system rather than trusting stale data. Safe re-runs hold
  because Stow symlinks are naturally idempotent.
- **Conflict resolution (symlinks):** when a symlink target already exists as a
  regular file (e.g. a pre-existing `.bashrc`), `dotmgr` halts and alerts the
  user by default. With `--force` it renames the existing file to
  `<name>.dotmgr.bak` and adopts the symlink. This policy is configured during
  `init` (see Security / Stow safety above).
- **Dry-run requirement:** `apply` and `sync` both support `--dry-run` (short:
  `-n`), which prints the planned filesystem diff — packages to install,
  symlinks to create, tasks to execute — without writing anything. On a fresh
  host a dry-run of `sync` is **mandatory** before the first real `apply`, so
  the operator can review Stow's intended symlink targets.

## Extension Points

The following interfaces are declared now so future phases can extend behavior
without rewriting the Phase 1 engine into `if/elif` chains.

### `PackageProvider` — package management abstraction

```python
class BasePackageProvider:
    def is_available(self) -> bool:
        ...

    def install(self, spec: dict) -> bool:
        ...

    def remove(self, spec: dict) -> bool:
        ...

    def list_installed(self) -> list[str]:
        ...
```

Phase 1 ships `AptProvider` (Ubuntu, via `apt-get`) and `PipProvider`
(Python dependencies). Phase 3 slots in `BrewProvider` (MacOS), `SnapProvider`,
and `FlatpakProvider` behind the same interface.

### `HostResolver` — host/context abstraction

```python
class HostResolver:
    def get_hostname(self) -> str:
        ...

    def get_os(self) -> str:
        ...

    def evaluate_condition(self, condition: str) -> bool:
        ...
```

Phase 1 ships a `DefaultHostResolver` (applies all blocks unconditionally).
Phase 3 adds `HierarchicalHostResolver` supporting a `hosts/<hostname>/`
overlay directory and per-host variable injection, enabling declarations like
`condition: "host.os == 'ubuntu'"` or
`condition: "host.name == 'desktop-home'"`.

## Alternatives Considered (ADR)

### ADR-001: Python over Rust / Go for the CLI engine

- **Context:** we need a CLI engine that is easy to write, maintain, and test in
  clean Ubuntu installations and virtualized environments.
- **Decision:** implement the engine in **Python 3.10+**, using `click` for CLI
  argument parsing and `questionary` for interactive prompts.
- **Consequences:**
  - *Pros:* Python is present by default on modern Ubuntu, keeps the engine
    hackable by non-systems developers, and integrates naturally with
    YAML/JSON manifests. Bootstrap dependencies (`click`, `questionary`,
    `pyyaml`) install via a lightweight pip bootstrap step.
  - *Cons:* slightly lower runtime performance than a compiled binary; hot
    paths can be ported to Rust later via PyO3 if profiling shows it is needed.
    The manifest schema and Stow semantics are portable either way.

### ADR-002: GNU Stow over `chezmoi` / `yadm` / `Ansible`

- **Context:** symlinking is the core mechanism for applying config files; writing
  a custom, bug-free symlink orchestrator from scratch is error-prone.
- **Decision:** orchestrate symlinks with **GNU Stow**.
- **Consequences:**
  - *Pros:* a single-purpose symlink tool with no daemon, no imperative
    playbooks, and a trivial mental model. `chezmoi` and `yadm` couple
    encryption and templating to the manager, which adds surface area and
    makes the "public engine / private config" split harder to reason about.
    Ansible is a full orchestration framework — overkill for per-user
    dotfiles. Stow is already packaged on every target distro.
  - *Cons:* adds `stow` as a required package in the bootstrap phase (easily
    installed via `apt install stow`).
