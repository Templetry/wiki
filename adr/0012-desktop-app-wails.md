# ADR 0012 — Desktop app with Wails, engine embedded

**Status:** ✅ Accepted · **Date:** 2026-08-02

## Context

The owner wants a desktop app to drive the engine. The engine is local-first already (local CLI, local renders), so a desktop UI is its natural extension — no server, no hosted OAuth. Options analyzed: Compose Desktop (owner's stack, engine via CLI subprocess, dogfooding), Wails (Go, engine embedded as a library), Tauri (Rust, sidecar).

## Decision

**Wails**: the desktop app is a Go program that imports the engine packages directly (`github.com/Templetry/engine/...`) with a TS/React frontend. This realizes ADR-0010's embed principle on the desktop: no IPC, structured `file:line` errors reach the UI as values, and the dynamic form is fed by `json.Marshal` of the manifest types.

ADR-0010 (web execution model) stays Proposed for a future hosted phase; the desktop app takes over as the primary UI.

## Consequences

- Repo `Templetry/desktop`; ships as native binaries (WebView2 on Windows).
- The app fetches templates through the same `catalog` + `source` packages as the CLI — one behavior, two frontends.
- Trade-off accepted: UI in TS/React (new-ish stack for the owner) with a thin Go glue layer.
