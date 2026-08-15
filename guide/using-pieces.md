# Using pieces

A **piece** is a decoupled unit of code you adopt *after* the project exists: role-based access control, an audit trail, a whole CRUD entity, dependency automation. It is the answer to "I didn't need this on day one, and I don't want to hand-write it on day ninety".

Pieces differ from features in one way that matters: a feature is decided when the project is generated and then dissolves into it; a piece arrives later and **keeps its own identity, variables and update cycle**.

## Seeing what is available

```sh
templetry pieces ./my-api
```

```
  rbac                     Role-based access control after the NIST model…
* audit-trail              Append-only record of who changed what and when…
  renovate                 Automated dependency updates…  (common)
(* already applied — add with: templetry add <piece>)
```

- `*` marks pieces already applied to this project.
- `(common)` marks pieces that live in a shared repository rather than in your form — see [common pieces](#common-pieces) below.

## Adopting one

```sh
templetry add rbac ./my-api
templetry add crud-resource ./my-api --set entity=Product
```

A piece may declare its own **variables**, and they can rename things: `crud-resource --set entity=Product` lands `models_product.py`, `routers/product.py` and `class Product`. That is a piece *per object* — adopt it once per entity.

## The safety rule

**A piece may only add files that do not already exist.** If a rendered piece path collides with something in your project, the whole operation is refused with the offending path, and nothing is written. Shared files (a `package.json`, a version catalog) are touched only through the piece's *declared patches*.

The consequence is worth stating plainly: **adopting a piece cannot overwrite your work.** If it needs to wire itself into your code, the template must expose a socket for it (see [authoring pieces](authoring-pieces.md)); it will not edit your files behind your back.

Still: `templetry add` writes to your working tree. Review with `git diff` before committing, as the command reminds you.

## What gets recorded

Each adopted piece is appended to `.templetry-answers.yml`:

```yaml
pieces:
  - name: rbac
    source: "github.com/Templetry/python@main/fastapi-users/pieces/rbac"
    commit: 8f21c0…
    variables:
      admin_role: admin
    files:
      - src/my_api/rbac.py
      - src/my_api/models_rbac.py
```

That record is why a piece can be updated later, and why re-adding one is refused: the project knows what it already has.

## Keeping pieces current

`templetry update` re-renders **every applied piece** at its source's head with the inputs you gave it, and puts the result through the same diff and three-way merge as the template's own files:

```sh
templetry update ./my-api            # preview
templetry update ./my-api --apply    # write
```

Entries show which piece owns each file:

```
  modified  src/my_api/rbac.py   [rbac]
```

See [Keeping projects updated](keeping-updated.md) for the full cycle.

## Common pieces

Some pieces are not specific to one form. They live in a shared repository ([Templetry/pieces](https://github.com/Templetry/pieces)) and declare which templates they support:

- **Universal** ones (`renovate`) apply anywhere.
- **Per-ecosystem** ones share a name with different implementations: you ask for `audit-trail` and get the one written for your stack. Asking from an unsupported project fails clearly: `piece audit-trail does not apply to web-react-spa`.

Everything else works the same, with one difference that matters for maintenance: a common piece records *its own* repository as its source, so when it is fixed upstream, `templetry update` brings the fix to every project that adopted it.

## Limits (today)

- **Removal is not implemented.** The answers file records which files each piece owns, so it stays possible; for now, undoing a piece means deleting those files yourself.
- **Pieces cannot depend on pieces.** Adopting `scim` will not pull `rbac` in for you; the piece's description says what it expects.
