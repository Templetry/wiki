# ADR 0010 — Engine execution model: embedded library in a Go backend

**Status:** 🟡 Proposed · **Date:** 2026-08-01

## Context

The engine core exists (library + CLI, ADR-0006). The web app (Phase 3) must render server-side — git pushes and forge tokens never reach the browser. Full analysis in [study/engine-execution-v1.md](../study/engine-execution-v1.md): embed as library vs subprocess vs microservice.

## Decision (proposed)

The web backend is written in **Go and embeds the engine as a library** — another `cmd/` over the same packages. Structured `file:line` errors flow natively to web forms; the manifest's existing JSON tags feed the dynamic-form endpoint directly. One binary, one container on the VPS.

Renders are synchronous (sub-second); verify stays out of the MVP web flow (each template's CI already verifies).

The CLI remains the engine's vehicle for the other two contexts: developer terminals and template CI.

## Consequences

- Phase 3 stack: Go backend + TS/React frontend.
- No IPC layer, no job queue, no separate engine service to operate.
- Hardening item recorded before third-party templates: dest paths must be validated against output-root escape.
