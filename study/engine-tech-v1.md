# Engine study II — Internal architecture & technology selection

**Date:** August 2026 · **Version:** 1.0 · **Status:** study for ADR-0006
**Scope:** the *shapes* (language-independent internal architecture) and the *technologies* (three candidate stacks with concrete libraries). Feeds ADR-0006.

---

## Part 1 — Shapes: internal architecture (language-independent)

These decisions hold regardless of the implementation language. They come from lessons in the state of the art (study I §3) and from Templetry's own requirements.

### 1.1 Module decomposition

```
templetry-engine
├── manifest/   # parse + validate template.yml, derive casings
├── source/     # obtain the template as a FileSet (local dir first; tarball later)
├── planner/    # (manifest, inputs, FileSet) → Plan          [pure]
├── ops/        # the operation types: data, not behavior
├── render/     # execute a Plan against a virtual file tree   [pure]
├── verify/     # run the manifest's verify command in Docker  [effectful]
└── cli/        # thin shell over the library
```

### 1.2 The five shape decisions

1. **Virtual file tree** (lesson from Nx generators). Every operation executes against an in-memory tree; disk is touched only at the very end. This buys three things at once: dry-run for free (render without the final write), trivial golden tests (compare trees, no temp dirs), and atomicity (no half-rendered output on failure).

2. **The Plan is data.** The planner produces a serializable list of operations (`CopyFile`, `RenamePath`, `FilterDirectives`, `ApplyPatch`, `WriteAnswers`). Being data, it can be printed (the dry-run UX from study I §7.4), diffed in tests, and logged. The renderer is a dumb executor.

3. **Pure core, effectful edges.** `planner/` and `render/` are pure functions over values — no filesystem, no network, no clock. Everything effectful (fetching, disk I/O, Docker) lives at the edges. This is what makes NFR4 (determinism) testable rather than aspirational.

4. **Two phases: plan, then apply.** Terraform's model, already implicit in study I's pipeline (fetch → plan → render → verify → deliver). The CLI exposes both: `templetry plan` (show me) and `templetry render` (do it).

5. **Structured errors with position.** The directive scanner reports `file:line` on every failure (unclosed `tpl:if`, unknown feature key). Errors are values with location, not strings — the web app will need to surface them in forms later.

### 1.3 Determinism rules (NFR4)

- Stable ordering: files processed in sorted path order, always.
- No timestamps, no randomness, no locale-dependent behavior in the core.
- Output normalized to LF (study I §6); `.gitattributes` in templates.
- Same (template commit, inputs) → byte-identical output tree. This is the golden-test contract.

### 1.4 Testing shape

- **Golden tests** as the backbone: fixture template + fixed inputs → expected output tree, versioned in the repo. Comparing virtual trees keeps them fast and diff-friendly.
- **Property tests** for casing derivation (round-trips, idempotence, collisions between casings).
- The **directive scanner** is the highest-risk unit (per-language comment styles, nesting, unclosed blocks) — it gets the densest test suite.
- **Fixture templates are miniatures**, not real projects: 5–10 files each, exercising one mechanism at a time (rename-only, directives-only, patches-only, all-combined).

---

## Part 2 — Technologies: the three candidate stacks

What the engine actually does, 80% of the time: parse YAML, walk file trees, match globs, scan comment lines, replace strings with casing awareness, apply JSON patches, unpack tarballs, shell out to Docker. The comparison below maps each need to concrete, maintained libraries.

### 2.1 Kotlin (JVM + GraalVM native image)

| Need | Library |
|---|---|
| CLI framework | Clikt |
| YAML | kaml (kotlinx.serialization) |
| JSON Patch RFC 6902 | zjsonpatch |
| Casing derivation | KaseChange |
| Globs | `java.nio` PathMatcher (or small custom matcher) |
| Tarballs | Apache Commons Compress |
| Docker | shell out to `docker run` (no SDK needed) |
| Manifest schema | kotlinx.serialization + hand validation, or everit json-schema |
| Testing | Kotest (data-driven; golden via file snapshots) |
| Distribution | GraalVM native image — mature in 2026, ~50–100 ms startups; JVM fallback |

**For:** the author's strongest language (velocity, joy, long-term maintenance). Coherent with the product's flagship templates (Android/KMP). The engine embeds directly into a future Ktor backend — same artifact serves CLI and web.
**Against:** the library ecosystem for *this niche* is Java-flavored and adequate rather than tailor-made. GraalVM adds a build-time toolchain (MSVC on Windows) and occasional reflection config — mitigated here because kotlinx.serialization is compile-time.

