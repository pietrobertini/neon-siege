# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Neon Siege** is a top-down browser-based shooter game implemented as a single self-contained HTML file. No build step, no dependencies, no server required — open and play directly in any modern browser.

## Running the Game

**Local development:**
```bash
python3 -m http.server 8000
```
Then open `http://localhost:8000/shooter.html` in a browser.

**On a remote server:**
Use any available port (e.g., 9900):
```bash
python3 -m http.server 9900
```

## File Structure

Everything is in `/home/mastercontrol/claude/shooter.html`:
- **HTML** (lines 1–40): Canvas container and UI overlays (Menu, Game Over, How to Play)
- **CSS** (lines 41–150): Retro styling, scanlines, glow effects, button animations
- **JavaScript** (lines 151–end): Game engine

## Game Architecture

### State Machine
States flow: `MENU → PLAYING → LEVEL_CLEAR → PLAYING → GAME_OVER → MENU`

Managed by `state` variable and `update()` function which branches on current state.

### Game Loop
`requestAnimationFrame` with delta-time capping at 50ms. Each frame:
1. Update player, enemies, bullets, particles
2. Check collisions (bullet vs enemy, enemy vs player)
3. Draw everything to canvas
4. Check if level is won (all enemies dead)

### Core Systems

**Player** (`player` object):
- Position, HP, angle (for gun rotation)
- Movement via arrow keys (180 px/s), diagonal allowed
- Aim: rotates to mouse position via `atan2`
- Fire: left-click to spawn bullets toward cursor
- Invincibility frames after taking damage (900ms)

**Enemies** (array `enemies`):
- Three types: Grunt (1 HP, beeline), Tank (3 HP, slow), Dasher (1 HP, sprints)
- Spawn off-screen edges, approach player with varying speeds
- Death animation: expand + fade over 20 frames, spawn particles
- Spawn delay system staggeres enemy arrivals each level

**Bullets** (array `bullets`):
- Fire in direction of mouse cursor, travel 620 px/s
- Circular collision detection (bullet radius 4, enemy radius varies)
- Trail effect (3 ghost copies at decreasing opacity)
- Cull after 1.8s or off-screen

**Particles** (array `particles`):
- Spawned on enemy death (8–10 per enemy)
- Random velocity spread, gravity-like decel, fade out

### Levels & Progression

| Level | Enemy Count | New Type | Speed Modifier |
|-------|------------|----------|-----------------|
| 1–2 | 8, 14 | Grunt | +5 per level |
| 3–4 | 18, 24 | Tank (+3) | +3 per level |
| 5+ | 30+ | Dasher (+5) | +5 per level |

Level clear: 2.2-second splash, then spawn next level with higher enemy count.

### Visuals

- **Drawing**: Canvas 2D API primitives (arcs, rects, lines) — no image assets
- **Palette**: Player green/cyan, Grunt red, Tank purple, Dasher yellow, bullets bright yellow
- **Effects**: Muzzle flash (2-frame glow), grid background, vignette overlay, scanline CSS overlay
- **Fonts**: `monospace` with `text-shadow` glow for retro look

## Key Functions

- `update(dt, now)` — Main game tick, branches by state
- `updatePlayer()`, `updateEnemies()`, `updateBullets()`, `updateParticles()` — System updates
- `checkCollisions()` — Bullet vs enemy + enemy vs player
- `checkLevelEnd()` — Win condition (all enemies dead)
- `spawnLevel()`, `spawnEnemiesForLevel()` — Level setup
- `draw*()` functions — All rendering
- `loop(now)` — RAF loop, calls update + draw each frame

## Common Tasks

**Adjusting difficulty:**
- Enemy count: Edit `enemyCountForLevel()` returns
- Enemy speed: Modify `speed` in `createEnemy()` and `updateEnemies()`
- Player speed: Change `PLAYER_SPEED` (180)
- Fire rate: Change `FIRE_RATE` (150 ms)

**Adding a new enemy type:**
1. Add type string in `spawnEnemiesForLevel()` types array
2. Add spawn condition (e.g., `if (level >= X)`)
3. Add case in `createEnemy()` with stats (hp, speed, radius, score)
4. Add drawing logic in `drawEnemy()` with color/shape

**Changing visuals:**
- Colors: Grep for hex codes in draw functions (#0ff for cyan, #f33 for red, etc.)
- Grid/background: `drawGrid()` and canvas fill color
- UI fonts/glow: CSS `text-shadow` or Canvas `shadowColor`/`shadowBlur`

## Git Workflow

All changes committed locally with clean messages, then pushed to GitHub:
```bash
git add <file>
git commit -m "Clear description"
git push
```

Repository: https://github.com/pietrobertini/neon-siege

## Notes

- Game is 800×600 canvas; adjust `W` and `H` constants to resize
- No external libraries; pure vanilla JS + Canvas
- Mouse position tracked in real-time; click and hold to continuous fire
- All game state is in the `loop()` function scope for simplicity — refactoring into classes is viable if complexity grows
