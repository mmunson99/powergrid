# Contributor notes (powergrid fork)

Working notes for contributing changes from this fork (`myfork` =
`mmunson99/powergrid`) back to upstream (`origin` = `boardgamers/powergrid`).
These exist to avoid repeating mistakes we've already hit.

## Always base a PR branch on fresh upstream master

Upstream moves fast (new maps and fixes land frequently). If you branch from a
stale local `master`, your diff will silently **revert** other people's work —
e.g. a branch forked before the Australia map was added showed up as *deleting*
the Australia uranium-mine market UI in `viewer/src/components/Game.vue`.

Before starting any change destined for a PR:

```
git fetch origin
git checkout -b my-fix origin/master   # branch off the latest upstream master
```

Then apply only the intended changes and confirm the net diff is what you
expect:

```
git diff origin/master --stat          # should list ONLY the files you touched
```

If a change you already made upstream got merged (check `git log HEAD..origin/master`),
don't re-include it — start clean from the merged master instead of rebasing a
tangled local history.

## Opening the PR requires being signed in to GitHub

On the compare page
(`github.com/boardgamers/powergrid/compare/master...mmunson99:powergrid:<branch>`),
the green **Create pull request** button is **hidden when logged out**. The
tell-tale sign is the top-right corner showing **Sign in / Sign up** instead of
your avatar — the "Able to merge" banner and the diff still render, so it's easy
to miss. Sign in to the `mmunson99` account, reload the compare URL, and the
button appears next to the "Able to merge" row.

## The Prettier / ESLint single-quote deadlock

ESLint is configured with `quotes: ['error', 'single']` (see
`viewer/.eslintrc.js` and the engine config). Prettier, however, **flips a
string to double quotes when it contains a single-quote/apostrophe**, to avoid
escaping. A string like:

```ts
"... marked with an 'X' ..."   // double-quoted to hold the inner 'X'
```

is mutually unsatisfiable: Prettier wants the double quotes (because of the
inner `'X'`), ESLint demands single quotes. CI fails on whichever check runs.

**Fix: don't put quote marks inside the string.** Reword so the single-quoted
string needs no escaping:

```ts
'... marked with an X ...'     // single-quoted, no inner quotes, passes both
```

Avoid apostrophes too (`don't`, `player's`) in these long rule strings for the
same reason — prefer rewording over escaping.

## Pre-commit hook reformats files

A Husky `pre-commit` hook runs `lint-staged` → `prettier --write`. It will
reformat staged files and can unstage your changes if formatting differs. To
keep commits smooth, run Prettier yourself first:

```
npx prettier --plugin-search-dir=. --write <files you changed>
```

## Verify locally before pushing (mirror CI's lint_and_test)

```
npx prettier --plugin-search-dir=. --check .   # formatting
cd engine && npx eslint . --ext .ts            # 0 errors required (warnings OK)
cd engine && npm test                          # mocha specs
cd viewer && npm run build                      # compiles + lints Game.vue etc.
```

ESLint **warnings** (e.g. pre-existing `no-explicit-any` /
`explicit-module-boundary-types` in `engine.ts`, `wrapper.ts`, `*.spec.ts`) do
not fail CI — only **errors** do. Don't be alarmed by the warning list; check
the exit code / error count.
