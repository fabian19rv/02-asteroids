# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone built with plain HTML5 Canvas and vanilla ES6+ JavaScript. No build step, no bundler, no dependencies, no package.json. The entire game lives in a single file, `game.js` (~420 lines), rendered onto a fixed 800x600 canvas defined in `index.html`.

## Running

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build, lint, or test tooling in this repo — changes to `game.js` take effect on page reload with no compile step.

## Architecture

Everything is in `game.js`, organized top-to-bottom as: input handling → math utils → entity classes → game state → update loop → draw loop → main loop.

- **Input**: a global `keys` map (held state) and `justPressed` map (edge-triggered, consumed via `pressed(code)`) are filled by `keydown`/`keyup` listeners. Game logic reads these maps directly rather than passing input through parameters.
- **Toroidal space**: all entities wrap around canvas edges via `wrap(v, max)`. Any new entity with position needs this applied in its `update`.
- **Entity classes** (`Bullet`, `Asteroid`, `Ship`, `Particle`): each owns its own `update(dt)` and `draw()`, and marks itself `dead = true` when it should be removed. There's no shared base class or entity manager — the global arrays (`bullets`, `asteroids`, `particles`) are filtered each frame with `.filter(e => !e.dead)`.
- **Asteroid sizes**: size is an int 3 (large) → 1 (small), indexing into parallel arrays `RADII`, `SPEEDS`, `POINTS`. `Asteroid.split()` returns two smaller asteroids (or `[]` at size 1), spawned at the same position with new random velocity/polygon.
- **Game state machine**: a single `state` variable (`'playing' | 'dead' | 'gameover'`) drives branching in `update()`. `'dead'` is a temporary post-death state (`deadTimer` counts down before respawn with invincibility); `'gameover'` waits for Space to call `initGame()` again.
- **Collision detection**: brute-force O(n*m) distance checks (`dist(a, b) < radius sum`) in `update()` — bullets vs asteroids, then ship vs asteroids (skipped while `ship.invincible > 0`). No spatial partitioning; the asteroid/bullet counts are small enough that this doesn't matter.
- **Main loop**: `requestAnimationFrame(loop)` computes `dt` in seconds (clamped to 0.05 max to avoid big jumps on tab-switch/lag), then calls `update(dt)` followed by `draw()`. All physics (ship thrust/drag, asteroid motion, bullet TTL) is `dt`-scaled, not frame-rate dependent.

## Notes

- `README.md` references power-ups and a special "shooting star" asteroid type that are not implemented in the current `game.js` — treat the code as the source of truth over the README when they disagree.
