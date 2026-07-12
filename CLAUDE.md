# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json — just three files (`index.html`, `style.css`, `game.js`).

## Running the game

No install or build required. Either open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8000
# or
npx serve .
```

Then visit `http://localhost:8000`. There is no test suite, linter, or bundler configured for this project.

## Architecture

All game logic lives in `game.js` (~300 lines) as a single script with module-level state — there is no class structure or separate modules.

- **Board model**: `board` is a `ROWS × COLS` matrix (20×10). Each cell is `0` (empty) or an integer `1`–`7` identifying the piece color that occupies it (index into `COLORS`/`PIECES`).
- **Pieces**: defined in `PIECES` as square matrices. Rotation is done via `rotateCW`, a transpose + row-reverse — there's no separate rotation-state table (no full SRS).
- **Wall kicks**: `tryRotate` attempts the rotated shape at offsets `[0, -1, 1, -2, 2]` columns, using the first that doesn't collide.
- **Collision**: `collide(shape, ox, oy)` is the single source of truth for whether a shape can occupy a position — used by movement, rotation, ghost-piece projection, and spawn checks.
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulating elapsed time in `dropAccum` and advancing the piece one row once `dropInterval` is exceeded (or locking it if it can't move down).
- **Locking a piece**: `lockPiece()` → `merge()` (writes the piece into `board`) → `clearLines()` → `spawn()` (promotes `next` to `current`, generates a new `next`, and checks for game over via collision at spawn position).
- **Scoring/leveling**: line clears use `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row, soft drop adds 1 pt/row. Level increases every 10 lines, and `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering**: `draw()` clears and redraws the whole board canvas each frame (grid, locked blocks, ghost piece at `globalAlpha = 0.2`, current piece). `drawNext()` renders the preview piece on a separate small canvas (`#next-canvas`).
- **Input**: a single `keydown` listener switches on `e.code` (arrows + `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause), guarded by `paused`/`gameOver` flags.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, and initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS×BLOCK` by `ROWS×BLOCK`).
