# Guides

How to *use* Templetry. The rest of this wiki explains why it is the way it is — [`adr/`](../adr/) for decisions, [`study/`](../study/) for research, [`spec/`](../spec/) for the normative rules. These pages are the practical path.

## Which guide do I need?

| I want to… | Read |
|---|---|
| Install it and create my first project | [Getting started](getting-started.md) |
| Use the catalog well: features, presets, verify | [Using templates](using-templates.md) |
| Add capabilities to a project I already generated | [Using pieces](using-pieces.md) |
| Keep a generated project current as its template improves | [Keeping projects updated](keeping-updated.md) |
| Work from the app instead of the terminal | [The desktop app](desktop.md) |
| Create a template of my own | [Authoring templates](authoring-templates.md) |
| Package something reusable as a piece | [Authoring pieces](authoring-pieces.md) |
| Drive Templetry from an AI agent | [AI agents and MCP](ai-agents.md) |
| Use GitLab, Gitea/Forgejo or a self-hosted server | [Multi-forge](multi-forge.md) |

## The five-minute mental model

**Parent → form → feature → piece.**

- A **parent** is a repository per concept (`web`, `python`, `go`).
- A **form** is a structural variant inside it (`react-spa`, `fastapi-users`). You *choose* one; forms are not combined.
- A **feature** is a toggle resolved when the project is generated (`router`, `docker`). Freely combinable, and `presets` name useful combinations.
- A **piece** is a decoupled unit you adopt *after* creation (`rbac`, `audit-trail`, `renovate`), with its own variables and its own update cycle.

Two properties make the rest make sense:

1. **Templates are real projects.** Every form compiles on its own and its CI builds the *rendered output* — so a broken template fails in the catalog, not in your project.
2. **Generated projects stay connected.** Each one carries a `.templetry-answers.yml` recording the template, the exact commit and your inputs, which is what lets you pull improvements later through a real three-way merge.
