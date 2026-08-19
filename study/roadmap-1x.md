# Roadmap 1.x — after v1.0.0

**Date:** 2026-08 · **Status:** planning — owner-selected lanes, ordered by dependency and leverage. Each lane graduates into its own study/ADR before implementation.

## The six lanes

### A — `templetry update` in the CLI *(enabler lane)* — ✅ shipped 2026-08-11 (engine + desktop v1.1.0)

The update cycle (drift detection → assisted re-render → three-way merge) lives only in the desktop app today. Moving it into the engine library + CLI:

- makes updates scriptable and CI-friendly (a workflow that opens a PR when your template improves),
- gives the desktop the same behavior through the library it already embeds,
- **is the technical prerequisite of lane F** — pieces reuse exactly this machinery (render with recorded inputs + merge into an existing tree).

Shape: `templetry update [--dry-run]` reading `.templetry-answers.yml`; library packages `update/` (diff+merge, today in desktop's `update.go`) move down into the engine.

### B — Engine additives — ✅ shipped 2026-08-12 as engine v1.3.0

The three candidates the specs already noted, all additive (minor-safe under the [compatibility policy](../spec/compatibility.md)):

- **YAML/TOML patches** (the FastAPI dry-run wanted TOML; spec says "v1.1 planned").
- **`requires`/`conflicts` between features** (postponed from v1 by ADR-0007/spec).
- **Presets** — named feature combos in the manifest (study IV's "future sugar"); also the natural UI for choosing pieces at creation time.

### C — New parent beyond Kotlin — ✅ shipped 2026-08-12 (`Templetry/web`)

A **web** parent (React + Vite + TS) or **backend** parent (Spring Boot or FastAPI — both in the owner's stack). Content work like `android/single-module`; publicly proves the engine-agnostic thesis (ADR-0002) outside the JVM. The FastAPI manifest from the dry-run validation is a ready-made starting spec.

### D — Distribution & trust — 🔶 partially shipped 2026-08-12

- ✅ **Licenses across the org** (prerequisite surfaced by packaging): engine under Apache License 2.0 (relicensed from PolyForm Noncommercial on 2026-08-19), everything else MIT.
- ✅ **scoop bucket** ([Templetry/scoop-bucket](https://github.com/Templetry/scoop-bucket)): `scoop bucket add templetry … && scoop install templetry` installs CLI + MCP server, autoupdating from engine releases.
- ✅ **macOS/Linux desktop builds**: the release workflow is a three-platform matrix (Windows NSIS+portable, Linux GTK/WebKitGTK, macOS universal .app) with one aggregated release; the in-app updater stays Windows-only with a friendly pointer elsewhere.
- Pending: **code signing** (certificate/budget decision), winget submission (better after signing), Homebrew tap, settings sync via GitHub.

### E — AI-ready surface (docs for AIs + MCP) — ✅ MVP shipped 2026-08-11 (engine v1.2.0)

Make Templetry legible and drivable by AI agents:

- **MCP server** exposing the engine: `list_templates`, `get_form_schema` (the manifest is already JSON-taggable — the dynamic form endpoint idea from ADR-0010 reborn as MCP tools), `plan`, `render`, `update`. A thin Go binary over the same library; agents can scaffold and update projects conversationally.
- **AGENTS.md / AI_INDEX standard in every form**: `kmp` and `android/modular-features` already carry AGENTS.md + `docs/ai/`; make it a form-authoring convention (checklist in `meta/template`'s GUIDE) so every *rendered project* is born agent-friendly.
- **Machine-readable specs**: publish the manifest/answers/registry schemas as JSON Schema; `llms.txt` on the wiki/org profile.

### F — Lazy pieces — 🔶 v1 shipped 2026-08-12 (engine v1.4.0; desktop panel, piece updates and common pieces pending)

**Owner's framing:** a template can have *pieces* — fully decoupled units of code you may not want at creation time but can bring in later, with update capability and their own sub-customization. Choosable at creation or added after the project exists. Two kinds: **template-local pieces** (belong to one form) and **common pieces** (shared across templates).

Initial design sketch (to be validated in study VII):

- **A piece is a directory + `piece.yml`**: its own variables (sub-customization), `files`, patches, `requires` (other pieces/features), and a declaration of what it's compatible with (parent/form, or a capability tag for common pieces).
- **Application = the update machinery** (lane A): render the piece with the project's recorded identity inputs + the piece's own answers, three-way merge into the existing tree. The answers file grows a `pieces:` section recording each applied piece + its source commit — so **pieces update independently**, with the same drift → merge cycle as the base template.
- **Template-local pieces** live in the form repo (`<form>/pieces/<name>/`); **common pieces** live in their own catalog level (a `pieces` entry in registry v2.x — additive schema change) with per-parent compatibility declarations.
- **The compile principle extends** (ADR-0003): a piece's CI renders base form + piece (and declared piece combos) and compiles — same teeth as forms.
- Relationship to features: **feature = render-time toggle baked into the base tree; piece = post-creation unit with independent lifecycle.** The anti-explosion rule applies: something expressible as a feature must not become a piece.

Open questions for study VII: piece-vs-piece merge ordering and conflicts; sub-customization casings colliding with project identity; can a piece be *removed*; do common pieces need their own parent repo per ecosystem (a Room piece for android vs kmp) or one piece with per-target trees; CI cost of the combo matrix.

## Proposed order

| Phase | Content | Why this order |
|---|---|---|
| 1 | ✅ **A** (update in library + CLI) — engine v1.1.0 ships the `update` package and `templetry update [dir] [--apply]`; desktop v1.1.0 consumes it (−282 lines of duplicated cycle) | Foundation: unlocks F, feeds E's `update` tool, immediate standalone value |
| 2 | ✅ **E-lite** — `templetry-mcp` ships with engine v1.2.0 (stdio, dependency-free, five tools incl. `update`); AGENTS.md convention in the meta GUIDE (§8) and added to `android/single-module`. Pending from E: JSON Schema publication + llms.txt | Cheap, high leverage — rides the freshly-moved library APIs |
| 3 | ✅ **B** — engine v1.3.0: YAML/TOML patches (extension-dispatched, deterministic re-serialization), requires/conflicts enforced on resolved states, presets with defaults → preset → explicit order (`--preset` in CLI, arg in MCP, selector in desktop v1.2.0) | Small additive wins; presets become the pieces-selection UX |
| 4 | **F study VII → pieces v1** (template-local), then common pieces | The headline 1.x feature, entered with its prerequisites in place |
| 5 | ✅ **C** — `web/react-spa` (Vite + React + strict TS): first form with presets, first with a declared `verify:`, HTML-comment directives switching the entry point; validated locally (npm builds + browser run) and green in CI on both presets. Catalog: 4 parents / 8 forms. Lane order changed by owner: **F (pieces) moves last, D next** | Content lane — can run in parallel with any phase |
| 6 | 🔶 **D** — licences across the org, scoop bucket, multi-OS desktop builds and multi-forge (BYOR + GitLab/Gitea accounts) shipped; signing, winget and Homebrew pending | Continuous; signing when the certificate decision is made |

## Status (2026-08-14)

**A, E, B, C shipped · D partially · F v1 shipped.** Beyond the original lanes, the catalog grew to 21 forms / 12 pieces across eleven ecosystems, and pieces became an industrial catalog of their own ([study VIII](industrial-pieces-v1.md)). What remains is listed in the backlog of [state-of-the-art.md](../state-of-the-art.md).

Registry/answers changes in B and F are **additive** (new optional fields) — no schema break, per the compatibility policy.
