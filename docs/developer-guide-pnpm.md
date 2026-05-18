---
layout: default
title: Developer Guide (pnpm)
nav_order: 7
---

# Developer Guide (pnpm)

## Prerequisites

- Node.js 18+
- Rust (stable channel)
- Scoop installed locally
- Visual Studio Build Tools (or the full [Tauri prerequisites](https://v2.tauri.app/start/prerequisites/))
- **pnpm** (recommended package manager)

## Setup

```bash
git clone https://github.com/AmarBego/rscoop.git
cd rscoop
pnpm install
```

## Environment Configuration

Ensure pnpm is properly configured with the following settings:

```bash
# Verify pnpm installation
pnpm --version

# Check current pnpm configuration
pnpm config list

# Recommended pnpm configuration (optional)
# Set global store directory to avoid C:\ drive space issues
pnpm config set store-dir "d:\packages\pnpm\store"
pnpm config set global-dir "d:\packages\pnpm\global"
pnpm config set global-bin-dir "d:\packages\pnpm\global-bin"
pnpm config set cache-dir "d:\packages\pnpm\cache"
```

## Development Workflow

### Run in dev mode with hot reload:

```bash
pnpm run tauri dev
```

### Frontend only (just Vite, no Rust rebuild):

```bash
pnpm run dev
```

### Build production installers:

```bash
pnpm run tauri build
```

Output goes to `src-tauri/target/release/bundle`.

### Additional Commands

```bash
# Install a new dependency
pnpm add <package-name>

# Install a dev dependency
pnpm add -D <package-name>

# Update dependencies
pnpm update

# Clean node_modules and reinstall
pnpm install --clean

# Run specific script
pnpm run <script-name>
```

## Directory layout

| Folder | What's in it |
|---|---|
| `src/` | SolidJS frontend. Pages, components, hooks, stores. |
| `src-tauri/` | Rust backend. Commands, Tauri config, plugins. |
| `pics/` | Screenshots for the README |
| `docs/` | This documentation site (GitHub Pages) |

## Backend notes

- Commands are in `src-tauri/src/commands/`, grouped by domain (search, install, buckets, doctor, profile, etc.).
- `operations.rs` manages the background install/update/uninstall queue. Use `EnqueueAction` to push work; the queue processes FIFO via Tokio tasks.
- Execra is the runtime for long-running jobs. Use it for process execution, cancellation, streamed output, and structured operation status instead of adding new ad-hoc process wrappers.
- `tray.rs` builds the tray menu from installed Scoop apps, extracting real exe icons and supporting pinned/hidden app preferences.
- Use the existing Rust helpers for probing Scoop state, parsing manifests, cache cleanup, shim/shortcut inspection, and filesystem operations. rScoop delegates core package actions to Scoop, but most surrounding app logic should stay in Rust for speed and predictable error handling.
- Log progress with `log::info!` / `log::warn!`. The frontend operation modal picks these up through `tauri-plugin-log`.

## Frontend notes

- Hooks in `src/hooks/` wrap backend calls and manage state. Extend existing hooks instead of duplicating `invoke` calls in components.
- `installedPackagesStore` holds the canonical package list. Call its `refetch()` after any operation that changes Scoop state.
- Settings store in `src/stores/` uses `tauri-plugin-store` for persistence.
- UI is built with Tailwind + daisyUI. Shared styles are in `App.css`.

## Debugging

- Open **Settings > About** to see version info, check for updates, and read logs.
- Set the `RUST_LOG` environment variable for verbose output during development.
- The system tray has a **Refresh Apps** entry that reloads Scoop app shortcuts without restarting.
- Enable **Debug Mode** in **Settings > Window** to unlock rapid test intervals for the auto-update scheduler and access a debug info panel.

## pnpm Benefits

- **Faster installation**: pnpm uses a shared package store, avoiding redundant downloads.
- **Less disk space**: Packages are stored once and hard-linked across projects.
- **Better dependency isolation**: No "phantom dependencies" - only direct dependencies are accessible.
- **Monorepo support**: Native workspaces support for multi-package projects.

## Migration Notes

From npm to pnpm:

1. Remove `package-lock.json` if it exists
2. Ensure `pnpm-lock.yaml` is present
3. Run `pnpm install` to install dependencies
4. Update any CI/CD scripts to use `pnpm` commands

## Troubleshooting

### pnpm command not found

Add pnpm to PATH:
```bash
# Windows
$env:PATH += ";C:\Users\<username>\AppData\Local\pnpm"
```

### Build fails with cargo metadata error

Ensure Rust toolchain is properly configured:
```bash
# Add Rust to PATH
$env:PATH += ";C:\Users\<username>\scoop\apps\rustup\current\.cargo\bin"
```

### Global dependencies not found

Ensure global bin directory is in PATH:
```bash
$env:PATH += ";d:\packages\pnpm\global-bin"
```
