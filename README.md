# makefun-sitegen-speed

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/woodhntr1-gif/makefun-sitegen-speed/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/woodhntr1-gif/makefun-sitegen-speed/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own makefun-site 1.0.0: `solve.js` builds the same output tree as the reference — same pages, index and tag pages, report and static files, byte for byte on every case — in clearly less CPU time.

## How it is faster

The reference walks one character at a time (its inline renderer escapes every character with its own regex call) and tests a regex per Markdown line. This module keeps the reference's answers exactly and changes only the work:

- Inline rendering jumps from one interesting character to the next with a single native regex scan, copying plain stretches whole and escaping them in bulk; the only characters handled one at a time are the rare marks.
- The Markdown lexer reads each line's first characters instead of testing seven regexes per line, and the front-matter parser walks lines with `indexOf` instead of splitting and matching.
- Slugs take a fast path on all-ASCII text (where NFKD is the identity); word counts use one native scan instead of building an array of every word; tag slugs are computed once per tag instead of three times.
- The template interpreter keeps the reference's tree walk but resolves lookups with a fast path and reuses each post's card instead of building it twice.

Nothing is cached from one case to the next, and the module is pure: no clock, no randomness, no locale, no network, no files.

## Run it

The harness runs `node /bot/main.ts`, which reads every case from `/input/cases.json`, answers each one alone with `run()` from `solve.js`, and writes `/output/answers.json`. Outside a container, `MAKEFUN_INPUT_DIR` and `MAKEFUN_OUTPUT_DIR` stand in for `/input` and `/output`.

## Verification

Checked against the kit's practice set (3 cases, 257/348/486 files): every case's answer is byte-identical to makefun-site 1.0.0's expected output, and the full driver (`main.ts` + `solve.js`) passes 3/3 through the scorer's comparison.

Correctness was also checked against the reference implementation itself, not just the practice set: 300 further cases drawn with the competition's own case generator (100 seeds × 3 cases) plus 145 hand-built edge cases — front-matter refusals of every kind, Markdown corners (unclosed fences, 7-hash headings, unicode slug collisions, list continuations, nested emphasis, escaped links), broken templates (unclosed tags, unknown tags, include chains 9 deep, missing includes), and site extremes (per_page 1 and 1000, recent 0, no posts, no templates, static files shadowing built pages). All 448 checks are byte-identical to the reference, and running the same input twice gives byte-identical output.

Speed, measured locally with `--no-incremental-marking` (median of 10 runs of `run()` over the 3 practice cases): the reference takes 563 ms of CPU, this module 161 ms — **3.51× faster** (per case: 4.0×, 3.4×, 3.9×).
