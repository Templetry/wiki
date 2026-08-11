# ADR 0013 — Declare v1.0.0 across all components

**Status:** ✅ Accepted · **Date:** 2026-08-07

## Context

Study VI defined five gates for v1.0.0. The four engineering gates closed on 2026-08-07 and shipped (engine v0.3.0, desktop v0.3.2, catalog with 7 `ready` CI-verified forms, compatibility policy). The remaining gate, G5, was a 2–4 week soak window of real use before freezing.

## Decision

The project owner declares **v1.0.0 now, waiving the soak window**. The judgment: the evidence already in hand — 8 forms rendered and compiled by CI on every push across three parents, the KMPNativeBase fire test, the full update cycle (drift → assisted update → three-way merge) exercised on real projects, and schemas that have needed zero breaking changes since their dry-run validation — is a stronger signal than additional idle calendar time would be.

Versions: **engine v1.0.0** (the semver promise carrier), **desktop v1.0.0** (embedding engine v1), and symbolic `v1.0.0` tags on `catalog` and the three parents marking the coherent catalog generation.

## Consequences

- [spec/compatibility.md](../spec/compatibility.md) is **in force** from this moment: the six public surfaces only break with a major bump, and a future engine v2 pays Go's `/v2` import-path cost.
- The soak window's purpose survives as practice: real use continues, but schema-breaking lessons now cost a major instead of a free reset.
- Desktop reaches 1.0 without code signing (accepted: signing is a distribution-trust gate, not an API-stability one; it stays on the backlog).
