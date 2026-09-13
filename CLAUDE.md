# CLD-FileIndex

This repo is one thing: the `.cld` **file index** -- what is inside a
file -- and the tooling that keeps those indexes honest. Its sibling
CLD-DirIndex covers what is inside a directory. Resist merging them
back, and resist adding anything that is not about indexing a file's
contents.

## Where the work is

Read the README's "open problem" section before changing the spec. The
short version: the seven type letters in `spec/file-index-format.txt`
were derived from JavaScript and do not extend to other languages or to
non-source file types. The checker is fine -- the alphabet is not. The
fix is per-file-type profiles, and it is not designed yet.

Do NOT paper over this by adding letters ad hoc. A letter added for one
language without a profile model just moves the problem.

## Working here

- Run `tests/run-tests.sh` before committing a change to the checker.
  33 assertions over throwaway git fixtures.
- `scripts/git-hooks/pre-commit` MUST stay byte-identical to the copy in
  CLD-DirIndex. See the contract section of the README.
- POSIX `/bin/sh` only -- no bashisms, no GNU-only flags. Anything
  project-specific belongs in `cld.conf`, never in the checker.

## Same-commit rule

Changing a symbol means updating that file's `<filename>.cld` in the
same commit. That discipline is what the tooling exists to support.
