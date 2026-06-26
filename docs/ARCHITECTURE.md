# Architecture: Dotfile Manager (Context-as-Code)

## Overview
This project implements a declarative, idempotent system for managing user dotfiles, packages, and custom scripts across Linux environments. It uses a "Public Engine / Private Configuration" model to balance usability with security.

## Core Philosophy: Context-as-Code
The project maintains its architecture, design decisions, and roadmap as first-class citizens within the repository. AI agents interacting with this repo are expected to read these files to understand the project's constraints and intent.

## Source of Truth
1. **Desired State (Manifest):** `manifest.yaml` in the private configuration repository is the **absolute source of truth**. It defines what should be installed and configured.
2. **Observed/Local State (`installed.json`):** This is a performance cache, not a source of truth. The CLI uses it to detect drifts and optimize performance, but the manifest always overrides it during `sync` operations.

## Design Decisions
- **Modularity:** Configurations are organized into atomic "blocks" (sections).
- **Tooling:** Uses `GNU Stow` for symlink management and `python` with `click`/`questionary` for the interactive CLI.
- **Bootstrapping:** A public engine repository provides the installation logic, while a private repository holds host-specific and user-specific secrets/configurations.
- **Platform Strategy:** Linux/Ubuntu is the primary target. MacOS support is planned for the future, to be documented in `ROADMAP.md`.
- **Versioning:** The manifest uses versioning to ensure backward compatibility as the CLI logic evolves.
