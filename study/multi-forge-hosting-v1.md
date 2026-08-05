# Study V — Hosting beyond GitHub: feasibility per forge

**Date:** August 2026 · **Status:** 📦 Archived — adapters discarded for the desktop era (capability without users; BYOR covers exotic hosts). This study is the paved road if real demand appears.

Two independent capabilities per forge: **fetching templates** (catalog sources) and **creating/pushing repos** (the app's create pipeline). `git push` is universal; only tarball download, repo-creation API and auth differ.

## Per-forge feasibility

| Capability | GitLab (SaaS/self-hosted) | Gitea / Forgejo / Codeberg | Bitbucket Cloud |
|---|---|---|---|
| Tarball of a ref | `GET /api/v4/projects/:id/repository/archive.tar.gz?sha=REF` | `GET /api/v1/repos/{owner}/{repo}/archive/{ref}.tar.gz` | `GET https://bitbucket.org/{ws}/{repo}/get/{ref}.tar.gz` |
| Create repo | `POST /api/v4/projects` (personal or group via `namespace_id`) | `POST /api/v1/user/repos` · `POST /api/v1/orgs/{org}/repos` — **API mirrors GitHub's** | `POST /2.0/repositories/{ws}/{slug}` |
| Desktop auth | **OAuth device flow supported** (self-managed apps too) | **Device flow supported** (Gitea ≥1.20 / Forgejo); PAT trivial fallback | No device flow — app passwords / OAuth web only |
| Effort estimate | Medium (ids vs owner/name paths; namespaces) | **Low** — near drop-in next to the GitHub code; one adapter covers three hosts + any self-hosted instance | Medium-high (different API shapes, weakest auth story) |

## What changes in Templetry

1. **Registry v2.1**: form/parent sources need a scheme — `github:owner/repo`, `gitea:https://host/owner/repo`, `gitlab:https://host/id-or-path` — with bare `owner/repo` defaulting to GitHub (backwards compatible).
2. **Engine `source`**: `FetchTarball(sourceRef, ref, subdir)` dispatching per scheme — each ~30 lines given `FromTarGz` is already generic.
3. **App auth**: per-forge credentials in the keyring (`Templetry:gitea:<host>` etc.); ADR-0009's principle stands — GitHub login identifies you in the app, each forge gets its own token (PAT first, device flow where supported).
4. **Create pipeline**: owner picker grows a forge dimension; `runGit` already works against any HTTPS remote with the env credential helper.
5. **BYOR fallback** (already designed): any git host works today by hand-creating an empty repo — adapters are convenience.

## Verdict and order

Fully feasible; no architectural changes — ADR-0009's minimal-adapter design absorbs it as-is. Recommended order: **Gitea/Forgejo first** (lowest cost, three hosts + self-hosting — including a test instance on the owner's VPS), **GitLab second** (largest user base), Bitbucket only on demand.
