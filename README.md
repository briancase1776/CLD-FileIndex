# CLD-FileIndex

A `.cld` **file index** records what is inside a file. One index per
file, named `<filename>.cld`, sitting beside it:

```
FILE lib/example.js
C ExamplePanel          Loads the example sub-section HTML fragments
F ExamplePanel.#load    Fetches each section HTML in order and appends it
D #oHooks               Callback bundle handed in by core at construction
```

Its sibling is [CLD-DirIndex](https://github.com/briancase1776/CLD-DirIndex),
which records what is inside a *directory*. The two are deliberately
separate repos: a directory index is language-agnostic by construction
and is finished, while this one is not (see below).

## State: usable, not done

What works today, and is covered by `tests/run-tests.sh` (33 assertions):

| Check | |
|---|---|
| `SYM-DEAD` | **blocks** -- the target file does not exist |
| `SYM-NAME` | **blocks** -- not named `<target-basename>.cld` |
| `SYM-DUP` | **blocks** -- two indexes claim one target |
| `SYM-HDR` | **blocks** -- unknown line-1 species |
| `SYM-KEY` | **blocks** -- field 2 is not a usable lookup key |
| `SYM-MISS` | *warns* -- a declaring source file with no index |

Those are **identity** checks -- is this index telling the truth about
its target -- and they are language-agnostic. They work now.

## The open problem: the type-letter alphabet is JavaScript-shaped

`spec/file-index-format.txt` defines seven type letters:

```
F function/method/getter   C class      K constant/pattern   E event
D data field               G module global                   R load-time effect
```

That alphabet was derived from one JavaScript codebase, and it shows:

- `G` is *"a name the file hangs on the global object"* -- `window.Foo`.
  It has no meaning in Go, Rust, Java, or Python.
- `R` describes import side effects: monkey-patches, anonymous
  listeners, self-registration calls.
- `E` (event) is browser-flavoured.
- Every example uses `#private` fields -- JavaScript private syntax.

The gaps matter more than the misfits. There is no letter for a Go
interface or struct, a Rust trait or impl or macro, a Python decorator
or `@property`, a C typedef or macro, a Java enum or annotation. You
cannot index a Go file in `F/C/K/E/D/G/R` without lying about what the
symbols are -- and a lying index is the one thing this tooling exists
to prevent.

The same applies beyond source code. "What is inside a file" has an
answer for a CSS file, a JSON schema, a Markdown document, a SQL
migration. None of them are symbols in the JavaScript sense.

**The shape of the fix is a per-file-type profile**: a type-letter
table, declaration patterns, and accessor keywords per language or
format, plus a rule for what happens when a type needs a letter the
alphabet does not have. That design does not exist yet. It is the
reason this repo is separate -- so that work can churn without dragging
a finished directory-index tool through every revision.

Only two things in the *code* are JavaScript-specific, and both are
already configurable in `cld.conf`: the `get|set|static|async` accessor
list in `SYM-KEY`, and `CLD_DECL_RE` / `CLD_SYM_MISS_EXTS` for
`SYM-MISS`. The work is in the spec, not the checker.

## Contract with CLD-DirIndex

Both repos ship `scripts/git-hooks/pre-commit`, and a host repo that
adopts both has only one `.git/hooks/pre-commit`. **Keep that
dispatcher byte-identical in both repos.** It runs everything in
`pre-commit.d/` in lexical order, so each side just drops its own `NN-`
gate in (`20-index-audit` there, `30-symbol-audit` here) and they
coexist. If the dispatchers ever diverge, whichever installer runs
second silently wins.

`scripts/cld-config.sh`, `scripts/git-hooks/install.sh` and the test
harness are also copies rather than a shared dependency. That is a
deliberate trade: the directory side is finished, so a frozen copy does
not drift.

## Quick start

```sh
./scripts/git-hooks/install.sh   # gate this repo
tests/run-tests.sh               # 33 passed, 0 failed
```
