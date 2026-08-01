# Preliminary study — Repository generation engine from templates

**Project:** **Templetry** — a manager for architectures, projects and frameworks
**Date:** August 2026 · **Version:** 1.1 · **Status:** preliminary study, no implementation
**v1.1 changes:** multi-forge — target-repo creation must work on any open git host, not only GitHub (FR9, D9).

---

## 1. Goal and scope

Build the **engine** that generates a new repository's content from a template, within these scope limits:

- **In:** multi-platform template rendering (Android, KMP, web, backend, etc.), feature composition, output verification, local CLI.
- **Out (later phases):** web app, GitHub OAuth, repo creation via API, visual catalog. The engine must not depend on any of these.
- **Target user:** initially one (the author); the design stays open to third parties.

**Engine success metric:** from command to compilable project in < 30 seconds (excluding verify), for any registered template, with zero framework-specific code in the engine.

---

## 2. Requirements

### Functional
| # | Requirement |
|---|---|
| FR1 | Substitute the template's identity (packages, names, slugs) in file content **and paths** |
| FR2 | Automatically derive casing variants of every variable (Pascal, kebab, snake, flat, camel) |
| FR3 | Conditional inclusion/exclusion of files and blocks based on selected features |
| FR4 | Modify comment-less structured files (JSON) declaratively |
| FR5 | Copy binaries untouched, preserving execute permissions |
| FR6 | Dry-run: show the operation plan without executing anything |
| FR7 | Verify that the output compiles, without toolchains installed on the host |
| FR8 | Record in the output which template, version and values were used (answers file) |
| FR9 | Create the target repo on **any open git host** (GitHub, GitLab, Gitea/Forgejo, Bitbucket, self-hosted), not only GitHub. The app's login can remain GitHub OAuth |

### Non-functional
| # | Requirement |
|---|---|
| NFR1 | **Agnostic:** zero framework knowledge in engine code |
| NFR2 | **Templates compile:** every template repo is a valid project with its own CI |
| NFR3 | Engine = pure library (directory in → directory out) + CLI on top; no HTTP, no GitHub |
| NFR4 | Deterministic: same inputs → byte-identical output (the basis for golden tests) |
| NFR5 | Maintainable by one person: effort grows with templates, not with the engine |

---

## 3. State of the art

| Tool | School | What to learn from it | Why not adopt it |
|---|---|---|---|
| **Copier** | Textual (Jinja2, YAML) | Answers file + **three-way-merge updates** (the only one managing code lifecycle, not just scaffolding). Form rendering from a YAML questionnaire. | Placeholder templates don't compile; delimiter collisions with Gradle/Kotlin; the manifest (public API) would belong to someone else |
| **Cookiecutter** | Textual (Jinja2, JSON) | De facto standard; pre/post-gen hooks; 4000+ templates as a reference of real needs | No updates or migrations; same Jinja problems |
| **Yeoman / Plop / Hygen** | Programmatic (JS) | In-project generators (add a component to an existing repo) — possible future phase | Every template is code to maintain; scales poorly for one person |
| **Nx generators** | Programmatic (AST) | Virtual file tree (eases dry-run and tests); automated migrations | Coupled to its ecosystem; high complexity |
| **Projen** | Code-first | "Config is regenerated, never edited" philosophy — anti-drift | Ties templates to its runtime; different mental model |
| **Backstage scaffolder** | Action pipeline | Declarative manifest that renders forms; catalog/execution separation | Enterprise weight; needs a platform team |
| **scaffold / stencil / boilr** (Go) | Textual (Go templates) | Single-binary distribution as a CLI format | Conceptually Cookiecutter |
| **GitHub Templates** | Plain copy | The baseline to beat; the `generate` API as reference | No variables, no logic |

**Survey conclusion:** language coverage is not a differentiator (all textual engines are agnostic by construction). No tool combines *compilable templates + feature composition + manifest as API*. That gap justifies building our own engine.

---

## 4. Design space — decisions

### D1. Templating school — PROPOSED: hybrid
- **(a) Textual placeholders** (Jinja): maximum expressiveness; templates stop compiling, lose CI and IDE, collide with Gradle/Kotlin `${}`.
- **(b) Programmatic**: maximum power; every template is a program. Rejected per NFR5.
- **(c) Real project + structured rename**: robust and verifiable; no conditionals.
- **Proposal:** (c) + comment directives + structured patches. Rationale: NFR2 is worth more the more platforms exist — each template's CI verifies what the author can't hand-test across 15 ecosystems.

