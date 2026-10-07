# Operational Guidelines for AI Agents

Welcome agent! This document contains essential instructions and workflows for working productively and quickly in `px0`.

---

## 1. Quick Development Workflow (Fast Inner Loop)

To keep your iteration fast and avoid unnecessary delays:

| Task | Command | Typical Time | Notes |
| :--- | :--- | :--- | :--- |
| **Fast Verification** | `make check` or `go test -short .` | **~10s** | Use this during active code editing. Runs fast unit tests and bundle checks. |
| **Build Binary** | `make build` | **~2s** | Bundles frontend (`web/app.js`) and compiles local `./px0` binary. |
| **Frontend Bundle** | `make web` or `node scripts/build-web.js` | **<100ms** | Run whenever modifying files under `web/src/`. |
| **Full Test Suite** | `make test` or `go test .` | **~20s** | Run before concluding your task or opening a PR. |

> [!TIP]
> Always run `go test .` rather than `go test ./...` in the root package to avoid traversing external directories or fixtures.

---

## 2. Codebase Organization

- **Backend Architecture**: All Go source files live in the root package (`package main`). There are no subpackages. Assets are statically embedded via `//go:embed`.
- **Frontend Architecture**: Modular ES modules reside in [`web/src/`](web/src/) and are bundled into [`web/app.js`](web/app.js) via [`scripts/build-web.js`](scripts/build-web.js).
- **Benchmark Corpus**: Heavy repositories for benchmarking live in `../bench-repos` (outside the project root) to keep workspace searches and indexers fast.
- **Detailed Map**: For a complete file catalog and UI element mapping, see [`docs/agents/README.md`](docs/agents/README.md).

---

## 3. Core Architectural Tenets

1. **Reads First, Edits via Harness**: `px0` is an ultra-fast code reader and reviewer. File modifications are delegated to external coding harnesses (`agent.go`, `thread.go`), never authored directly to disk by `px0` (with the exception of explicit user-initiated Git operations in the sidebar).
2. **Zero Runtime Dependencies**: The output must remain a single, standalone static binary. No CGO, no Node.js runtime requirement for the user, no external databases.
3. **No Workspace Litter**: `px0` never creates `.px0/` folders or cache files inside a workspace tree. Persistent configuration and thread transcripts live in `$XDG_CONFIG_HOME/px0/` or `~/.px0/`.
4. **Fast & Non-Blocking**: Operations must remain responsive. All background tasks, network calls, and git operations must respect timeouts and set `GIT_TERMINAL_PROMPT=0`.

---

## 4. Documentation Policy

- **During Iterative Work**: Focus on code changes and fast verification (`make check`). Do not block your workflow with premature documentation edits.
- **On Feature Completion**: If you introduce a new feature, CLI flag, shortcut, or major architectural change, update the corresponding documentation:
  - CLI flags / Shortcuts -> [`README.md`](README.md)
  - Major architectural patterns -> [`docs/internals/`](docs/internals/)
  - User features -> [`docs/features/`](docs/features/)

---

## 5. Version Bump & Release Protocol

When committing a version bump (triggered after updating the `VERSION` file):
1. **Verify Release Pipeline**:
   - Verify frontend bundling: `node ./scripts/build-web.js`
   - Verify full test suite: `make test`
   - Verify builds: `build.sh` / `.github/workflows/release.yml`
2. **Commit Release Fixes First**: If bundlers or scripts need fixes, commit those separately first.
3. **Commit Version Bump**: Stage `VERSION` with the new version number using the standard release commit message.
