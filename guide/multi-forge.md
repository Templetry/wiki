# Multi-forge

Templetry is not a GitHub product. "Multi-forge" covers two independent axes, and they are worth separating because they solve different problems:

- **Read** — where a *template* lives, and where the update cycle fetches it from.
- **Write** — where a *new project* is pushed.

Rationale: [ADR-0009](../adr/0009-multi-forge.md), [ADR-0015](../adr/0015-multi-forge-foundation.md).

## Read: hosting templates anywhere

A registry entry may declare a `source` scheme (schema v2.1, additive):

| Scheme | Reference form |
|---|---|
| `github:` | `github:owner/repo` — **the default** for bare references |
| `gitlab:` | `gitlab:host/group/project` (`gitlab:gitlab.com/acme/templates`) |
| `gitea:` | `gitea:host/owner/repo` — serves Gitea, Forgejo, Codeberg and self-hosted instances |

```json
{ "key": "acme", "label": "Acme", "repo": "gitlab.com/acme/templates",
  "source": "gitlab", "ref": "main", "forms": [ … ] }
```

Nothing changes in your commands. `templetry init acme/service --out ./svc` behaves identically whichever forge hosts it, because each backend does only two things: build a tarball URL and resolve a ref to a commit.

**Bare references still mean GitHub**, so every existing answers file and registry keeps working unchanged. And because the answers file records the *full* scheme, the update cycle follows the template wherever it lives — a project generated from a GitLab-hosted template updates from GitLab.

Private templates take a token:

```sh
TEMPLETRY_TOKEN=<token> templetry init acme/service --out ./svc --registry https://…/registry.json
```

## Write: getting a new project onto a host

The desktop's destination picker offers three kinds, in increasing order of convenience:

**1. Local only.** No remote. Always available.

**2. Bring your own remote (BYOR).** Create an empty repository yourself, paste its URL. The app renders, initializes, commits and pushes. No API, no adapter — this works against **every git host that exists**, including a bare repo on a server you own.

**3. A namespace from a signed-in account.** The app creates the repository for you and pushes. This is what an adapter buys: convenience, not capability.

## Accounts

Settings → Accounts, in the desktop app:

| Forge | Authentication |
|---|---|
| GitHub | OAuth **device flow** from the top bar — no password, no token to manage |
| GitLab | **Personal access token** per host (`gitlab.com` or your company server) |
| Gitea / Forgejo | **Personal access token** per host (`codeberg.org`, self-hosted) |

Each adapter supports `whoami`, listing your owners/groups, creating a repository and listing repositories — enough for the Cloud section to show every account's repositories side by side.

### Why tokens instead of OAuth for GitLab and Gitea

Their device flows require **registering an application on each instance**. For gitlab.com that would be one registration; for self-hosted servers it is one per company, which cannot be shipped centrally in a desktop app. A personal access token works on gitlab.com, Codeberg and a private server alike, from the first minute.

### Where credentials live

Tokens are validated against the host's API and then stored in the **OS keyring**, keyed by `<scheme>@<host>`. The settings file stores only the account list — no secrets — so **Settings → Export** produces a file you can share or move between machines safely.

## Scope

Adapters cover identity, namespaces and repository creation. Forge-specific features — merge requests, pipelines beyond reading status, issue trackers — are out of scope by design: Templetry scaffolds and updates projects, it is not a forge client.
