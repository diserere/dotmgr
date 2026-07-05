# Dotfile Manager (dotmgr)

A declarative, idempotent dotfile manager that stores architecture, decisions,
and agent workflows as first-class, versioned artifacts in the repository
(Context-as-Code).

## What this is

A CLI tool (`dotmgr`) that uses GNU Stow to manage user dotfiles, packages,
and custom scripts across Linux environments. It follows a **Public Engine /
Private Configuration** split: the engine and documentation live in a public
repository, while host-specific and user-specific configuration lives in a
separate private repository.

The project is designed to be driven by AI agents as well as humans: every
architectural constraint, decision, and workflow policy is documented inside
the repo itself, so an agent can recover full context without external sources.

## Why

- Reproducible Linux environments across reinstalls and machines.
- Declarative manifests instead of imperative setup scripts.
- Context lives next to code, so agents and collaborators share the same
  constraints.

## Status

This is pre-Alpha. The current repository contains architecture, a roadmap,
and contribution guidelines only — no engine code yet. See
[`docs/ROADMAP.md`](docs/ROADMAP.md) for planned phases.

## Requirements

- Linux (Ubuntu is the primary target; see ROADMAP for other platforms).
- Python 3.10+ (for the CLI).
- GNU Stow (for symlink management).

## Quick start

Until Phase 1 delivers the `dotmgr` CLI, the repository is documentation-only.
To read the project context:

1. Clone this repository.
2. Read the entry-point files in order (see below).

When the CLI lands, installation will be documented here.

## Development Setup

```bash
# Clone the repository
git clone git@github.com:diserere/dotmgr.git
cd dotmgr

# Create and activate a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install the package in editable mode with dev dependencies
pip install -e .
pip install pytest ruff mypy

# Install pre-commit hooks (optional but recommended)
pip install pre-commit
pre-commit install

# Run checks
ruff check src/ tests/
mypy src/
pytest
```

## Documentation

Read these files in order to recover full project context:

1. `README.md` — this file.
2. [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — system skeleton, Source of
   Truth, and design decisions.
3. [`docs/ROADMAP.md`](docs/ROADMAP.md) — current state and priorities.
4. [`docs/CONTRIBUTING.md`](docs/CONTRIBUTING.md) — development process.
5. [`docs/AGENT_WORKFLOW.md`](docs/AGENT_WORKFLOW.md) — agent collaboration
   policy and review cycle.
6. [`docs/CONTEXT_AS_CODE.md`](docs/CONTEXT_AS_CODE.md) — Context-as-Code
   philosophy.

## Contributing

See [`docs/CONTRIBUTING.md`](docs/CONTRIBUTING.md) for branching, commit,
and language conventions.
