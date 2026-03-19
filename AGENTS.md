# AGENTS.md

## Cursor Cloud specific instructions

### Overview

**opcode** is a Tauri 2 desktop app (React 18 + TypeScript + Vite 6 frontend, Rust backend) providing a GUI for Claude Code. It also has a secondary `opcode-web` binary that serves the same UI via HTTP on port 8080.

### Key development commands

See `README.md` and `package.json` for the full list. Highlights:

| Command | Purpose |
|---------|---------|
| `bun install` | Install frontend dependencies |
| `bun run dev` | Vite dev server only (port 1420) |
| `bun run tauri dev` | Full dev mode (frontend + Rust, needs display) |
| `bunx tsc --noEmit` | TypeScript type checking |
| `cd src-tauri && cargo check` | Rust type checking |
| `cd src-tauri && cargo clippy` | Rust linting |
| `cd src-tauri && cargo fmt --check` | Rust format check |
| `cd src-tauri && cargo test` | Rust unit tests |
| `cd src-tauri && cargo build --bin opcode-web` | Build web server binary |
| `cd src-tauri && cargo run --bin opcode-web` | Run web server on port 8080 |

### Important caveats

- **Frontend must be built before Rust compilation**: `cargo check`/`cargo build`/`cargo test` require `dist/` to exist. Run `bunx vite build` (or `bun run dev` in background) before any Rust commands. The Rust code uses `include_str!("../../dist/index.html")` and `tauri::generate_context!()` which expect the built frontend.
- **Headless environments**: `bun run tauri dev` requires a display (X11/Wayland). In headless Cloud Agent VMs, use the `opcode-web` binary instead: build the frontend with `bunx vite build`, then `cd src-tauri && cargo run --bin opcode-web` to serve on port 8080.
- **Bun is the primary package manager**: The project uses `bun.lock`. Use `bun install` (not npm/yarn/pnpm).
- **Bun binary location**: Bun installs to `~/.bun/bin/bun`. Ensure `$HOME/.bun/bin` is on `PATH`.
- **System dependencies required on Linux**: `libwebkit2gtk-4.1-dev`, `libgtk-3-dev`, `libayatana-appindicator3-dev`, `librsvg2-dev`, `libssl-dev`, `libxdo-dev`, `libsoup-3.0-dev`, `libjavascriptcoregtk-4.1-dev`, `patchelf`, `build-essential`, `pkg-config`. These are pre-installed in the VM snapshot.
- **SQLite is bundled**: No external database needed. The `rusqlite` crate compiles SQLite from source via the `bundled` feature.
- **Claude Code CLI is optional for development**: The app wraps the `claude` CLI. Full runtime functionality requires it, but building, testing, and UI development work without it. API responses will return errors about missing `~/.claude` directory, which is expected.
- **Clippy warnings are expected**: The codebase has many dead-code warnings from the web server binary not using all Tauri commands. These are benign.
