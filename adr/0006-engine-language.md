# ADR 0006 — Engine implementation language

**Status:** 🟡 Proposed · **Date:** 2026-08-01

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

## Study

Full technology and architecture study in [study/engine-tech-v1.md](../study/engine-tech-v1.md) (2026-08). Key finding: distribution no longer discriminates (GraalVM native and `bun build --compile` are both production-mature), so the decision hinges on author affinity (→ Kotlin), niche ecosystem fit (→ TypeScript) and reuse with the phase-3 web app (→ TypeScript, via a single zod manifest schema). Go exits the race.

## Decision

*Proposed: TypeScript on Bun — pure library `@templetry/engine` + thin `templetry` CLI, distributed via `bun build --compile`. Kotlin remains the defensible alternative if author affinity is weighted first. Pending acceptance.*
