# Dotfile Manager (dotmgr)

A declarative, idempotent system for managing user dotfiles, packages, and custom scripts across Linux environments.

## Quick Start
1. **Bootstrap:** 
   `wget -qO- https://raw.githubusercontent.com/my-tool/engine/main/bootstrap.sh | sh`
2. **Setup:**
   `dotmgr init --config <YOUR_PRIVATE_REPO_URL>`
3. **Configure:**
   `dotmgr setup`

## Core Philosophy
- **Context-as-Code:** All architectural decisions, constraints, and workflows are stored as first-class, versioned artifacts within the repository.
- **Security:** Public engine, private configuration. Sensitive data (secrets) MUST NOT be committed to the repository.
- **Source of Truth:** The `manifest.yaml` (Desired State) is the absolute source of truth.

## Documentation
- [Architecture](docs/ARCHITECTURE.md)
- [Roadmap](docs/ROADMAP.md)
- [Contributing](docs/CONTRIBUTING.md)
- [Context-as-Code Philosophy](docs/CONTEXT_AS_CODE.md)
