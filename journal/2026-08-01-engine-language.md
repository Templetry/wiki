# 2026-08-01 — Engine study II and language decision

Second session block of the day: inside-out, starting with the engine.

## What happened

1. **Study II written** (`study/engine-tech-v1.md`): the language-independent architecture shapes (virtual file tree, plan-as-data, pure core / effectful edges, plan-then-apply, positioned errors) and the three candidate stacks mapped to concrete libraries.
2. **2026 distribution finding**: GraalVM native image and `bun build --compile` are both production-mature — distribution stopped discriminating between stacks.
3. **Efficiency deep-dive** (study Part 5), requested with the project scoped to engine-only: workload characterization plus per-stack numbers. Ranking: Go > Kotlin/GraalVM > Bun > Kotlin/JVM, with the honest caveat that deltas are tens of ms against a 30-second success metric.
4. **ADR-0006 accepted: Go**, weighting the machine-efficiency axis first. Trade-offs recorded: author's least familiar language, no code sharing with the future web app.
5. Go 1.26.5 toolchain installed; `Templetry/engine` repo scaffolded.

## Next steps

- Dry-run manifest validation (3 hand-written manifests) — still the top Phase 0 item, now feeding directly into Go type definitions.
- Close ADR-0007 (directive grammar) during that validation.
- First engine milestone: the directive scanner with its golden-test suite (highest-risk unit, per study §1.4).
