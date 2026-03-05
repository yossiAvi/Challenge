# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**שאגת הארי** ("Harry's Roar") — a Hebrew-themed Pac-Man clone. Single-file HTML5 canvas game (`index.html`) with no build system, no dependencies, and no tests. Everything — HTML, CSS, JS, audio, game logic — lives in one file (~1900 lines).

To play/test: open `index.html` in a browser, or push to GitHub and open via GitHub Pages.

## Git Workflow

```bash
git add index.html
git commit -m "description"
git push
```

Git user is configured globally: `Yossi Avitan <yossi.avitan@gmail.com>`. The remote is GitHub (`yossiAvi/Challenge`).

## Architecture

The script is organized into 17 numbered sections (comments mark each one):

| Section | Contents |
|---|---|
| 1 | Constants (`CANVAS_W=800`, `CANVAS_H=600`, `TILE=20`, `COLS=21`, `ROWS=27`, game states) |
| 2 | Maze data (`MAZE_INIT` array), `isPassable()`, `isWall()`, `bfsPath()` BFS pathfinding |
| 3 | Canvas/context setup, image loading (PNG files: shaked, naim, kaminai, bibi, trump, shiran) |
| 4 | Web Audio API — procedural music (`MELODY` array, `scheduleMelodyLoop`, kick/snare/hihat) |
| 5 | Player object and movement logic (`updatePlayer`, `tryMove`) |
| 6 | Particles and popup text system |
| 7 | Background (sky gradient + animated jets) |
| 8 | Player object literal + init/reset functions |
| 9 | Enemy class (`Ghost`) — two AI types: `direct` (BFS to player) and `predictive` (4 tiles ahead) |
| 10 | Bonus items (bibi, trump, shiran) and bomb items |
| 11 | Collision detection (`checkPlayerEatDot`, player↔enemy, player↔bonus) |
| 12 | Game state transitions (power mode, hungry mode, win/death) |
| 13 | Keyboard input handler |
| 14 | All render functions (`drawMaze`, `drawPlayer`, `drawEnemies`, `drawStartScreen`, etc.) |
| 15 | Main `update(dt)` loop |
| 16 | Init & start (`loadImages` callback → `requestAnimationFrame(gameLoop)`) |
| 17 | Mobile/touch support (resize, D-pad, swipe, orientation) |

## Critical Maze Tile Values

```
0 = dot (passable, eatable)
1 = wall (impassable)
2 = ghost gate (impassable to both player AND ghosts outside pen)
3 = power pellet (passable, eatable)
4 = ghost pen interior (impassable to player, impassable to ghosts outside pen)
5 = eaten dot (passable — MUST stay 5, never use 2 for eaten dots)
```

**Important:** Eaten dots must be set to `5`, not `2`. Value `2` blocks `isPassable()` for both player and ghosts, which was a past bug that froze movement.

## Enemy System

`Ghost` class instances are stored in `enemies[]`. Each ghost:
- Starts `inPen = true`, exits via `penExitPhase` (0=in pen → 1=move to exit column → 2=move up through gate)
- Uses `bfsPath()` every `GHOST_MOVE_INTERVAL` frames (12 frames normally, slower in power mode)
- `type: 'direct'` targets player's current tile; `type: 'predictive'` targets 4 tiles ahead
- When eaten in power mode: `exploding` state → `dead` state → respawns after 4s

## Player Movement

Tile-based with pixel interpolation. `player.nextDx/nextDy` queues the next direction; `tryMove()` attempts the queued direction first, then the current direction. Player spawns at row 13, col 10 (maze center).

## Audio (Web Audio API)

Always wrap audio calls in `try/catch` — iOS Safari may throw on `AudioContext` creation. `getAudioCtx()` lazily creates the context. Music is scheduled ahead using `audioCtx.currentTime`. `musicGain.gain.value = 0.18`, `percGain.gain.value = 0.14`.

## Mobile Support (Section 17)

- Canvas scales via CSS `width/height` using `window.visualViewport` (not `innerHeight` — avoids browser chrome mismatch)
- D-pad is `position: absolute; right: 4px; top: 50%` — overlays the right black border of the canvas
- Button size is dynamic: `--btn` and `--btn-font` CSS variables set by JS based on canvas scale
- Portrait mode: `#rotateMsg` overlay shown via `@media (orientation: portrait) and (max-width: 900px)`
- iOS tap-to-start: uses `click` event on canvas (NOT `touchend`) because `passive: true` touchstart allows iOS to generate click events
- `orientationchange` + `screen.orientation.change` + `visualViewport resize` all trigger `resizeCanvas()`

## Image Assets

PNG files in root: `shaked.png` (player), `naim.png`, `kaminai.png`, `bibi.png`, `trump.png`, `shiran.png` (enemies/bonus). All loaded via `loadImages()` before game starts.