### 2.2 TypeScript (Bun runtime)

| Need | Library |
|---|---|
| CLI framework | citty or clipanion |
| YAML | `yaml` |
| JSON Patch RFC 6902 | fast-json-patch |
| Casing derivation | change-case (the reference implementation of this exact problem) |
| Globs | micromatch / Bun.Glob |
| Tarballs | `tar` |
| Docker | Bun.spawn / execa |
| Manifest schema | **zod** — one definition yields runtime validation + static types + JSON Schema for the web form |
| Testing | `bun test` — snapshot testing IS golden testing, native |
| Distribution | `bun build --compile` — production-ready in 2026, cross-compiles, ~60–90 MB binaries (Claude Code ships this way) |

**For:** the ecosystem is tailor-made for this niche — string/path/glob processing is JavaScript-tooling's home turf. Snapshot testing gives golden tests natively. The zod point is strategic: the manifest is Templetry's public API, and one zod schema becomes the single source of truth for engine validation *and* the web app's dynamic forms (phase 3), end-to-end typed. One language across the whole product.
**Against:** second language for the author (good but not native). Larger binaries (irrelevant in practice). Bun-on-Windows is solid now but younger than JVM/Go.

### 2.3 Go

| Need | Library |
|---|---|
| CLI framework | cobra |
| YAML | goccy/go-yaml |
| JSON Patch RFC 6902 | evanphx/json-patch |
| Casing derivation | iancoleman/strcase (fewer variants than change-case) |
| Globs | bmatcuk/doublestar |
| Tarballs | stdlib `archive/tar` (best-in-class) |
| Docker | `os/exec` |
| Manifest schema | santhosh-tekuri/jsonschema |
| Testing | stdlib + `testdata/` golden files (idiomatic Go pattern) |
| Distribution | unbeatable: small static binary, trivial cross-compile |

**For:** the strongest stdlib for tar/files, the leanest binaries, golden files as an idiomatic native pattern (the Go clones in study I §3 exist for a reason).
**Against:** the author's weakest language — highest friction for a solo project whose success depends on sustained personal velocity. Zero code sharing with the web phase.

---

## Part 3 — The 2026 finding: distribution no longer discriminates

The ADR-0006 table gave Go a distribution edge. Verified in August 2026, that edge has largely evaporated:

- **GraalVM native image** is production-mature for CLI tools: millisecond startups, no JVM at runtime ([overview](https://oneuptime.com/blog/post/2026-02-20-java-graalvm-native-image/view), [reference](https://www.graalvm.org/latest/reference-manual/native-image/)).
- **`bun build --compile`** is production-ready, cross-compiles, and is how Claude Code ships to millions of developers ([docs](https://bun.com/docs/bundler/executables), [write-up](https://medium.com/@reactjsbd/from-npm-to-a-single-binary-compiling-your-node-js-cli-with-bun-3dcb69e6d35a)).

All three stacks deliver a standalone binary. The decision therefore hinges on three factors only:

1. **Author affinity** — solo-project velocity and joy (favors Kotlin).
2. **Niche ecosystem fit** — libraries for string/glob/casing/snapshot work (favors TypeScript).
3. **Reuse with phase 3** — sharing the manifest schema and engine with the web app (favors TypeScript strongly; Kotlin partially via Ktor).

## Part 4 — Analysis and proposal

Go exits the race: its historical advantage was distribution, now neutralized, and it scores lowest on affinity and reuse. The real choice is Kotlin vs TypeScript:

- Choose **Kotlin** to optimize for the author: mastery, enjoyment, and a coherent identity with the Android/KMP templates that motivated the project.
- Choose **TypeScript + Bun** to optimize for the product: tailor-made libraries, golden tests as native snapshots, and — decisive — the zod manifest schema as a single source of truth reused by the phase-3 web forms. The manifest is the public API (ADR-0002); having exactly one definition of it across the entire product removes the most dangerous class of drift.

**Proposal: TypeScript on Bun**, with the engine as a pure library (`@templetry/engine`) plus a thin CLI (`templetry`), compiled with `bun build --compile` for distribution. Kotlin remains a fully defensible alternative if the affinity factor is weighted first — the shapes in Part 1 are identical either way, so no architectural work is lost if the choice is ever revisited.
