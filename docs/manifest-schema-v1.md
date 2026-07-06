# Manifest Schema v1

This document defines the `manifest.yaml` schema — the absolute source of truth
for desired system state in dotmgr. The schema version is **1** and is
independent of the project SemVer (see
[ADR-001](adr/ADR-001-python-over-rust.md) for versioning policy).

## Overview

The manifest is a YAML file stored in the **private configuration repository**.
It declares which blocks (Stow packages), system packages, and custom tasks
should be applied to a host.

```yaml
schema_version: 1
blocks:
  - name: zsh
    ...
```

## Schema Reference

### Top-level fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schema_version` | integer | yes | Must be `1`. Bumped on backward-incompatible changes. |
| `blocks` | array | yes | Ordered list of configuration blocks (min 1). |

---

### Block object

Each block is an atomic, named configuration unit that maps to a Stow package.
Blocks are applied in the order they appear in the manifest.

#### Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | yes | — | Unique identifier. Used as Stow package name and CLI argument (`dotmgr apply <name>`). Pattern: `^[a-zA-Z0-9_-]+$`. |
| `description` | string | no | — | Human-readable description. |
| `enabled` | boolean | no | `true` | Toggle block on/off without removing it from the manifest. |
| `stow` | object | no* | — | Stow symlink configuration. |
| `file_permissions` | array | no | — | Permission rules applied after Stow (requires `stow`). |
| `packages` | object | no* | — | Package declarations per provider. |
| `tasks` | array | no* | — | Custom shell tasks. |
| `condition` | string | no | — | Host condition expression (Phase 3, currently ignored). |

\* At least one of `stow`, `packages`, or `tasks` must be present.

---

### `stow` object

Controls symlink management via GNU Stow.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `source` | string | yes | — | Directory name inside `<private-repo>/dotfiles/`. This is the Stow package. Pattern: `^[a-zA-Z0-9_-]+$`. |
| `target` | string | no | `~` | Target directory for symlinks. |

**Example:**
```yaml
blocks:
  - name: git
    stow:
      source: git        # Links <private>/dotfiles/git/ → ~/
      target: ~

  - name: nvim
    stow:
      source: nvim       # Links <private>/dotfiles/nvim/ → ~/.config/
      target: ~/.config
```

Private repository structure for the examples above:
```
private-repo/
├── manifest.yaml
└── dotfiles/
    ├── git/
    │   ├── .gitconfig
    │   └── .gitignore_global
    └── nvim/
        └── .config/
            └── nvim/
                ├── init.lua
                └── lua/
```

---

### `file_permissions` array

Post-processing rules that set file permissions after Stow creates symlinks.
Symlinks themselves inherit permissions from the files they point to, so these
rules modify the **source files** inside the private repository.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `pattern` | string | yes | — | Glob pattern relative to the stow source directory. E.g. `**/*.sh`, `bin/*`, `**/*`. |
| `mode` | string | yes | — | Octal permission mode. E.g. `755` for executable, `644` for readable. Pattern: `^[0-7]{3,4}$`. |

**Example:**
```yaml
blocks:
  - name: scripts
    description: Custom scripts in PATH
    stow:
      source: scripts
      target: ~/.local/bin
    file_permissions:
      - pattern: "**/*"
        mode: "755"
      - pattern: "**/*.py"
        mode: "644"
```

This is equivalent to running `chmod -R 755` on all files in the scripts
package, then `chmod 644` on all `.py` files. Order matters — rules execute
in array order, so later rules can override earlier ones.

See [ADR-002](adr/ADR-002-gnu-stow-over-chezmoi.md) for why Stow was chosen
over alternative symlink managers.

---

### `packages` object

Package declarations keyed by provider name. Currently supported providers:
`apt` (Ubuntu/Debian) and `pip` (Python).

Each provider value is an array of package specs. A spec can be a simple string
(name only) or an object with options.

**Simple form:**
```yaml
packages:
  apt:
    - zsh
    - zsh-syntax-highlighting
  pip:
    - pynvim
    - black
```

**Extended form (with version/state):**
```yaml
packages:
  apt:
    - name: ripgrep
      state: latest
    - name: cmake
      version: "3.28"
```

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | yes | — | Package name. |
| `version` | string | no | — | Specific version to install. Provider-dependent. |
| `state` | string | no | `present` | Desired state: `present` (install if missing), `latest` (upgrade), `absent` (remove). |

---

### `tasks` array

Arbitrary shell commands executed sequentially after packages and Stow symlinks
have been applied for the block. Each task runs via `sh -c`.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | yes | — | Human-readable label for logs and errors. |
| `command` | string | yes | — | Shell command to execute. |
| `when` | string | no | — | Condition expression (syntax TBD, Phase 3). If present and evaluates to falsy, the task is skipped. |

**Example:**
```yaml
blocks:
  - name: zsh
    stow:
      source: zsh
    packages:
      apt:
        - zsh
    tasks:
      - name: Set zsh as default shell
        command: chsh -s "$(which zsh)" "$(whoami)"
      - name: Install Oh My Zsh
        command: sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
        when: "not test -d ~/.oh-my-zsh"
```

---

### `condition` field

Placeholder for host selection (Phase 3 — `HierarchicalHostResolver`). The
syntax is not yet defined. When implemented, blocks with conditions will only
apply to matching hosts.

```yaml
blocks:
  - name: desktop-apps
    packages:
      apt:
        - steam
        - discord
    condition: "host.name == 'desktop-home'"
```

---

## Full Example

```yaml
schema_version: 1

blocks:
  - name: zsh
    description: Zsh shell with Oh My Zsh
    stow:
      source: zsh
    packages:
      apt:
        - zsh
        - zsh-syntax-highlighting
    tasks:
      - name: Set zsh as default shell
        command: chsh -s "$(which zsh)" "$(whoami)"

  - name: git
    description: Git configuration
    stow:
      source: git
      target: ~

  - name: nvim
    description: Neovim configuration
    stow:
      source: nvim
      target: ~/.config
    packages:
      apt:
        - neovim
      pip:
        - pynvim

  - name: scripts
    description: Custom scripts in PATH
    stow:
      source: scripts
      target: ~/.local/bin
    file_permissions:
      - pattern: "**/*"
        mode: "755"

  - name: dev-utils
    description: Common development tools
    packages:
      apt:
        - build-essential
        - curl
        - git
        - ripgrep
        - tmux
      pip:
        - black
        - ruff
    tasks:
      - name: Verify installations
        command: rg --version && tmux --version
```

## Backward Compatibility

- **Additions** to the schema (new optional fields like `file_permissions`,
  new providers) are backward-compatible.
- **Removals** or **type changes** require a `schema_version` bump.
- The engine must reject manifests with an unknown (higher) `schema_version`.
