# Keeping projects updated

Scaffolding usually ends at creation: the template improves, your project does not. Templetry keeps the connection, because every generated project records what made it.

## The cycle

```sh
templetry update ./my-api            # preview: shows what would change
templetry update ./my-api --apply    # write it
```

What happens under the hood:

1. Read `.templetry-answers.yml`: which template, which commit, which inputs, which pieces.
2. Fetch the template at its **current head** and re-render it with **your recorded inputs**.
3. Fetch the template at the **recorded commit** and render that too — the "before" picture.
4. Compare all three: what the template used to produce, what it produces now, and what is on your disk.
5. Do the same for every applied piece, from its own source.

## How each file is decided

| Status | What it means |
|---|---|
| `added` | The template now produces a file your project does not have |
| `modified` | The template changed it and **you never touched it** — safe overwrite |
| `merged` | Both sides changed it, and the changes combined cleanly |
| `conflict` | Both sides changed the same lines — merged with conflict markers for you to resolve |

The third band (step 3 above) is what makes `modified` and `merged` distinguishable from each other. Without it, every difference would look like a conflict.

**Nothing is ever deleted.** Files you added, files the template dropped, files you renamed: all untouched. Applying an update only adds and rewrites.

## After applying

```sh
git diff
```

That is not a formality. A `merged` file combined two sets of changes automatically, and a `conflict` file contains markers you must resolve. Review, then commit — or `git checkout .` if you disagree with the result. The update never commits for you.

## What happens to the answers file

It is **not** merged like ordinary content: it is rewritten from the record, moving the template's and every piece's commit anchor forward. Your inputs, features and the list of applied pieces always survive an update.

## Detecting drift without updating

The desktop app checks for you: any project whose template has moved past its recorded commit shows a **template updated** chip in the Local section, with a preview before anything is written. See [the desktop guide](desktop.md).

## In CI

`update` is scriptable, so a scheduled workflow can open a pull request whenever the template moves:

```yaml
- run: templetry update . --apply
- uses: peter-evans/create-pull-request@v7
  with:
    title: "Template update"
    branch: templetry/update
```

Your project's own CI then decides whether the update is safe — which is the same loop the catalog uses on itself.

## When a project drifts far

If your project diverged heavily, expect conflicts. Two honest options:

- **Update anyway** and resolve them; the merge is per-file, so most of it lands cleanly.
- **Stop updating** and delete `.templetry-answers.yml`. The project keeps working; it simply stops being connected. That is a legitimate end state, not a failure.
