# Demo script

**Status:** production material for the website and the READMEs. Seven recordings, each with one job.

Every command here was checked against the CLI surface of engine v1.10.1. Where a demo needs something that does not exist yet, it says so instead of inventing a flag.

## Rules for all of them

**One idea per recording.** A demo that shows three things teaches none. If a beat does not serve the single takeaway, cut it.

**Nothing is faked.** No edited output, no retyped results, no cuts that hide a wait. If a step is slow, either show the wait or pick a faster example — the whole product thesis is that this is verified rather than claimed, and a doctored demo poisons that.

**Record it as a script, not as a performance.** Use [VHS](https://github.com/charmbracelet/vhs): a `.tape` file describes the keystrokes and timings, and `vhs demo.tape` renders a GIF. That makes a demo reproducible, reviewable in a diff, and re-recordable when the CLI changes — the same rule already applied to the scoop manifest and the Homebrew formula. Hand-recorded GIFs rot silently.

**Terminal setup.** 100×26, Ink & Brass palette (`#15171e` ground, `#eae7de` text, `#d9a441` for the prompt), Consolas or another face from the [guidelines](guidelines.md), 16 px. Prompt is a bare `$` — no username, no host, no git branch decoration.

**Never on screen:** tokens, real repository names that are not yours, `~/Documents/...` paths that leak a name, notification banners.

**Length.** The number given per demo is a ceiling, not a target. Under it is better.

---

## 1 · The day after · 25 s

**The one that matters.** It is the only demo no competitor can film, and it is the argument for the whole product.

**Takeaway:** *your project keeps improving after you generate it, and your own edits survive.*

| Beat | On screen |
|---|---|
| 1 | `templetry init go/http-service --out ./billing --set "project_name=Billing" --set "module_path=example.com/billing"` |
| 2 | `cd billing && go build ./...` — it builds. Two seconds, no commentary. |
| 3 | Edit something obviously *yours*: add a handler, or change a log line. Show the diff briefly. |
| 4 | `templetry update` — **preview is the default**, nothing is written yet. Show the list of what would change. |
| 5 | `templetry update --apply` |
| 6 | `git diff` — the template's improvement arrived **and** your handler is still there. |
| 7 | `go build ./...` — still builds. |

**The beat that lands is 6.** Hold on it. Everything before it is setup.

**Needs:** a template that has actually moved since the project was created. Until then, record it against a fork of a parent with one deliberate commit in between, and say so in the caption. This is the demo that most wants real dogfooding behind it.

---

## 2 · Templates that compile · 20 s

**Takeaway:** *this is not a skeleton with placeholders — the template itself is a real project.*

| Beat | On screen |
|---|---|
| 1 | Open a template file from the catalog on screen — `parents/go/http-service/internal/api/mux.go`. Let the viewer read it: **no `{{ }}` anywhere**. Ordinary Go. |
| 2 | `templetry init go/http-service --out ./demo --set "project_name=Demo" --set "module_path=example.com/demo"` |
| 3 | `go test ./...` — passes. |

**The beat that lands is 1**, and it only lands if the viewer knows what Cookiecutter and Copier template files look like. On the website, put this demo directly under the comparison table, where that contrast has just been made.

**Alternative for a viewer who does not know:** show the same file split against a Cookiecutter template of the same thing. Riskier and longer — keep it for a blog post, not the landing page.

---

## 3 · Adopt a piece · 20 s

**Takeaway:** *you do not have to decide about auth on day one.*

| Beat | On screen |
|---|---|
| 1 | An existing project, already generated. `templetry pieces` — the list of what this project can adopt. |
| 2 | `templetry add rbac` |
| 3 | `git status` — new files, and nothing of yours overwritten. |
| 4 | The build or the test suite passing. |

**Say out loud in the caption what beat 3 shows:** a piece may only add files that do not exist, and may only touch shared files through declared patches. That constraint is why adopting one is safe, and it is invisible unless named.

---

## 4 · Find the template you want · 15 s

**Takeaway:** *twenty-six templates, and you reach yours in one command.*

| Beat | On screen |
|---|---|
| 1 | `templetry list` — the whole catalog, briefly. Long enough to feel like a lot. |
| 2 | `templetry list --kind backend --language python` |
| 3 | `templetry list --framework react --tags` |

Better as a **desktop** recording than a terminal one: the kind chips filtering live are more legible than a list reprinting. If recording the app, use the same beats and let the chips do the talking.

---

## 5 · Any forge · 20 s

**Takeaway:** *not everyone lives on GitHub, and Templetry does not assume it.*

Desktop app. Create a repository on **GitLab or Codeberg**, not GitHub — the point evaporates if the demo shows the obvious one.

| Beat | On screen |
|---|---|
| 1 | The accounts panel with more than one forge configured. |
| 2 | Pick a template, pick the non-GitHub account, create. |
| 3 | The repository existing on that forge, with the project in it. |

**Blocked until the authenticated GitLab/Gitea paths have been exercised for real.** They are written against published APIs and have never run against a server with a token. Do not film a path nobody has walked — record this one after the first real run, not before.

---

## 6 · Built for agents · 25 s

**Takeaway:** *your agent can scaffold for you.* Uncontested ground — no comparable tool ships an MCP server.

| Beat | On screen |
|---|---|
| 1 | The MCP server configured in an agent client. One screenful of config, no more. |
| 2 | A plain-language request: *"start me a FastAPI service with user management, call it Billing"*. |
| 3 | The agent calling `list_templates`, then `get_form_schema`, then `render`. |
| 4 | The generated project, building. |

Beat 3 is what makes it real rather than a chatbot trick: the agent is **reading the form's schema** and answering its variables, not guessing at a shell command.

Keep the client's own branding incidental. The demo is about the server, and clients change.

---

## 7 · Sixty seconds, end to end · 60 s

The long one, for the README and for anyone who wants the whole shape. Demos 1, 3 and 4 stitched with title cards between them — not a re-recording. If a section needs different pacing to work here, fix it in the original and re-render both.

Only worth cutting **after** the short ones exist and have been watched by someone who is not you.

---

## Order to record them

1. **2 · Templates that compile** — needs nothing that does not exist. Do it first.
2. **4 · Find the template** — same.
3. **3 · Adopt a piece** — same.
4. **1 · The day after** — wants a real project with real history behind it.
5. **6 · Built for agents** — needs an agent client set up, no blockers.
6. **5 · Any forge** — blocked on the authenticated forge paths being exercised.
7. **7 · End to end** — last by construction.

Numbers 2, 4 and 3 could be recorded today. That is the honest starting point, and three good short demos beat one long one that waited for everything.

## Where they go

| Demo | Placement |
|---|---|
| 1 · The day after | Website hero, and the "day after" section |
| 2 · Templates that compile | Directly under the comparison table |
| 3 · Adopt a piece | The pieces section, and `pieces/README.md` |
| 4 · Find the template | The catalog section |
| 5 · Any forge | The multi-forge section |
| 6 · Built for agents | Its own section — the differentiator with no competition |
| 7 · End to end | `engine/README.md` and the organization profile |

Keep the `.tape` files in this folder next to this file, so the script and the recipe live together and neither drifts from the other.
