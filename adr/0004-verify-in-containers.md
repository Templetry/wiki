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
