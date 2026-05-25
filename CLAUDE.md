# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

This repo is a collection of **single-file browser games** built with vanilla HTML, CSS, Canvas, and JS. No build system, no package manager, no dependencies. Each game is one self-contained `.html` file you open directly in a browser.

Current games:
- `shooter.html` — top-down arena shooter (the main project)
- `tictactoe.html` — two-player hot-seat tic-tac-toe

## Running / "building"

There is no build, lint, or test command. To run or verify changes:

```sh
open shooter.html        # or any other .html file
```

When verifying a change to a game, actually open it in the browser and play through the affected behavior — type checking and test suites do not exist here.

## Git workflow (standing user rule — IMPORTANT)

**As you do work, commit to git and push to GitHub regularly with clean commit messages so we never lose progress.** The user has explicitly and durably authorized this — do not ask before each commit/push for routine work.

Cadence:
- Commit + push after every meaningful unit of work (a feature added, a bug fixed, a tuning pass, a refactor, a doc update). Do not batch multiple unrelated changes into one commit.
- Never leave the working tree dirty at the end of a turn that produced real changes. If you edited code, the turn ends with `git push` already done.
- If a change spans multiple logical pieces, make multiple commits in order, then push.

Commit-message style:
- Terse imperative subject (≤72 chars), blank line, optional bullet body explaining *why* if non-obvious.
- One concern per commit. Don't mix unrelated edits.
- Use `HEREDOC` form when invoking `git commit -m` so formatting is preserved (see existing commit history for examples).

Still confirm before destructive operations: `push --force`, `reset --hard`, branch deletion, history rewrites, anything that could overwrite or discard committed work.

Remote: `https://github.com/Gnorezgnaw/retro-shooter` (default branch `main`).

## Architecture: `shooter.html`

One file, ~1000 lines, organized into labeled sections (`// =====` banners). Important conventions that aren't obvious from a quick skim:

### State machine
`gameState` is a single string, one of `STATE.MENU | PLAYING | LEVEL_COMPLETE | GAME_OVER | VICTORY`. Both the main `loop()` update branch and the render branch switch on it. Click events also dispatch via `gameState` (advance level, restart, return to menu).

### Entities
Flat plain-object arrays — no classes. `player` is a single object; `enemies`, `bullets`, `particles` are arrays. Collision is circle-circle via squared-distance comparison against `radius` sums.

### Sprite system
Sprites are defined as arrays of ASCII strings (one char per pixel) plus a palette object (`{ char: colorString | null }`). `createSprite(pixels, palette, scale)` rasterizes each definition once at startup into an offscreen `<canvas>`, and `drawSprite(sprite, x, y, angle)` blits the cached canvas with rotation. **All sprites are drawn facing right (angle 0) with the gun/front on the right side** — rotation handles aiming. A walk cycle is two pre-rendered frames (`F1`/`F2`) swapped on a timer while moving.

To add a new enemy or weapon visual: define new `PIXELS_F1`/`F2` arrays + a `PAL_*` palette, register them in the `sprites` object, then reference them from the render branch.

### Tuning tables
Gameplay balance lives in two tables near the top of the script section — change these rather than editing logic:
- `ENEMY_TYPES` — hp, speed, radius, damage, behavior (`melee` | `shooter` | `boss`), range, fireRate per enemy type
- `LEVELS` — array of `{ name, waves: [{ enemyType: count, ... }] }`. A wave is a count-per-type bag; spawning is shuffled and edge-randomized.

Enemy `behavior` strings are interpreted in `updateEnemies` — adding a new behavior means updating that switch and the spawn table.

### Frame loop
`loop()` uses `requestAnimationFrame` with `dt` (capped at 0.05s) so motion is framerate-independent. All update functions take `dt`; render is pure.
