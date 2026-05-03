# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

Open `index.html` directly in a browser, or use a local server:

```bash
npx serve .
```

Then visit `http://localhost:3000`. No build step, no bundler, no dependencies.

## Architecture

The entire game logic lives in a single file: `game.js` (~424 lines). There is no module system — everything is plain ES6+ in one `'use strict'` script loaded by `index.html`.

**Canvas setup:** Fixed 800×600 canvas (`W`, `H` constants). The 2D context (`ctx`) is a module-level global used directly by all `draw()` methods.

**Game loop:** `requestAnimationFrame`-based loop with a delta-time cap of 50ms to prevent physics spikes. State is advanced in `update(dt)` and rendered in `draw()`.

**Game state machine:** A single `state` string (`'playing' | 'dead' | 'gameover'`) controls which logic runs each frame.

**Entities** — each class follows the same pattern: constructor sets position/velocity, `update(dt)` advances physics with toroidal wrapping via `wrap()`, `draw()` renders to `ctx`, and a `dead` boolean marks removal:
- `Ship` — player ship with invincibility timer, shoot cooldown, thrust flame rendering
- `Asteroid` — random irregular polygon; `split()` returns two smaller asteroids (size 3→2→1, then destroyed)
- `Bullet` — time-to-live projectile
- `Particle` — explosion fragment with fade-out alpha

**Collision detection:** Simple circle-circle (`dist()` vs sum of radii). Bullets vs asteroids, then ship vs asteroids (skipped while `ship.invincible > 0`).

**Level progression:** `asteroids.length === 0` triggers `nextLevel()`, which increments `level` and spawns `3 + level` new size-3 asteroids.

**Scoring constants** (defined at top of file):
- Size 3 (large): 20 pts
- Size 2 (medium): 50 pts
- Size 1 (small): 100 pts
