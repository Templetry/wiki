# Engine study III — Execution model

**Date:** August 2026 · **Status:** study for ADR-0010

Where and how the engine runs, now that it exists.

## The three execution contexts

| Context | How the engine gets there | Status |
|---|---|---|
| Developer terminal | CLI with the engine embedded (`templetry plan/render`) | ✅ exists |
| Web app (Phase 3) | **the decision this study makes** | proposed below |
| Template CI | the release CLI binary, rendering feature combinations for verification | free — same CLI |

Rendering for the web MUST happen server-side: git pushes and forge tokens live on the server, never in the browser. (Curiosity noted and discarded: Go compiles to WASM and could render in-browser, but pushing git from a browser needs forge-specific tree APIs — against ADR-0009's minimal-adapter rule.)

## How the web app invokes the engine — three options

**A. Embedded library in a Go backend.** The backend imports the engine packages directly. No IPC; structured errors (`file:line` values) flow untouched from scanner to HTTP response to web form. One binary, one container.

**B. Subprocess.** Any-language backend shells out to `templetry render`. Maximum isolation, language freedom; costs JSON serialization of everything and process management.

**C. Microservice** (`templetryd`). Full separation — and operational complexity a personal-VPS project cannot justify (the anti-Backstage rule again).

## Proposal: A — embedded in a Go backend

The engine was built for this without knowing it: a pure library with a CLI on top (NFR3) means the web backend is just another `cmd/` over the same packages. Consequences:

- **Phase 3 stack resolves**: backend in Go (embeds engine), frontend stays TS/React. ADR-0006's accepted trade-off ("no code sharing with the web") shrinks: backend and engine are one program; only the frontend is another language.
- The manifest types already carry JSON tags — the dynamic-form endpoint is `json.Marshal(manifest)`.
- **Sync render, async-or-absent verify**: renders are sub-second (plain HTTP request); verify takes minutes and the MVP web doesn't need it — each template's CI already verifies. No job queue in the MVP.
- Deployment: one container on the VPS; Docker socket available there for future in-web verify.

## Hardening item (recorded, not blocking)

The engine does not yet validate that renamed `dest` paths stay inside the output root (a hostile identity `to:` containing `../` could escape). Irrelevant for first-party templates; mandatory before accepting third-party ones. Tracked for the hardening pass.