### D2. Where framework knowledge lives — PROPOSED: in the manifest
The engine exposes 5 generic operations; `template.yml` configures them. Supporting a framework = writing a template, never touching the engine. (The anti-decision of Yeoman/Backstage.)

### D3. The three transformation mechanisms — PROPOSED
1. **Identity-map renaming** (universal): the template declares its canonical strings (`com.template.base`, `template-app`, `TemplateApp`) and which variable+casing they map to.
2. **Comment directives** (`tpl:var`, `tpl:if` / `tpl:endif`): per-extension comment-style table (~20 entries: `// # <!-- --> /* */`). Unknown extension → copy without scanning (safe by default).
3. **Structured patches** for comment-less formats: JSON Patch **RFC 6902** (don't invent a format), extended to YAML/TOML with the same operation vocabulary.
- Documented escape hatch: `*.tpl` files rendered with full textual templating, losing the compile guarantee (exceptional path, not the main one).

### D4. Verification — PROPOSED: container declared in the manifest
`verify: {image, run}`. The engine only knows `docker run`. No toolchains on the host; each template's CI = a matrix over its main feature combinations.

### D5. Updates of generated projects — PROPOSED: prepare, don't implement
Write `.templetry-answers.yml` (template, version/commit, values) into every generated project from day 1 — zero cost, retroactively indispensable. Copier-style three-way merge stays for a future phase; it is the expensive piece (months).

### D6. Engine language — **OPEN**
| Criterion | Kotlin | TypeScript | Go |
|---|---|---|---|
| Author affinity | ★★★ | ★★ | ★ |
| Code sharing with the future web | via Ktor backend | ★★★ (same stack) | ✗ |
| CLI distribution | JVM or GraalVM native | requires Node | ★★★ single binary |
| Libraries (casings, JSON Patch, tar, YAML) | ★★ | ★★★ | ★★ |
| Risk/novelty | low | low | medium (new language) |

Note: the engine is a pure library (NFR3), so the language doesn't shape the architecture — it shapes distribution and reuse. Needed before the first prototype, not before the schema.

### D7. Exact directive grammar — **OPEN**
Minimum viable: `tpl:var <key>` (line) and `tpl:if <expr>` / `tpl:endif` (block). To decide: `tpl:else`? expressions (`and`/`or`/`not`) or feature keys only? line vs whole-file directives? Principle: the fewer, the better — every new directive is public API.

### D8. Project name — **RESOLVED: Templetry**
Chosen after verifying ~70 candidates across 6 rounds (August 2026). Derived from *templet*, the original spelling of "template" (and a real loom tool). Verified: GitHub (`templetry`, exact org) and npm free; no existing product with that name; enough distance from Templafy/Templify.
Consequences: CLI `templetry` (short alias `tpl`), answers file `.templetry-answers.yml`, directive prefix `tpl:` (brand-consistent). Fully-available rejected finalists, kept as plan B: Repojig, Forgehand.

### D9. Multi-forge (FR9) — PROPOSED: minimal adapters + capabilities + universal fallback
The engine is unaffected (NFR3); only the orchestrator. Problem split:
- `git push` is universal — works against any host without adaptation.
- The forge-specific parts are only **creating the repo via API** and **post-setup**.

Proposed design:
1. **Minimal adapter interface:** `createRepo(name, visibility) → clone URL`. Abstract no more API than that (anti maintenance-pit).
2. **Capability model:** each adapter declares which post-setup it supports (`topics`, `branch_protection`, `secrets`…); the app degrades gracefully, no lowest common denominator.
3. **Universal fallback — "bring your own remote":** the user hand-creates an empty repo on their host and pastes the URL; the app only pushes. Covers 100% of git hosts with zero adapters — adapters are convenience, not requirement.
4. **Authentication separate from login:** GitHub OAuth identifies the user in the app; creating repos on other forges requires a per-host token (PAT), stored encrypted. Gitea and Forgejo share a nearly identical API (one adapter serves both, plus Codeberg).

Implementation order: BYOR + GitHub adapter in the MVP; Gitea/Forgejo and GitLab later, based on real usage.

---

## 5. Manifest draft v1 (to be dry-run-validated)

```yaml
# template.yml — draft, NOT a spec
name: react-vite-ts
platform: web          # catalog tags; the engine ignores them
framework: react
variables:
  - key: project_name        # the engine derives kebab, Pascal, snake...
    type: string
identity:
  - from: "template-app"     # appears in package.json, index.html...
    to: "{project_name.kebab}"
  - from: "TemplateApp"      # root component
    to: "{project_name.pascal}"
features:
  - key: router
    files: ["src/routes/**"]
    patches:
      - file: package.json
        op: add
        path: /dependencies/react-router-dom
        value: "^7.0.0"
verify:
  image: node:22
  run: npm ci && npm run build
```

**Schema validation plan (§7):** write this same schema for three dissimilar templates — KMP (identity = JVM package + deep paths), React (JSON everywhere), Python/FastAPI (snake_case, pyproject.toml) — and check none needs hacks. If one does, the schema is wrong, not the template.

---

## 6. Edge-case catalog (design checklist)

- **Substring collisions** in renaming: `template-app` inside `template-application` or a README badge URL. → Word-boundary renaming or a per-file exclusion list.
- **Cross-casing collisions**: `templateapp` (flat) can appear accidentally in normal text. → Application order: longest strings first; review in dry-run.
- **CRLF vs LF** (the author develops on Windows): normalize output to LF + `.gitattributes` in templates; golden tests must be insensitive or normalize.
- **Execute permissions** (`gradlew`, scripts): tar preserves them, zip not always; they don't exist on Windows — declare executables in the manifest or preserve them from GitHub's tarball.
- **Binary detection**: heuristic (null bytes) + extension list; never scan directives in binaries.
- **Empty directories**: git doesn't track them — templates use `.gitkeep`; the engine must not break them.
- **BOM and encodings**: assume UTF-8; detect BOM and preserve it or fail with a clear error.
- **Deep paths when renaming packages** (`src/main/kotlin/com/template/base/…`): segment-by-segment directory renaming; beware the 260-char Windows path limit.
- **Lockfiles** (`package-lock.json`, `gradle.lockfile`): contain the project name → either regenerated in verify, or part of the identity map.
- **The tarball's `.git`**: doesn't exist (tarball advantage over clone) — the new repo is born clean.
- **Mutually exclusive or dependent features**: `requires`/`conflicts` in the manifest? Candidate for v1.1 — don't complicate v1.

---

## 7. Validation plan (no production code)

1. **Dry manifest validation** (D6 not needed): hand-write the 3 manifests of §5. Criterion: zero hacks.
2. **Golden tests** (once a prototype exists): fixed inputs → byte-exact expected output, versioned in the engine repo.
3. **Per-template CI**: the template compiles itself on every push + compiles the rendered output of the main feature combinations (matrix).
4. **Dry-run as a product**: the readable operation plan is both a UX feature and an engine debugging tool.

---

## 8. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| The manifest schema falls short and must break | High (it's the public API) | Dry validation (§7.1); `schema_version` field from v1 |
| Identity renaming produces false positives | Medium | Edge cases §6; dry-run with diff; golden tests |
| Scope creep toward Backstage | High | §1 scope; the web is phase 2; team/marketplace features out |
| Three-way-merge updates never arrive and generated projects are orphaned | Medium | `.templetry-answers.yml` from day 1 keeps the door open |
| Slow verify (Android builds take minutes) | Low | Verify optional/async interactively; mandatory only in CI |
| N forge adapters devour maintenance | Medium | Minimal interface (`createRepo` only) + capability model + BYOR fallback; new adapters only on real usage |

---

## 9. Proposed roadmap

- **Phase 0 — Study** *(this document)* → close D6 (language), D7 (grammar); dry-validate the manifest (3 templates).
- **Phase 1 — Engine**: library + CLI (`render`, `plan`); golden tests; migrate the existing Android or KMP template to the format.
- **Phase 2 — Verify + CI**: containers; templates with CI matrix; second and third real templates.
- **Phase 3 — Web MVP**: GitHub OAuth, catalog from `registry.json`, dynamic form from the manifest, create repo + push (GitHub adapter + BYOR fallback). Deploy on the VPS (Docker).
- **Phase 4+ (optional)**: GitLab and Gitea/Forgejo adapters, three-way-merge updates, in-project generators, third-party templates.

---

## 10. Open questions to close Phase 0

1. Engine language (D6) — needed before the prototype.
2. Minimal directive grammar (D7) — just `var` + `if/endif`?
3. ~~Project name (D8)~~ — **resolved: Templetry** (see D8).
4. Do v1 features support dependencies between each other (`requires`), or is that postponed to v1.1?
5. Does the manifest declare executables explicitly, or trust the tarball?
