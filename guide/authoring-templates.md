# Authoring templates

The fastest start is the meta-template, which scaffolds a template with its manifest, author guide and verify CI already in place:

```sh
templetry init meta/template --out ./my-template --set "project_name=My Template"
```

This guide explains what you are looking at, and how to grow it into a catalog form.

## The one rule everything follows

**Your template is a real project.** It compiles, its tests run, your IDE understands it. There are no `{{ placeholders }}` breaking the syntax — that is the whole point ([ADR-0003](../adr/0003-templates-compile.md)): a template you cannot build is a template whose defects you discover in someone else's project.

So: write the project you actually want, make it work, and *then* declare how it gets renamed.

## Step 1 — Choose a canonical identity

Pick concrete names and use them consistently:

| Kind | Example |
|---|---|
| Package / module | `com.template.base`, `example.com/template-app` |
| Class / type names | `TemplateApp` |
| Slug (package.json name, docker tag) | `template-app` |
| Bare token (plugin ids, storage dirs) | `templateapp` |

## Step 2 — Declare the identity map

```yaml
schema_version: 1
name: my-template          # kebab-case, matches the registry entry
description: What it scaffolds
platform: backend          # catalog tags; the engine ignores them
framework: fastapi

variables:
  - key: project_name
    label: Project name
    type: string
    pattern: "^[A-Za-z][A-Za-z0-9 ]*$"

identity:
  - from: "TemplateApp"
    to: "{project_name.pascal}"
  - from: "template-app"
    to: "{project_name.kebab}"
```

Every string variable yields casing variants automatically: `{k}`, `{k.pascal}`, `{k.camel}`, `{k.kebab}`, `{k.snake}`, `{k.flat}`. Replacements apply to **file contents and paths**, longest match first, and dotted identities also rename their slash form (`com.template.base` renames `com/template/base/` directories *and* the doc references to that path).

### Identity archaeology

The part that takes real work is finding **every form your canonical identity takes**. A KMP template needed three: the dotted package, a bare token used in plugin ids and storage directories, and the slash form inside docs. The loop that finds them fast:

```sh
templetry render --template . --out /tmp/out --set project_name="Verify App"
grep -rn "template-app\|TemplateApp\|templateapp" /tmp/out    # must print nothing
```

Zero leftovers is the bar. Anything that survives is an identity you forgot to declare.

## Step 3 — Add optional parts as features

```yaml
features:
  - key: docker
    label: Dockerfile
    default: true
    files: ["Dockerfile", ".dockerignore"]
  - key: analytics
    requires: [router]        # both must end up active
    conflicts: []             # mutually exclusive features

presets:
  - key: minimal
    label: Bare
    features: {docker: false, analytics: false}
```

A file is excluded when it matches the globs of an *inactive* feature and no active one. Globs match **template paths**, not renamed ones.

## Step 4 — Conditional lines and values: directives

Three directives, all inside ordinary comments so the file stays valid:

```kotlin
// tpl:if room
implementation(libs.room.runtime)
// tpl:endif

minSdk = 26 // tpl:var min_sdk 26
```

- `tpl:if <key>` / `tpl:endif` — include a block only when a feature is active. `!key` negates. They must be alone on their line.
- `tpl:var <key> <literal>` — replace `<literal>` on that line with the variable's value. The literal must actually appear on the line.

Files whose extension has no known comment style are copied without scanning — safe by default. See [the directive spec](../spec/directives.md) for the comment table.

## Step 5 — Comment-less formats: patches

JSON has no comments, so features edit structured files declaratively (RFC 6902), for `.json`, `.yml`/`.yaml` and `.toml`:

```yaml
features:
  - key: router
    patches:
      - file: package.json
        op: add                    # add | replace | remove
        path: /dependencies/react-router-dom
        value: "^7.0.0"
```

Patched files are re-serialized deterministically, which means **comments and hand formatting in the patched file are lost**. For formats that support comments, prefer directives; reserve patches for JSON and machine-managed data.

## Step 6 — Declare how it is verified

```yaml
verify:
  image: node:22
  run: npm install && npm run build
```

Then anyone can check the output builds without installing your toolchain:

```sh
templetry verify --template . --set "project_name=Verify App"
```

## Step 7 — Write AGENTS.md

Ship the operating contract with the template: mission, core rules, the checks that must pass before finishing, and the safe change workflow. It renames with the project like everything else, so every generated project is born agent-ready. Keep it imperative and short — agents follow lists, not prose.

## Step 8 — Grow into a parent with forms

When a second structural variant appears:

1. Move the template into a subdirectory named after the form (`starter/`). Everything keeps working.
2. Add the sibling form with its own `template.yml`. **Do not share files between forms** — each is self-contained.
3. Give the parent a root README with a forms table.
4. Add a **Verify workflow** with one job per form × input combo: install the CLI, render, then build with the real toolchain.

The anti-explosion rule: additive variation is a **feature**; only structural variation justifies a new **form**. Features are cheap; forms are projects you maintain forever.

## Step 9 — Publish

Serve a `registry.json` (schema v2) from any URL:

```json
{
  "schema_version": 2,
  "parents": [
    { "key": "mystuff", "label": "My templates", "repo": "you/mystuff", "ref": "main",
      "forms": [ { "form": "starter", "name": "my-template", "path": "starter",
                   "status": "ready", "description": "What it scaffolds" } ] }
  ]
}
```

Consumers point at it with `--registry <url>`, or **Settings → Catalogs** in the app. Templates may also live on GitLab or Gitea — add a `source` field, see [multi-forge](multi-forge.md).

House rule worth stealing: flip `status` to `ready` **only when its CI is green**.

## Edge cases the spec already handles

Line endings normalize to LF; execute bits survive from the tarball; binaries are never scanned; output paths cannot escape the target directory or use names that break on Windows; manifests tolerate a UTF-8 BOM. You do not need to think about any of these — but if one bites, [the manifest spec](../spec/template-yml.md) is the normative answer.
