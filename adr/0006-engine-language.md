# ADR 0006 — Engine implementation language

**Status:** ✅ Accepted · **Date:** 2026-08-01

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

Full technology and architecture study in [study/engine-tech-v1.md](../study/engine-tech-v1.md) (2026-08). Findings:

- Distribution no longer discriminates (GraalVM native and `bun build --compile` are both production-mature).
- Runtime-efficiency deep-dive (study Part 5): **Go wins every efficiency column** (startup ~2–10 ms, ~5–20 MB RSS, best throughput and parallelism), Kotlin/GraalVM second, Bun a close third, Kotlin/JVM last. Caveat: the deltas are tens of ms / tens of MB per render against a 30-second success metric — real pipeline costs (fetch, Docker verify) dwarf them by orders of magnitude.

Final decision matrix — each finalist owns one axis: machine efficiency → **Go** · author velocity/affinity → **Kotlin** · niche libraries + future web reuse → **TypeScript**.

## Decision

**Go**, weighting the machine-efficiency axis first (best startup, memory, throughput and per-file parallelism; smallest static binaries; `archive/tar` and golden-file testing are first-class in the stdlib).

Accepted trade-offs, stated explicitly: it is the author's least familiar language (a learning investment absorbed deliberately), and there is no code sharing with the phase-3 web app — the engine will be consumed through its CLI/API boundary.

## Consequences

- Module `github.com/Templetry/engine`; thin CLI `templetry` on top (cobra), pure-library core per the study's Part 1 shapes.
- Reference libraries (study §2.3): goccy/go-yaml, evanphx/json-patch, bmatcuk/doublestar, iancoleman/strcase (casing variants to be extended in-house), stdlib `archive/tar`, `os/exec` for Docker.
- Golden tests follow the idiomatic `testdata/` pattern.
- Cross-compiled release binaries for the three desktop platforms from day one.
