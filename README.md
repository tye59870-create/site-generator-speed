# site-generator-speed

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/tye59870-create/site-generator-speed/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/tye59870-create/site-generator-speed/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for makefun-site 1.0.0, make.fun's own static site generator.
`solve.js` returns, for every case's input, exactly the output tree that
makefun-site 1.0.0's build returns for the case's site — each page's
`index.html`, the index and tag pages, `build-report.txt` and each static
file, byte for byte — only faster. Pure JavaScript, standard library only;
there is nothing to install and no WebAssembly, native addon or prebuilt
binary.

## Run it

```sh
node main.ts
```

`main.ts` reads every case from `/input/cases.json` (or
`$MAKEFUN_INPUT_DIR`), answers each with `run` from `solve.js`, and writes
`/output/answers.json` (or `$MAKEFUN_OUTPUT_DIR`). Node 22, standard
library only.

## How it is faster

The reference is a straightforward pipeline; this one keeps its exact
semantics and removes its wasted work:

- HTML escaping runs on whole text runs, never one regex per character.
- `slugify` has an ASCII fast path (NFKD is the identity on ASCII) and a
  cache; each tag's slug is computed once, not three times.
- Front matter is found by slicing the source, never by splitting the
  whole file and joining its body back.
- Markdown is lexed and rendered in one pass, with a single-pass inline
  scanner that jumps between special characters instead of stepping
  character by character.
- Templates render into an array with a scope stack (no per-iteration
  scope wrapper, no closure per loop element), and the `site` / `tags` /
  `recent` scope is built once per case instead of spread into every
  render.
- Word counting and date formatting avoid regex and array allocation.

## Test it

The kit's practice set (3 cases) is graded locally with the kit's own
grader, run from the unpacked kit:

```sh
node cli/makefun.mjs grade-local --kit . --bot <repository> --driver process --worktree
```

It prints the scorer's `scores.json`. This build passes all 3 practice
cases (`passed: 1`, `cases_passed: 3`, `cases_total: 3`), reproduced byte
for byte across the two executions, at about 2.9x the speed of the
reference on the same machine.

## License

MIT. See [LICENSE](LICENSE).
