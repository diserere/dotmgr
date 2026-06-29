# Architecture: Dotfile Manager (dotmgr)

## 1. Glossary & Terminology

To prevent ambiguity across the CLI codebase and configuration structures, the following terms are formally defined:

*   **Engine (CLI / `dotmgr`):** The Python-based CLI utility that parses the user's manifest, evaluates conditions, resolves dependencies, and applies the desired state to the target environment.
*   **Private Config (Repository):** The user's private, version-controlled repository on GitHub/Gitlab containing custom configurations, scripts, and the primary `manifest.yaml`.
*   **Manifest (`manifest.yaml`):** The declarative configuration file located at the root of the Private Config repository. It defines the desired state of all modules.
*   **State Cache (`installed.json`):** A local JSON file (typically stored in `~/.local/state/dotmgr/installed.json`) representing the observed state of applied modules. This is a performance cache, not the source of truth.
*   **Module:** The atomic unit of configuration (standardized term replacing "block" or "section"). A module consists of system packages, symlink packages, custom install tasks, and application conditions.
*   **Package:** A system-level software dependency managed by a system package provider (e.g., `apt`).
*   **Symlink Configuration:** A set of files/directories inside the config repo orchestrated onto the target environment (usually `$HOME`) via symlinks.
*   **Task:** A custom command or script run during the execution of a module.

---

## 2. Secrets Management Strategy

Storing sensitive credentials (SSH keys, API tokens, passwords) in version control is a major security risk. `dotmgr` enforces a zero-secrets policy in the git repository.

### Secrets Injection Architecture
1.  **Git-Ignored Local Environment (`.env`):**
    *   A `.env` file containing local secrets is stored outside the repo at `~/.config/dotmgr/.env`. This file is loaded into the environment before applying configurations.
    *   `dotmgr` can read this file and inject variables during symlink processing or task execution.
2.  **Decryption via Local GPG / Age (Optional/Advanced):**
    *   The Private Config repository can contain encrypted files (e.g., `id_rsa.age`).
    *   A task in the manifest can execute a decryption command using a local private key already present on the host.
3.  **Interactive Prompting:**
    *   For bootstrap or on-demand setups, `dotmgr` can prompt the user securely in the terminal and inject the input directly into the required task without saving it to disk.

---

## 3. Failure Model, Dry-Run, and Idempotency

### Dry-Run Mode
To ensure safe deployments, `dotmgr` supports a `--dry-run` flag. In this mode, the Engine will:
*   Parse the manifest and evaluate conditions.
*   Display exactly what packages would be installed.
*   Display exactly what symlinks would be created by `GNU Stow` and their targets.
*   Display exactly what custom tasks would be executed.
*   *No side-effects will be applied to the host system.*

### Failure Model & Idempotency
1.  **Atomicity of Modules:** Each Module is applied sequentially. If a task or package installation within a module fails, the entire module's execution halts, and it is NOT marked as installed in the `installed.json` cache.
2.  **Safe Re-runs:** Since `GNU Stow` symlinks are naturally idempotent, re-running `dotmgr setup` is entirely safe. If a run fails midway, running it again will safely skip already-applied symlinks and retry only the failed steps.
3.  **Conflict Resolution (Symlinks):** If a symlink target already exists as a regular file (e.g., a pre-existing `.bashrc`), `dotmgr` will NOT overwrite it. Instead, it will:
    *   Halt and alert the user.
    *   Optionally rename the existing file to `.bashrc.dotmgr.bak` and proceed if `--force` is specified.

---

## 4. Extension Points (Abstractions)

To ensure future-proofing for other platforms (such as MacOS) and package managers, the Engine utilizes modular abstractions:

### PackageProvider Interface
An abstract interface for handling package installations.
```python
class BasePackageProvider:
    def is_installed(self, package_name: str) -> bool:
        pass
    def install(self, package_name: str) -> bool:
        pass
```
*   **AptProvider:** (Default for Ubuntu) Utilizes `apt-get` for package management.
*   **BrewProvider:** (Future MacOS) Utilizes `brew` for package management.
*   **PipProvider:** (Future Python dependencies) Utilizes `pip` inside target envs.

### HostResolver Interface
Responsible for inspecting host characteristics to resolve conditional variables in `manifest.yaml`.
```python
class HostResolver:
    def get_hostname(self) -> str:
        pass
    def get_os(self) -> str:
        pass
    def evaluate_condition(self, condition: str) -> bool:
        pass
```
This enables declarations like `condition: "host.os == 'ubuntu'"` or `condition: "host.name == 'desktop-home'"`.

---

## 5. Architectural Decision Records (ADRs)

### ADR-001: Choosing Python for CLI Engine
*   **Context:** We need a robust CLI engine that is easy to write, maintain, and run in virtualized test environments and clean Ubuntu installations.
*   **Decision:** Implement the CLI engine in **Python 3** using standard packaging, `click` for CLI argument parsing, and `questionary` for interactive prompts.
*   **Consequences:** 
    *   *Pros:* Python 3 is installed by default in almost all modern Ubuntu releases. Development speed is extremely high, and the user (QA engineer) knows Python.
    *   *Cons:* Requires bootstrapping dependencies (`click`, `questionary`, `pyyaml`) via a lightweight pip bootstrap script in `bootstrap.sh`.

### ADR-002: Choosing GNU Stow for Symlink Management
*   **Context:** Symlinking is the core mechanism of applying configuration files. Writing a custom, bug-free symlink orchestrator from scratch is error-prone.
*   **Decision:** Orchestrate symlinks using **GNU Stow**.
*   **Consequences:**
    *   *Pros:* `Stow` is a mature, standard Linux package. It cleanly handles directories, handles nested path creation, and cleanly un-links files on deletion.
    *   *Cons:* Adds `stow` as a required package dependency in the bootstrap phase (easily installed via `apt install stow`).
