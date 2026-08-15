# Using templates

## Browsing the catalog

```sh
templetry list                          # the official catalog
templetry list --registry <url|file>    # anyone else's
```

Each line is `parent/form` — the reference every other command takes.

### Filtering

Every form declares a **taxonomy** on three axes: what it *is* (`kinds`), what it is written in (`languages`) and what it is built on (`frameworks`).

```sh
templetry list --tags                                  # show each form's axes
templetry list --kind database                         # anything that ships a schema
templetry list --kind cli --language go --language rust
templetry list --framework react
```

The `kinds` vocabulary is fixed: `frontend`, `backend`, `database`, `infra`, `multiplatform`, `android`, `ios`, `desktop`, `cli`. A form usually carries more than one — `dotnet/razor-web` is frontend *and* backend, a KMP form is multiplatform *and* android *and* ios *and* desktop.

Flags combine as **OR within an axis, AND across axes**, so the third example above means "(cli) and (go or rust)". A form that declares no taxonomy matches no filter.

## Inspecting before generating

A form declares its inputs in its manifest. Two ways to see them without generating anything:

```sh
templetry plan --template <dir> --set key=value      # human-readable operation plan
templetry plan --template <dir> --set key=value --json
```

`plan` shows exactly what would be written, renamed or excluded. It is the fastest way to understand a template you did not write — and the first place to look when a render surprises you.

## Variables, features and presets

**Variables** are values: your project's name, base package, module path. Every string variable automatically yields casing variants (`{name.pascal}`, `{name.kebab}`, `{name.snake}`, `{name.camel}`, `{name.flat}`), which is how one answer renames a class, a directory and a package id consistently.

**Features** are toggles that include or exclude parts of the template:

```sh
templetry init web/react-spa --out ./app \
  --set "project_name=Shop" \
  --feature router --feature vitest=false
```

A feature is on by its declared default unless you say otherwise. `--feature x` means true, `--feature x=false` means false.

**Presets** are named combinations, so common cases need one flag instead of five:

```sh
templetry init web/react-spa --out ./app --set "project_name=Shop" --preset minimal
```

Resolution order is **defaults → preset → your explicit flags**, so you can take a preset and override one thing. Features may also declare `requires` and `conflicts`; violating them is an error naming both features, never a silent auto-fix.

## Generating

```sh
# from the catalog (fetches the form)
templetry init <parent>/<form> --out <dir> --set k=v [--feature f] [--preset p]

# from a local checkout of a template
templetry render --template <dir> --out <dir> --set k=v
```

`--force` writes into a non-empty directory. Without it, a non-empty target is refused — deliberately, because scaffolding over existing work is rarely what anyone meant.

## Verifying the output compiles

If the form declares a `verify` block, you can build the rendered project in a container without installing its toolchain:

```sh
templetry verify --template ./web/react-spa --set "project_name=Shop"
templetry verify --template ./web/react-spa --dir ./already-rendered
```

It needs Docker. Template authors get this for free in CI; as a user it is a quick way to confirm a form works before you invest in it.

## Determinism

Same template commit plus same inputs produce **byte-identical output**. Practical consequences: rendering twice into two directories and diffing them yields nothing, and re-rendering an old project reproduces exactly what it was born as. That property is what makes the [update cycle](keeping-updated.md) trustworthy.

## Where templates can live

Bare references mean GitHub, but a catalog may host forms on GitLab or Gitea/Forgejo. Nothing changes in your commands — see [Multi-forge](multi-forge.md). For private templates, pass a token:

```sh
TEMPLETRY_TOKEN=<token> templetry init acme/service --out ./svc --registry https://…/registry.json
```
