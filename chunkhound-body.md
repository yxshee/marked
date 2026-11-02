Closes #470.

## Why

`run_git_diff` ran `git diff <range> --` with the user's full git config and inherited stdin, so common settings changed the output the diff parser depends on:

- `color.ui=always` adds ANSI codes even on a pipe. The parser never sees `diff --git` / `+++` / `@@` at the start of a line and returns 0 chunks, with no error.
- `diff.external` hands the diff to another tool; with a tool that fails, git exits 128 (`fatal: external diff died`).
- A textconv driver (`.gitattributes` `diff=<driver>`) makes the search index the converted text instead of the real change.
- `diff.noprefix` / `diff.mnemonicPrefix` drop or change the `a/` `b/` prefixes the parser strips from `+++ b/<path>`.
- Under MCP stdio, the child inherits stdin, which is the JSON-RPC stream.

Reproduced with git 2.43 in a repo with `color.ui=always`, `diff.noprefix=true` and a `sed s/hello/CONVERTED/` textconv: the output was `^[[1mdiff --git f.txt f.txt^[[m ... ^[[32m+^[[m^[[32mCONVERTED world^[[m`; with `diff.external=false` git exited 128.

## Change

Both `git diff` calls in `chunkhound/core/git_diff/runner.py` now go through one helper that passes `--no-color --no-ext-diff --no-textconv --src-prefix=a/ --dst-prefix=b/`, sets `stdin=DEVNULL`, and runs git with `LC_ALL=C` so the existing "unknown revision" check for the root-commit retry also works under a non-English locale. The options come after the range so the existing tests that read the range from `args[2]` are unchanged.

## Test

`test_real_git_ignores_user_diff_config` in `tests/unit/test_git_diff_runner.py` makes a real repo with `color.ui=always`, `diff.noprefix`, a textconv driver and `diff.external=false`, runs `run_git_diff`, and checks there are no ANSI codes or converted text and that the parser returns one chunk for `notes.txt` with `+hello world`. It fails on `main` (git exits 128 from the external diff) and passes with this change.
