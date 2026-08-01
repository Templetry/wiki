# ADR 0011 — Catalog model: parents, forms and combinable features

**Status:** 🟡 Proposed · **Date:** 2026-08-02

## Context

Requirement: catalog parents with multiple *forms* (platform sets, modular vs not, different architecture layouts), combinable. Structural variation cannot be freely combined without breaking the compile principle (ADR-0003) or exploding maintenance (NFR5). Analysis in [study/template-forms-v1.md](../study/template-forms-v1.md).

## Decision (proposed)

Three-level catalog model:

- **Parent**: catalog concept grouping forms (`kmp`, `android`).
- **Form**: one structural variant = one repo that compiles with its own CI. Forms are *chosen*, not combined. Anti-explosion rule: a variant expressible as features must not become a form.
- **Features**: the freely combinable axis inside a form (platform targets, capabilities), using the existing manifest mechanism. Golden rule: additive → feature; structural → form.

Registry moves to `schema_version: 2` (`parents` → `forms`). The manifest spec is unchanged. Versions ride on git refs/tags per form.

Applied immediately: parent `kmp` with forms `modular-base` (kmp-native-base) and `single-module` (compose-multiplatform-app); parent `android` with `modular-base`. Template backlog: platform targets and capabilities as features.

## Consequences

- The overlap between the two KMP templates becomes catalog structure.
- Full combinability lives inside each form; across forms the user picks one — the price of keeping every artifact compilable and maintainable by one person.
- Engine/web will consume registry v2 (no consumer exists yet; migration cost is zero today).
