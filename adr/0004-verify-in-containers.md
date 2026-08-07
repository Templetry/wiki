# ADR 0004 — Verification runs in containers, declared in the manifest

**Status:** Accepted · **Date:** 2026-08-01

## Context

Verifying that the output compiles requires a different toolchain per ecosystem (Gradle, Node, Cargo...). Installing them all on the host doesn't scale and couples the engine to frameworks (against ADR-0002).

## Decision

Each template declares its verification in the manifest: `verify: {image, run}` (e.g. `image: node:22`, `run: npm ci && npm run build`). The engine only knows how to run a command in a Docker container.

## Consequences

- The host needs no toolchains; only Docker.
- In interactive use, verify is optional/async (an Android build takes minutes); in each template's CI it is mandatory.
- Fits the planned deployment (a VPS with Docker already running).

> **Outcome (2026-08-07): fully realized.** `templetry verify --template <dir>` renders to a temp dir (or takes `--dir`) and runs the manifest's `verify: {image, run}` in Docker via the `verify` package. Template CI remains the mandatory gate; the CLI step is the local-use complement.
