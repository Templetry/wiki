# Templetry — State of the art

**Snapshot: 2026-08-24** · the single up-to-date picture of what exists, where it lives and what state it is in. History lives in [`journal/`](journal/); decisions in [`adr/`](adr/); research in [`study/`](study/).

## What Templetry is

Templetry generates ready-to-work repositories from a library of **compilable** multi-platform templates, creates them on any git host, and keeps them alive afterwards: generated projects can pull template improvements and adopt new **pieces** long after creation.

The engine is a pure Go library plus a CLI and an MCP server; the desktop app embeds that same library behind a native UI. All framework knowledge lives in each template's `template.yml` — the engine has only generic operations ([ADR-0002](adr/0002-knowledge-lives-in-the-manifest.md), [ADR-0003](adr/0003-templates-compile.md)).

## Released components

| Repo | Role | State |
|---|---|---|
| [engine](https://github.com/Templetry/engine) | Go library + `templetry` CLI + `templetry-mcp` server | **v1.10.1** — binaries for linux/darwin/windows (amd64 + arm64) with SHA256SUMS |
| [desktop](https://github.com/Templetry/desktop) | Native app (Wails: Go backend embedding the engine, React/TS frontend) | **v1.11.0** — Windows (installer + portable), Linux, macOS universal |
| [catalog](https://github.com/Templetry/catalog) | Default registry (`registry.json`, schema v2) | **28 forms across 11 parents**; **12 form pieces + 3 common pieces** |
| [homebrew-tap](https://github.com/Templetry/homebrew-tap) | `brew install Templetry/tap/templetry` (CLI + MCP) — formula generated from the release, installed and tested on macOS and Linux CI | live |
| [scoop-bucket](https://github.com/Templetry/scoop-bucket) | `scoop install templetry` (CLI + MCP), autoupdating | live |
| [pieces](https://github.com/Templetry/pieces) | Common pieces adoptable by any compatible project (ADR-0016) | live |
| [renovate-config](https://github.com/Templetry/renovate-config) | Shared Renovate preset — one dependency policy for every repo | live |
| [wiki](https://github.com/Templetry/wiki) | Studies, ADRs, specs, journal, brand | living |
| [.github](https://github.com/Templetry/.github) | Organization profile + the shared `setup-templetry` action | living |

Licences: engine under **Apache License 2.0**; every other repo **MIT**. The engine was relicensed from PolyForm Noncommercial on 2026-08-19: a noncommercial engine barred the people the project needs — someone using it at work, and any contributor who would then be unable to use what they built.

**Every packaging path follows the engine on its own.** The scoop manifest and the Homebrew formula are generated from a release's own `SHA256SUMS`; the desktop, which compiles the engine in rather than calling it, cuts a minor of itself when the engine moves — taking the new version, proving the app still builds and its tests still pass, and only then tagging. Its release workflow is callable rather than duplicated, because a tag pushed with the default token does not start another workflow.

**Every repository now carries CI.** The catalog validates `registry.json` by fetching all 26 forms and 2 pieces from the repo and ref each declares; the pieces repo applies each common piece to a real target project through the registry pinned at the commit under test; the scoop bucket tracks engine releases on a schedule and checks that every URL it publishes resolves; the desktop runs its Go and frontend checks on every push rather than at release time. CodeQL runs on the engine and the desktop, and secret scanning with push protection is on across the organization.

## The catalog

Eleven parents, every form a real project whose CI renders **and compiles** it on each push.

Two forms added on 2026-08-24, both because a real project needed something the catalog could not generate:

- **kmp/library** — every kmp form was an application. There was no way to generate the thing those applications depend on: a shared multiplatform library, published to Maven, with no UI. All five verify combinations are green, iOS included.
- **web/react-router-ssr** — all four web forms rendered in the browser. Nothing generated a server-rendered app, which is what you need the moment the pages are meant to be found rather than logged into. Its CI job is separate from the shared matrix on purpose: the exit code is not what matters, because removing `main` from wrangler.jsonc makes the build go green and produce no server bundle at all. CI asserts `build/server/index.js` exists. `status: ready` in the registry is gated on green CI.

| Parent | Forms | Pieces |
|---|---|---|
| [kmp](https://github.com/Templetry/kmp) | `modular-features`, `single-module`, `modular-ui`, `library` | — |
| [android](https://github.com/Templetry/android) | `modular-features`, `single-module` | — |
| [ios](https://github.com/Templetry/ios) | `swiftui-app` | — |
| [web](https://github.com/Templetry/web) | `react-spa`, `vue-spa`, `nextjs`, `svelte-spa`, `react-router-ssr` | `axios-api`, `zustand-store`, `pinia-store`, `zod-env` |
| [go](https://github.com/Templetry/go) | `cli`, `http-service`, `rest-sqlite` | `version-endpoint`, `crud-resource` |
| [python](https://github.com/Templetry/python) | `fastapi-service`, `cli-typer`, `fastapi-users` | `rbac`, `api-keys`, `audit-trail`, `soft-delete`, `verifactu`, `crud-resource` |
| [rust](https://github.com/Templetry/rust) | `cli`, `axum-service` | — |
| [node](https://github.com/Templetry/node) | `express-api`, `fastify-api`, `nestjs` | — |
| [jvm](https://github.com/Templetry/jvm) | `spring-boot`, `ktor` | — |
| [dotnet](https://github.com/Templetry/dotnet) | `minimal-api`, `razor-web` | — |
| [meta](https://github.com/Templetry/meta) | `template` (the template that creates templates) | — |

**Twelve ecosystems, one engine, one manifest schema** — Kotlin Multiplatform, Android, Swift/SwiftUI, React, Vue, Next.js, Svelte, Go, Node/TypeScript, Python, Rust, JVM (Spring and Ktor), .NET. None of them required an engine change: the strongest evidence for the agnostic-engine thesis (ADR-0002).

### Environment profiles ([ADR-0018](adr/0018-environment-profiles.md))

**All 26 forms** ship `development` / `staging` / `production`, each in its own ecosystem's mechanism rather than a Templetry-shaped one — the same rule that keeps framework knowledge out of the engine (ADR-0002), applied to the catalog.

| Mechanism | Forms |
|---|---|
| `appsettings.{Environment}.json` | dotnet ×2 |
| `application-{profile}.yml` (Spring profiles) | jvm/spring-boot |
| Vite `.env.{mode}` | web ×3 |
| `.env.{profile}` inlined by `next.config.ts` | web/nextjs |
| `@nestjs/config` with a validate hook | node/nestjs |
| `.env.<profile>` + one validated accessor | python ×3, node ×2, go ×3, rust ×2, jvm/ktor |
| Android product flavors + `buildConfigField` | android ×2 |
| Xcode build configurations + compilation conditions | ios/swiftui-app |
| Gradle task generating one object into `commonMain` | kmp ×3 |
| Documented convention, not an implementation | meta/template |

A profile counts as done only when the three sources differ in something observable, the app reads the active one through a single validated accessor, and a test asserts that loading a named profile yields its values. Two platforms could not meet the third condition as written and say so instead of faking it: **iOS** makes the selection a compile-time fact and turns a missing flag into `#error`, and **Android** verifies each flavor's generated `BuildConfig` in CI, because per-variant codegen makes a plain unit test awkward across flavors.

KMP was the one ecosystem where both obvious mechanisms fail — `buildConfigField` does not reach iOS, Desktop or Web, and `expect`/`actual` would put the actuals in source sets that are themselves feature-gated, landing the profile in the intersection of two features. Generating into `commonMain` avoids per-platform files entirely.

## Shipped capabilities

### Engine + CLI (v1.10.1)

- **Render**: manifest parsing/validation, derived casings, feature exclusion, identity renaming, comment directives, structured patches for `.json`/`.yml`/`.toml`, deterministic output (same inputs → byte-identical result).
- **`requires`/`conflicts` between features and `presets`** (named feature combos, resolved as defaults → preset → explicit).
- **Remote catalogs and multi-forge hosting**: `github:`, `gitlab:` and `gitea:` source schemes (one adapter covers Gitea, Forgejo, Codeberg and self-hosted), with legacy bare refs untouched ([ADR-0015](adr/0015-multi-forge-foundation.md)).
- **`templetry update`**: re-render at the template's head with the recorded inputs, diff against disk, three-way merge via `git merge-file`; adds and modifications only, never deletes.
- **`templetry verify`**: runs the manifest's `verify: {image, run}` in Docker (ADR-0004 realized). Its one blind spot is Apple platforms — an iOS build needs macOS with Xcode, so `ios/swiftui-app` declares no verify block and says so instead of pretending; its parent's CI carries the guarantee on a macOS runner.
- **Pieces** (`templetry pieces` / `add`): decoupled units adopted after creation, with their own variables, identity map and drift anchor ([ADR-0014](adr/0014-lazy-pieces.md)). Applied pieces **ride the update cycle** — re-rendered at head and merged alongside the template, with the answers record rewritten rather than merged.
- **Common pieces** ([ADR-0016](adr/0016-common-pieces.md)): pieces living outside any form, in a shared repository listed in the registry, declaring `applies_to`. One name may have one implementation per ecosystem; each records its own source so a fix reaches every project that adopted it.
- **Hardened**: every write resolves its path against the output root and refuses escapes — `render.Contain`, applied by the render, piece and update writers alike; output paths cannot escape or abuse platform semantics (Windows device names, case-insensitive collisions, trailing dot/space); directive scanner fuzz-tested; manifests tolerate a UTF-8 BOM.
- **`templetry-mcp`**: dependency-free MCP server exposing `list_templates`, `get_form_schema`, `plan`, `render`, `update`, `list_pieces`, `add_piece`.

### Desktop (v1.11.0)

Three sections:

- **Build** — browse the catalog as a **collapsible tree** (catalogs, then parents, then forms, in the order each catalog declares) filtered **by taxonomy** (kind chips plus a search over every axis, ADR-0017), pick a form, fill the manifest-driven dynamic form (preset selector, and `requires`/`conflicts` surfaced instead of failing at render), preview it with **syntax colouring by extension** across forty-one languages, **verify** it (the render is built inside the manifest's container, log streaming), then create the repo and push. Templates are read through the catalog's source scheme, so forms hosted on GitLab or Gitea work like GitHub ones, with the token of the account on that host for private ones.
- **Cloud** — repositories across every signed-in account: GitHub and **gitlab.com** through the OAuth device flow, with no token to paste; Gitea/Forgejo and self-hosted servers with a personal access token in the OS keyring. An account that contributes nothing to the list says which one and why, instead of quietly shortening it. Every action works on every forge — local clones cross-linked, cloning authenticates as the owning account, and each repo opens a state preview **inside its own card**: languages, branches, latest CI runs or pipelines, README and docs rendered as sanitized markdown. A clone lands under the account it came from unless you ask for a flat folder. A repo carrying template manifests has them **read**, so each detected form reports its name, description and taxonomy rather than only that it exists.
- **Local** — recursive scan of the repositories folder, **grouped by the account that owns each repository** with its avatar, and an explicit bucket for those with no remote to read one from; per-repo preview inside its own card, with branches, remotes, last commit and working-tree state; drift detection on the template's anchor **and every applied piece's own**, the assisted update cycle, and a **pieces panel** listing form-local and common pieces alike.

Six panes can be dragged to size and remember their own width: the rendered-project preview, both document readers, the update diff, the sidebar and the build form column. Settings shows one section at a time, chosen from the sidebar.

Plus **BYOR**: paste any empty repository's URL and Templetry renders, commits and pushes — every git host on earth, no adapter needed. Ships for Windows, Linux and macOS; the in-app updater is Windows-only for now.

### Pieces: the industrial catalog

Researched in [study VIII](study/industrial-pieces-v1.md) with one rule — **encode a standard, not an opinion**:

| Piece | Encodes |
|---|---|
| `rbac` | NIST RBAC (ANSI/INCITS 359-2004): users → roles → permissions `(object, operation)`, with route guards |
| `api-keys` | Hashed keys with indexed prefix lookup, scopes, expiry, revocation, plaintext shown once |
| `verifactu` 🇪🇸 | Spain's RD 1007/2023: SHA-256 fingerprint chain, append-only registry, event log, QR payload ([study IX](study/verifactu-v1.md)) |
| `audit-trail` | Append-only who/what/when/where, with no write routes at all |
| `soft-delete` | `deleted_at` with opt-out filtering, restore, explicit purge |
| `crud-resource` | A whole entity (model, repository/router, tests) renamed to your object |

Pieces wire themselves through **sockets** — a registration point the piece plugs into from a file it owns — so adopting one never edits an existing file. Proven in Go (`api.Register`, `store.Register`) and Python (routers package walk).

## Decisions in force

Nineteen ADRs, indexed in [`adr/README.md`](adr/README.md). Load-bearing ones: own engine in Go (0001, 0006), agnostic engine (0002), templates compile (0003), verify in containers (0004), updates prepared from day one (0005), minimal directive grammar (0007), multi-forge (0009 + 0015), parents/forms/features (0011), desktop with Wails (0012 ⊃ 0010), **v1.0.0 declared** (0013), **lazy pieces** (0014), **common pieces** (0016), **the taxonomy** (0017), **environment profiles** (0018), **Apache-2.0 for the engine** (0019).

Discarded along the way: the hosted web app (superseded by the desktop).

## Specs

Normative documents in [`spec/`](spec/): [`template-yml.md`](spec/template-yml.md) (the manifest), [`directives.md`](spec/directives.md), [`answers-file.md`](spec/answers-file.md), [`piece-yml.md`](spec/piece-yml.md), [`validation-manifests.md`](spec/validation-manifests.md), and [`compatibility.md`](spec/compatibility.md) — **in force since v1.0.0**: the manifest, directives, answers file, registry v2, CLI surface and exported Go API only break with a major.

## Keeping the catalog alive

Two mechanisms, both added 2026-08-15 after [study X](study/where-next-v1.md) found that a green badge only meant "green when last pushed":

- **Weekly scheduled CI** in all ten parents (plus a manual trigger), so upstream drift surfaces here rather than in a user's project.
- **One engine version, in one file.** Every parent installs the CLI through [`Templetry/.github/actions/setup-templetry`](https://github.com/Templetry/.github/tree/main/actions/setup-templetry), whose `action.yml` holds the pinned release — and Renovate follows that line against the engine's releases, so the bump arrives as a PR the parents' own CI judges. The first version of this was "bump ten repositories by hand", which drifted by two releases within a day.
- **Renovate** on every repository that carries dependencies, extending one shared preset ([renovate-config](https://github.com/Templetry/renovate-config)). The loop it closes: Renovate proposes a bump inside a form → Verify renders that form and builds it → green means the upgrade is safe for everything generated from it. Generated projects get the same policy as an adoptable piece (`templetry add renovate`).

## Documentation

- [**`guide/`**](guide/) — the usage documentation: [getting started](guide/getting-started.md), [using templates](guide/using-templates.md), [using pieces](guide/using-pieces.md), [keeping projects updated](guide/keeping-updated.md), [the desktop app](guide/desktop.md), [authoring templates](guide/authoring-templates.md), [authoring pieces](guide/authoring-pieces.md), [AI agents and MCP](guide/ai-agents.md), [multi-forge](guide/multi-forge.md).
- Each repository's README covers *that* repository; the wiki covers the product. Version numbers live **here** and in the org profile — nowhere else, so there is one place to update.

## Where next

The plan in [study X](study/where-next-v1.md) has largely been executed: the promises made at v1.0.0 are kept (pieces ride the update cycle, the desktop has its pieces panel), the rot is stopped (weekly scheduled CI in all ten parents, Renovate everywhere, and the engine pin centralized in one file after the hand-managed version drifted), and the catalog has been expanded and industrialized.

What remains is deliberately **not** adoption work: certification and code signing are out of scope while Templetry is a self-use scaffolding project. The direction is more catalog, more pieces, and the engine gaps the pieces themselves surfaced.

## Open backlog

- **Pieces** (study VIII): `scim-provisioning` (RFC 7643), `oidc-login`, `multi-tenancy`, `outbox`, `rate-limit`, integrations (`stripe-billing`, `s3-storage`), domain CRUDs.
- **Engine**: `requires`/`conflicts` **between pieces**, piece **removal** (the answers file already records the owned files), and a jurisdiction/compat axis for pieces like `verifactu` — all surfaced by study VIII.
- **Desktop**: settings sync, in-app update on macOS/Linux. The GitLab and Gitea read paths are now checked against gitlab.com and codeberg.org by live tests (`go test -tags liveapi`), which caught a real truncation bug on the first run; still unexercised are the **authenticated** paths — private repositories, and creating and pushing to a repo on those forges.
- **Distribution**: Homebrew tap. (winget and code signing: deliberately deferred.)
- **Dependency automation**: the Renovate GitHub App still has to be installed on the org for the configs to do anything — the one item that needs a human click.
- **Forge adapters**: Bitbucket if anyone asks; BYOR already covers it.
