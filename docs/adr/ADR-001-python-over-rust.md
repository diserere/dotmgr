# ADR-001: Python over Rust / Go for the CLI Engine

- **Date:** 2026-07-06
- **Status:** accepted

## Context

We need a CLI engine that is easy to write, maintain, and test in
clean Ubuntu installations and virtualized environments.

## Decision

Implement the engine in **Python 3.10+**, using `click` for CLI
argument parsing and `questionary` for interactive prompts.

## Consequences

### Pros

- Python is present by default on modern Ubuntu, keeps the engine
  hackable by non-systems developers, and integrates naturally with
  YAML/JSON manifests.
- Bootstrap dependencies (`click`, `questionary`, `pyyaml`) install
  via a lightweight pip bootstrap step.

### Cons

- Slightly lower runtime performance than a compiled binary; hot
  paths can be ported to Rust later via PyO3 if profiling shows it is needed.
- The manifest schema and Stow semantics are portable either way.
