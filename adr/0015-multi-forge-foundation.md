# ADR 0015 — Multi-forge foundation: BYOR and source schemes

**Status:** ✅ Accepted · **Date:** 2026-08-13 · Revives [study V](../study/multi-forge-hosting-v1.md), refines [ADR-0009](0009-multi-forge.md)

## Context

ADR-0009 accepted multi-forge with a universal fallback — "bring your own remote" (BYOR): the user creates an empty repo anywhere and pastes its URL; the app only pushes. Study V was archived on the grounds that BYOR covered exotic hosts.

**Audit finding (2026-08-13): BYOR was never implemented.** The desktop offers a GitHub owner or local-only; there is no place to paste a remote. The v1 claim "BYOR covers every host" describes a design, not shipped code. Meanwhile the engine's template fetching is hardwired to `codeload.github.com`, so third-party catalogs cannot live on GitLab or Gitea either.

## Decision

Split multi-forge into its two independent axes and build the foundation of both, without forge adapters yet:

1. **Read axis — source schemes in the engine.** `source.Fetch(ref, subdir, token)` dispatches on a scheme prefix: `github:owner/repo` (bare refs default to it, preserving every existing answers file and registry), `gitlab:host/group/project`, `gitea:host/owner/repo` (one adapter serves Gitea, Forgejo, Codeberg and self-hosted instances). Each backend only builds a tarball URL and resolves a ref to a commit; `FromTarGz` is already generic. Registry gains an optional per-parent `source` field (schema v2.1, additive).
2. **Write axis — BYOR, implemented.** The desktop's owner picker gains a "paste a remote URL" option: the app renders locally, initializes the repo, commits and pushes to the given URL. No API, no adapter, works against every git host on earth.

Forge adapters (GitLab, Gitea) stay deferred: with BYOR real, an adapter buys convenience (creating the repo for you, listing your groups), not capability. They land when a user asks — the same anti-maintenance-pit rule ADR-0009 set.

> **Amendment (2026-08-13, same day): the owner asked, so the adapters shipped** in desktop v1.5.0 — GitLab and Gitea/Forgejo, with `whoami`, owners, create-repo and list-repos each. Authentication is a **personal access token per host**, not OAuth: GitLab and Gitea device flows require registering an application on every instance, which self-hosted servers make impossible to ship centrally, whereas a PAT works on gitlab.com, Codeberg and a company server alike. GitHub keeps its OAuth device flow. Tokens live in the OS keyring under `<scheme>@<host>`; the settings file stores only the account list, so exports stay shareable (the ADR-0009 rule that login and forge credentials are separate concerns).

> **Amendment (2026-08-22): the reason above was too broad, and gitlab.com gets OAuth.** "Device flows require registering an application on every instance" is true of a self-hosted server and it is why a personal access token stays the fallback — but **gitlab.com is one instance**. Templetry can register an application there once and ship its public application id, exactly as it does for github.com. Checked against the current documentation rather than restated from the original decision:
>
> | Forge | Device flow | A shipped application id works |
> |---|---|---|
> | github.com | yes | yes |
> | **gitlab.com** | yes, [GitLab 17.1+](https://docs.gitlab.com/api/oauth2) | **yes** |
> | self-hosted GitLab | yes, if 17.1+ | no — one application per instance |
> | Forgejo / Codeberg | yes, recent | no — its pre-registered applications are credential helpers, not ours |
> | Gitea | **no** — [authorization code grant only](https://docs.gitea.com/development/oauth2-provider/) | not applicable |
>
> So Gitea genuinely justifies a token; gitlab.com did not. A host may also carry its **own** application id in settings, which is how [`glab`](https://docs.gitlab.com/cli/authentication/) handles self-hosted servers and what makes one mechanism cover all three situations.
>
> The cost is not the flow — it is that **GitLab access tokens expire in two hours** and their refresh token invalidates both when used, unlike GitHub's, which do not expire. That needed a credential layer: the keyring entry became either a bare token, which is what a PAT has always been and stays, or a JSON grant carrying access, refresh and expiry. Every forge call and every git push resolves through it, renewing with a two-minute margin — a clone can outlive the seconds left on a token, and a refresh halfway through is a failure the user cannot act on.
>
> A personal access token remains supported everywhere and stays the only option for Gitea. Nothing about an existing account changes.

## Consequences

- The multi-forge claim becomes true in code, not just in an ADR.
- Templates and catalogs can be hosted outside GitHub; the answers file records the full scheme, so the update cycle follows the template wherever it lives.
- All additive: bare refs keep working, so every existing project keeps updating unchanged.
- Auth stays per-host (tokens passed in by the caller); the desktop keeps GitHub OAuth for identity and will add per-host tokens when adapters arrive.
