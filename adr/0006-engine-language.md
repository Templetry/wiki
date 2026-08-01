# ADR 0006 — Engine implementation language

**Status:** ⏳ Open · **Date:** 2026-08-01

## Context

The engine is a pure library + CLI (NFR3), so the language doesn't shape the architecture — it shapes distribution, library ecosystem, and code sharing with the future web app. Needed before the first prototype (Phase 1), not before.

## Options on the table

| Criterion | Kotlin | TypeScript | Go |
|---|---|---|---|
| Author affinity | ★★★ | ★★ | ★ |
| Code sharing with the web app | via a Ktor backend | ★★★ (same stack) | ✗ |
| CLI distribution | JVM or GraalVM native | requires Node | ★★★ single binary |
| Libraries (casings, JSON Patch, tar, YAML) | ★★ | ★★★ | ★★ |
| Risk/novelty | low | low | medium |

## Decision

*Pending.*
