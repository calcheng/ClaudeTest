# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Single-file browser games — no build tools, no dependencies, no package manager. Each game is a standalone `.html` file that can be opened directly in a browser.

## Running / Testing

Open any file directly in a browser:
```
open kart.html
open tictactoe.html
```

There are no build steps, no lint commands, and no test suite. Verification is manual: open the file and play.

## Git Workflow

After meaningful changes, commit and push:
```
git add <file>
git commit -m "descriptive message"
git push
```

Remote: `https://github.com/calcheng/ClaudeTest` (main branch)

## kart.html Architecture

A ~900-line single-file Mario Kart-style racing game. All code is in one `<script>` block, organized top-to-bottom in dependency order:

1. **Constants & config** — canvas size (900×600), physics tuning, power-up type constants
2. **Vec2** — 2D vector math class; `V2.ang(a)` creates a unit vector from angle
3. **Catmull-Rom spline** — `buildSpline()` samples `RAW_WPS` (20 hand-placed waypoints) into `SPT[]` (arc-length reparameterized, ~3px spacing) and `SPTAN[]` (tangents). This is the foundation for all track geometry, collision, and AI.
4. **Track queries** — `nearestSPIdx(pos)`, `nearestSPIdxFrom(pos, hint, window)` (faster, use when hint is available), `isOffTrack(pos)`, `distToTrack(pos)`
5. **Checkpoints** — `CPS[]`: 10 evenly-spaced indices into `SPT[]`. Lap counting uses `kart.nextCP` (sequential: starts at 1, wraps 9→0→lap complete)
6. **Particles & Projectiles** — simple arrays (`particles[]`, `projs[]`); projectiles handle green shell wall-bouncing and red shell homing
7. **Kart class** — physics update (acceleration/friction/steering/off-track penalty/jump), power-up state, `_spIdx` cached nearest spline index updated each tick via `nearestSPIdxFrom`
8. **AI class** — waypoint lookahead steering (reads `k._spIdx`, looks ahead ~80–100px), rubber-banding via `k.maxSpd` adjustment, stuck recovery
9. **Race manager** (`race` object) — state machine: `RS.TITLE → RS.COUNTDOWN → RS.RACING → RS.FINISH_WAIT → RS.RESULTS`
10. **Lap counting** (`tickLaps`) — validates forward direction via dot product with `SPTAN`; `nextCP` starts at 1 to skip the CP0 that karts spawn near
11. **Camera** — `applyCam(ctx, player)` rotates the world so the player always faces screen-up: `ctx.rotate(-player.angle - Math.PI/2)`. Minimap always uses raw world coordinates (not the rotated camera space).
12. **Track prerender** — `prerenderTrack()` draws the track once to an offscreen canvas at startup; `drawTrack(ctx)` blits it each frame via `drawImage`
13. **Game loop** — fixed-timestep physics at 1/60s accumulated against real `dt` (capped at 100ms to handle tab-switch spikes)

**Key coordinate convention**: kart `angle=0` means facing right (+x). Camera rotation formula `(-angle - π/2)` maps the kart's forward direction to screen-up. Kart sprites are drawn with `ctx.rotate(angle + π/2)` so the sprite nose points in the kart's facing direction.

**Power-up system**: `PU` object holds short string keys (`'mu'`, `'gs'`, etc.). `wrand(PU_WEIGHTS)` picks randomly. `kart.useItem()` dispatches on `kart.held`; lightning bolt directly mutates all `karts[]` entries.
