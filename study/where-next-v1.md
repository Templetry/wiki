# Study X — Where next

**Date:** 2026-08 · **Status:** study — a proposed plan, for the owner to accept, reorder or reject

## Honest diagnosis

Templetry is **technically ahead of its own usage**. In two weeks it went from an idea to a v1 with a written compatibility policy, 21 CI-verified forms across eleven ecosystems, 12 pieces, three forge integrations and an MCP server. That is a lot of *capability*.

What has not grown at the same pace:

1. **Usage.** The catalog is designed for people who will adopt it, but no project outside this org has been generated from it. [Study VIII](industrial-pieces-v1.md)'s own criterion — "a piece earns its place when it is rewritten badly in every project" — is currently applied from intuition, not evidence. The [v1.0.0 soak window was waived](road-to-v1.md); that debt is unpaid.
2. **Reach of the newest capability.** Pieces are the most differentiating thing the product does, and they are **CLI-only**. The desktop app cannot list or add one.
3. **Maintenance surface.** Thirteen repositories and ~30 workflows, maintained by one person. This is now a standing cost that competes with new features.

### Two findings from auditing the catalog (2026-08-14)

- **Nothing is verified on a schedule.** A form's CI runs only when its repo is pushed. A green badge therefore means *"this worked the day it was last touched"*, not *"this works"*. Templates rot silently as their upstream dependencies move — and a template with stale or broken dependencies is worse than no template, because it fails in the user's project, not in ours.
- **Parents verify against different, sometimes ancient CLIs.** `kmp` and `android` still render with engine **v0.1.0** — seven minors behind. Their green CI proves compatibility with a superseded engine, not with the one a user installs today. `web`, `rust`, `node`, `jvm` and `dotnet` sit at v1.4.0; only `go` and `python` are current.

Neither is a crisis today. Both are exactly the kind of quiet decay that makes a template catalog untrustworthy in six months.

## The plan

### Phase 1 — Keep the promises already made *(days)*

Things shipped code implies but does not yet do:

- **Piece updates.** The answers file records each piece's source and commit — a drift anchor — but `templetry update` ignores them entirely. A project can update its template and never its pieces. Either implement it or say so in the spec; leaving it implied is the worst option.
- **Pieces in the desktop.** A panel in Local's project preview: list what the template offers, mark what is applied, add with the piece's own variables. The bindings already exist (`list_pieces`, `add_piece` are in the MCP server; the Go API is the same).

### Phase 2 — Stop the rot *(days, then permanent)*

- **Weekly scheduled CI in every parent.** `schedule: cron` on the Verify workflow, so upstream drift surfaces here rather than in a user's project. This is the single highest-value change in the whole plan: it converts "green when pushed" into "green today".
- **One pinned CLI version, centrally managed.** Bring every parent to the current release and keep them together — a repository variable or a tiny reusable workflow, so the ten parents stop drifting apart.
- **Dependency updates on the templates themselves** (Dependabot/Renovate), because a template's value decays with its dependency versions.

### Phase 3 — Pay the soak-window debt by dogfooding *(weeks, in the background)*

The honest way to validate v1 is to run real work through it. The owner already has candidates: the portfolio (`carreerV2`), the career API worker, future side projects.

- Regenerate or migrate at least one real project from a catalog form.
- Adopt pieces on it (`rbac`, `audit-trail`) and live with them.
- Run `templetry update` on it monthly and let the friction dictate the backlog.

**Rule for this phase: no new forms or pieces are added speculatively.** What dogfooding demands gets built; what it does not demand waits. This directly answers the diagnosis — it converts guesses into evidence.

### Phase 4 — Decide whether adoption is a goal *(owner's call)*

This is the fork in the road, and it should be a conscious choice rather than a drift:

**If Templetry stays a personal tool**, most of what follows is unnecessary. Skip to "steady state".

**If it is meant to have users**, then the blockers are not features:

- **Code signing.** SmartScreen on an unsigned installer stops non-technical users cold. ~10 €/month for Azure Trusted Signing; nothing else in this list matters if the app cannot be installed comfortably.
- **A landing page and docs site.** The wiki is excellent for *us* and unreadable as a first contact. One page answering "what is this, why not Copier/Cookiecutter, how do I start in 60 seconds".
- **Packaging**: winget (after signing) and a Homebrew tap.
- **A worked example**, end to end: "generate an API, adopt RBAC, update it three weeks later" — with output shown.

### Phase 5 — At most one ambitious bet *(months)*

Only after phases 1–3, and only one:

- **Common pieces** ([study VII](pieces-v1.md) phase 2). `audit-trail` is the same idea in every ecosystem; today it would be copied five times. Cross-template pieces with per-parent implementations and a compatibility axis. This is the natural next architectural step and the one that scales the catalog rather than multiplying it.
- **Piece dependencies** (`requires` between pieces) — smaller, already identified in study VIII, and a prerequisite if the piece catalog keeps growing.
- **Templetry as a CI action** — regenerate/verify templates from other people's pipelines. Cheap to build, unclear demand.

## What to stop doing

- **Adding forms and pieces speculatively.** The catalog covers eleven ecosystems with zero external users; breadth is no longer the constraint. Depth of use is.
- **Treating green CI as proof of health** until phase 2 lands.

## Steady state, once phases 1–3 land

A monthly rhythm rather than a project: watch the scheduled CI, apply dependency updates, run `templetry update` on the dogfooded projects, and add a piece only when a real project asked for it twice.

## Decision points for the owner

1. **Adoption: yes or no?** Everything in phase 4 hangs on it.
2. **Certificate budget** (~10 €/month) — only if the answer to (1) is yes.
3. **Which real project gets dogfooded first**, and whether regenerating it is acceptable.
4. **Which single bet** in phase 5, when the time comes.
