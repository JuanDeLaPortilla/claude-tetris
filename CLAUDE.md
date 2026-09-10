# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A classic Tetris implementation in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No dependencies, no build step, no `package.json`.

## Running / testing

There is no build, lint, or test tooling. To run the game, open `index.html` directly or serve the directory statically:

```bash
python3 -m http.server 8000   # or: npx serve .   or: php -S localhost:8000
```

Then open `http://localhost:8000`. There is no automated test suite — verify changes by playing the game in a browser.

## Architecture

Three files cooperate directly via global scope (no modules, no imports):

- `index.html` — DOM structure: the main `#board` canvas (300×600, 10×20 cells), the `#next-canvas` preview, HUD spans (`#score`, `#lines`, `#level`), and the `#overlay` for pause/game-over states.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, structured around a single mutable global state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) rather than a class or module pattern.

Key mechanics in `game.js`:

- **Board model**: a `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying the locked piece.
- **Pieces**: defined as square matrices in `PIECES`. Rotation (`rotateCW`) is computed via transpose + row reversal — there are no pre-rotated states.
- **Collision** (`collide`): checks board bounds and overlap with locked cells; used both for movement and for the ghost-piece projection.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found, else the rotation is discarded.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time (`dropAccum`) and advances the piece one row once `dropInterval` is exceeded.
- **Locking & clearing**: `lockPiece` → `merge` (writes the piece into `board`) → `clearLines` (bottom-up scan, removes full rows, unshifts empty rows) → `spawn` (promotes `next` to `current`, generates a new `next`; if the new piece immediately collides, triggers `endGame`).
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row, soft drop 1 pt/row. Level increases every 10 cleared lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece** (`ghostY`): projects `current` downward until collision, drawn at `globalAlpha = 0.2`.

Tunable constants live at the top of `game.js` (`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`). If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
