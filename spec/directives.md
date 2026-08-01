# Directive specification — v1

**Status:** normative (ADR-0007 accepted). Every rule here is engine behavior.

Directives live inside comments, so every template file remains valid syntax in its own language (ADR-0003). Prefix: `tpl:`.

## Grammar

Exactly three directives exist in v1:

| Directive | Form | Effect |
|---|---|---|
| `tpl:if` | `tpl:if <feature-key>` or `tpl:if !<feature-key>` | Start a conditional block. Lines inside are kept only if the feature is active (or inactive with `!`). |
| `tpl:endif` | `tpl:endif` | Close the innermost `tpl:if`. |
| `tpl:var` | `tpl:var <variable-key> <literal>` | On this line, replace the first occurrence of `<literal>` (in the code part) with the variable's value, then remove the directive comment. |

Deliberately absent from v1 (each addition is public API): `tpl:else` (write two blocks with `!` instead), boolean expressions (`and`/`or`), whole-file directives (the manifest's feature `files` globs cover that), `requires`/`conflicts` between features (postponed to v1.1).

## Placement rules

- `tpl:if` and `tpl:endif` must be **alone on their line** (only the comment). The whole line is removed from output. Anything else is an error.
- `tpl:var` is **attached to a code line**: `<code> <comment-prefix> tpl:var <key> <literal>`. The output keeps the code with the substitution applied and the directive comment removed.
- Nesting `tpl:if` blocks is allowed.

## Comment styles

The scanner recognizes the comment style from the file extension (or special basename). Unknown extension → the file is **not scanned** (safe by default). Binary files are never scanned.

| Style | Extensions / basenames |
|---|---|
| `//` | .go .kt .kts .gradle .java .js .jsx .ts .tsx .swift .dart .rs .c .h .cpp .cs .scala .groovy |
| `#` | .py .rb .sh .bash .yml .yaml .toml .properties .tf .gitignore .gitattributes .editorconfig · Dockerfile Makefile |
| `--` | .sql .lua |
| `<!-- -->` | .html .xml .md .vue .svg |

JSON has no comments — that is what manifest **patches** are for (see template-yml spec). `.tpl` escape-hatch files are rendered as full text templates and never directive-scanned.

## Examples

```kotlin
android {
    namespace = "com.template.base"
    minSdk = 24                       // tpl:var min_sdk 24

    // tpl:if room
    dependencies { implementation(libs.room.runtime) }
    // tpl:endif
}
```

```html
<!-- tpl:if analytics -->
<script src="/analytics.js"></script>
<!-- tpl:endif -->
```

## Errors (all reported as `file:line`)

- Unknown directive after `tpl:` (typos never pass silently).
- `tpl:if` with an unknown feature key; `tpl:var` with an unknown variable key.
- `tpl:endif` without an open `tpl:if`; `tpl:if` left unclosed at end of file.
- `tpl:if`/`tpl:endif` sharing a line with code.
- `tpl:var` whose `<literal>` does not occur in the line's code part.
