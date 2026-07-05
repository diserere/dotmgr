# QA Strategy: Quality Assurance for dotmgr

This document outlines the testing approach for the dotfile manager, ensuring robustness, safety, and correctness of configuration deployment.

## Testing Layers

1. **Unit Testing:** 
   - Focus: `PackageProvider` implementations and `HostResolver` logic.
   - Strategy: Use Python's `unittest.mock` to simulate system state (e.g., simulate `apt` return values, OS environment) without touching the actual filesystem.

2. **Integration Testing:**
   - Focus: End-to-end `dotmgr` CLI commands (`init`, `apply`, `sync`).
   - Strategy: Test inside transient, disposable **Docker containers** (e.g., `ubuntu:latest`). This ensures we are testing against a clean, real environment.
   - Tooling: A dedicated `tests/` directory with scripts that set up a dummy Private Config repo, apply it, and verify the resulting symlinks/packages on the container filesystem.

3. **Validation (Dry-Run):**
   - Focus: Safety.
   - Strategy: Automated CI/CD pipelines MUST execute `dotmgr sync --dry-run` before any real `apply` is performed. This validates that the manifest configuration is syntactically correct and Stow can resolve the symlinks.

## Definition of Done (DoD)

The consolidated Definition of Done for the project lives in
[`docs/AGENT_WORKFLOW.md`](docs/AGENT_WORKFLOW.md#definition-of-done).
