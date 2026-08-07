# Study VI — Road to v1.0.0

**Date:** August 2026 · **Status:** study — feeds a future "declare v1.0.0" ADR

When can Templetry honestly call itself 1.0? This study defines what the number means here, inventories the public API against that bar, and proposes the gates and timeline.

## What v1.0.0 means for this project

Semver's contract: from 1.0.0 on, the public API only breaks with a major bump. Two consequences are specific to Templetry:

1. **The manifest is the product's public API** (ADR-0002) — v1.0.0 is a promise made mostly to *template authors*, not to the engine's own code.
2. **Go module rule**: the engine can move from v0.x to v1 keeping its import path; a later v2 would force the `/v2` path suffix on every consumer (CLI, desktop, CI). Breaking after 1.0 is therefore genuinely expensive — the freeze must be real.

Engine and desktop version independently. **v1.0.0 is an engine milestone**; the desktop can (and should) stay 0.x until its own trust gates — signing — are met.

## Public API inventory

| Surface | Spec | Evidence of stability |
|---|---|---|
| `template.yml` schema v1 | [spec/template-yml.md](../spec/template-yml.md) | Dry-run validated across 3 dissimilar ecosystems; 7 real forms compile + render in CI; survived the KMPNativeBase fire test with zero schema changes |
| Directive grammar v1 | [spec/directives.md](../spec/directives.md) | Closed at exactly 3 directives (ADR-0007); densest test suite in the engine |
| Answers file v1 | [spec/answers-file.md](../spec/answers-file.md) | Carried the full update cycle (drift → assisted update → three-way merge) in production use |
| Registry schema v2 | ADR-0011 | Consumed by CLI and desktop; third-party catalogs plug in unchanged |
| CLI surface | `list · init · plan · render · version` | Stable through v0.1.0 → v0.2.2; used by every parent's CI |
| Engine as Go library | module `github.com/Templetry/engine` | Embedded by the desktop app since v0.1 |

The schemas have not needed a breaking change since they were first validated — the strongest signal that the design phase did its job.

## Gates before the freeze

| # | Gate | Size | Why it blocks |
|---|---|---|---|
| G1 | ✅ **Output-root escape hardening** — done: v0.2.1 rejected escapes/absolute/backslash-colon; the 2026-08-07 pass closed the Windows vectors (case-insensitive collisions, reserved device names, trailing dot/space, control chars) and fuzzed the directive scanner (11M+ execs, no panics) | done | Recorded in [engine-execution-v1](engine-execution-v1.md) as mandatory before third-party templates — and v1.0.0 *is* the invitation to third parties |
| G2 | ✅ **`verify` decision** — resolved by shipping it (owner's call, 2026-08-07): `templetry verify --template <dir>` renders to a temp dir (or takes `--dir`) and runs the manifest's verify in Docker. ADR-0004 fully realized. Bonus from its testing: `manifest.Load` now tolerates a UTF-8 BOM (study I §6 edge case) | done | A v1 CLI must not carry a half-promised command; either outcome is fine, ambiguity is not |
| G3 | ✅ **No `planned` holes in the official catalog** — resolved by shipping `android/single-module` (owner's call, 2026-08-07): one-module Kotlin + Compose starter with a `min_sdk` select applied via `tpl:var`. Validated locally (render, zero identity leftovers); registry flips to `ready` once its Verify CI is green | done pending CI | v1 should not announce gaps in its own storefront |
| G4 | ✅ **Compatibility policy written** — [spec/compatibility.md](../spec/compatibility.md) (2026-08-07): the six public surfaces, what is explicitly not API, additive-vs-breaking rules, deprecation and support window. Normative from v1.0.0 | done | The promise needs to be legible, not implicit |
| G5 | **Soak window**: 2–4 weeks of real use — generate + update real projects, author at least one template with `meta/template` outside the org — with **zero schema changes forced by usage** | calendar time | The only honest test of "the schema holds"; a change during the window resets it |

Deliberately **not** gates: desktop signing and macOS/Linux builds (desktop's own 1.0 track), settings sync, multi-forge adapters (BYOR covers v1, ADR-0009), `requires`/`conflicts` between features (additive — can land in a 1.x minor).

## Proposed timeline

| Release | Content | When |
|---|---|---|
| desktop v0.3.0 | Three sections (Build/Cloud/Local), local scan, previews, markdown reader — already on `main` | now (retag after the Actions outage dropped v0.2.1) |
| engine v0.3.0 | G1 hardening (+ scanner fuzz pass while touching it) | days |
| engine v0.4.0 | G2 outcome (`verify` or its descope ADR) · G3 catalog completeness | ~1–2 weeks |
| engine v1.0.0-rc | Schemas frozen, G4 policy published, G5 soak window opens | mid-August 2026 |
| **engine v1.0.0** | Soak window closes clean | **~end of August / early September 2026** |

## Verdict

The expensive parts of a credible 1.0 — deterministic engine, validated schemas, CI-verified catalog, the full update cycle — already exist and are the hard evidence. What remains is small code (G1), one honest decision (G2), one content item (G3), one document (G4) and calendar time (G5). **v1.0.0 is a matter of weeks, not months, and the date is governed by the soak window, not by engineering effort.**
