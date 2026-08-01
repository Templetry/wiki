# ADR 0011 — Catalog model: parents, forms and combinable features

**Status:** ✅ Accepted (amended) · **Date:** 2026-08-02

## Context

Requirement: catalog parents with multiple *forms* (platform sets, modular vs not, different architecture layouts), combinable. Structural variation cannot be freely combined without breaking the compile principle (ADR-0003) or exploding maintenance (NFR5). Analysis in [study/template-forms-v1.md](../study/template-forms-v1.md).

## Decision

Three-level catalog model, **amended by the project owner: the parent IS the repo, and forms live inside it as subdirectories**:

- **Parent**: one repo per concept (`Templetry/kmp`, `Templetry/android`).
- **Form**: one structural variant = one subdirectory of the parent repo, compiling on its own with its own `template.yml`. Forms are *chosen*, not combined. Anti-explosion rule: a variant expressible as features must not become a form.
- **Features**: the freely combinable axis inside a form (platform targets, capabilities), using the existing manifest mechanism. Golden rule: additive → feature; structural → form.

Registry `schema_version: 2`: parents carry `repo` + `ref`; forms carry `path`, `name`, `status` (`ready`/`planned`) and description. The manifest spec is unchanged; the CLI needs no change (`--template <repo>/<form-dir>`).

Applied 2026-08-02: `Templetry/kmp` (forms `modular-features`, `single-module`; `modular-ui` planned) and `Templetry/android` (`modular-features`; `single-module` planned). The three individual template repos are archived, superseded by the parents. Next investment per owner decision: **platform targets as features**.

## Consequences

- The overlap between the two KMP templates becomes catalog structure.
- Full combinability lives inside each form; across forms the user picks one — the price of keeping every artifact compilable and maintainable by one person.
- Engine/web will consume registry v2 (no consumer exists yet; migration cost is zero today).
