# ADR 0014 — Lazy pieces: decoupled units with their own lifecycle

**Status:** ✅ Accepted · **Date:** 2026-08-12

## Context

Features toggle content at render time and then dissolve into the project; nothing lets a generated project *adopt* a unit of code later, keep it updated, or customize it separately. The owner's requirement and the full analysis live in [study VII](../study/pieces-v1.md).

## Decision

Adopt the pieces model exactly as study VII shapes it:

1. A **piece** is a directory plus `piece.yml` (own variables, patches) shipped inside its form under `pieces/<name>/`, written in the form's canonical identity.
2. **Decoupling is enforced**: piece files must not collide with existing project paths; shared files are touched only through declared patches. No merge on add.
3. The **answers file records each applied piece** — source, resolved commit, variables, owned files — making pieces independently updatable through the lane-A update machinery, and keeping future removal possible.
4. **v1 is template-local**; common (cross-template) pieces wait for the lifecycle to prove itself and will arrive with an additive registry change and compat declarations.
5. The golden rule stack grows one level: **additive → feature · structural → form · independent lifecycle → piece.**

## Consequences

- All changes are additive (answers `pieces:` key, new CLI/MCP surface, `pieces/**` excluded from form renders) — legal minors under the compatibility policy.
- Form CI grows one linear job per piece (base + piece compiles), not a combinatorial matrix.
- A piece that cannot live without rewriting project files is rejected by construction — the model polices its own boundary.
