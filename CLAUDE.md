# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript + HTML5 Canvas. No dependencies, no `package.json`, no build, no lint, no tests. The README and UI text are in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`) and visit `http://localhost:8000`. Verify changes manually in the browser.

## Architecture

Three files: `index.html` (DOM + two canvases), `style.css`, and `game.js` (all logic, a single global-scope script loaded with `'use strict'`; no modules).

Things that span the code and are easy to miss:

- **Board/piece encoding**: the board is a `ROWS × COLS` matrix of `0` or a color index 1–7. The same index is used in `PIECES` (shape matrices), `COLORS` (palette), and the board cells, so adding or reordering a piece means updating all of them in lockstep. `randomPiece` hard-codes `7` piece types.
- **Control flow is mutually recursive**: `lockPiece` → `merge` → `clearLines` → `spawn`, and `spawn` calls `endGame()` if the new piece collides immediately. `hardDrop`, `softDrop` and the `loop` gravity tick all funnel through `lockPiece`.
- **Game loop**: `requestAnimationFrame` handle is in `animId`; `loop` accumulates `dropAccum` against `dropInterval`. Pause/game-over cancel the frame, and resuming re-seeds `lastTime` and calls `loop` directly. `init()` doubles as restart (bound to the restart button).
- **Scoring/speed** live in `clearLines` (`LINE_SCORES[cleared] * level`; level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`). Soft drop awards 1 point per row, hard drop 2 per row.
- **Rotation**: `rotateCW` + horizontal-only kicks `[0,-1,1,-2,2]` in `tryRotate` (no SRS tables, no floor kicks).
- **Canvas size is coupled to constants**: `index.html` hard-codes `#board` as `300×600` (`COLS×BLOCK` × `ROWS×BLOCK`); changing `COLS`, `ROWS` or `BLOCK` requires updating it. The preview canvas (`#next-canvas`, 120×120) uses its own 30px block size inside `drawNext`.

## CI

`.github/workflows/` has `claude.yml` (PR assistant, `@claude` mentions) and `claude-code-review.yml` (automated PR review). Neither runs project tests.
