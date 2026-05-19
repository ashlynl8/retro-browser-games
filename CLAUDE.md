# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of retro-style browser games built with vanilla HTML5, CSS, and JavaScript. Each game is a self-contained HTML file with embedded CSS and JavaScript — no build process, no dependencies. Open any `.html` file directly in a web browser to play.

## Model Configuration

Default model for this project: **Haiku 4.5**. Set via `/model` or in settings for consistency.

## Running the Games

- **Tic-Tac-Toe**: Open `tic-tac-toe.html` in a browser
- **Shooter**: Open `shooter.html` in a browser

No local server or build step required.

## Development Commands

**There are no build, lint, or test commands.** Each game is a single HTML file designed to run directly in a browser — no server, bundler, or test framework needed. Verification is manual browser testing only.

## Architecture Overview

### Tic-Tac-Toe (`tic-tac-toe.html`)

- **DOM-based**: Uses HTML buttons in a CSS Grid layout
- **Game logic**: State stored in a `state` array (9-element board), `current` player, `gameOver` flag
- **Win detection**: `checkWin()` function checks against `LINES` array (rows, columns, diagonals)
- **Score tracking**: `scores` object (X, O, D) synced to DOM on each game event
- **Event-driven**: Button clicks trigger `play(index)`, reset triggers `init()`

### Shooter (`shooter.html`)

**Architecture patterns:**

1. **Procedural Pixel Art Sprites**
   - `PAL` object: color palette mapping single-char tokens to hex values
   - `sprite()` function: converts 2D string arrays into pixel grids (array of color values)
   - Each sprite is a 2D array with character tokens; rendered via `drawSprite(grid, x, y, pixelSize)`

2. **Canvas Game Loop**
   - `requestAnimationFrame` with fixed `dt` (clamped to 33ms for tab-switch resilience)
   - `update(dt)`: State machine dispatch + entity logic
   - `render()`: Clear screen, draw background grid, entities (sorted by y for depth illusion), HUD, overlays

3. **Game State Machine**
   - States: `MENU`, `LEVEL_INTRO`, `PLAYING`, `LEVEL_COMPLETE`, `GAME_OVER`, `WIN`
   - Accessed via global `state` variable; state-specific rendering and logic in `update()`

4. **Entity System**
   - **Player**: position (x, y), velocity, HP, fire cooldown, walk frame (animation), aim angle
   - **Enemies**: Two types — Walkers (slow, red, direct steering) and Runners (fast, pink, zigzag evasion)
   - **Bullets**: Linear projectiles with 0.8s lifetime
   - **Particles**: Short-lived visual feedback on hits and deaths
   - All stored in plain arrays; dead entities filtered each frame

5. **Input System**
   - Keyboard: ArrowUp/Down/Left/Right and WASD for movement
   - Mouse: Position tracked in `mouseX`, `mouseY` (canvas-relative); `mouseDown` for firing

6. **Audio (Web Audio API)**
   - `AudioContext` created on first user click (browser autoplay rules)
   - `tone()`: Oscillator + envelope for pitched sounds (shoot, death, level-clear, game-over)
   - `noise()`: White-noise burst for hit feedback
   - Accessed via `sfx.<event>()` (e.g., `sfx.shoot()`, `sfx.hit()`)

7. **Collision Detection**
   - Circle vs. circle: radius-based distance checks
   - Bullet-to-enemy: checked per frame, removes bullet and damages/kills enemy
   - Enemy-to-player: damage + invulnerability frames + screen shake

8. **Level Configuration**
   - `WAVES` array: 5 levels with `walkers`, `runners`, `spawnInterval` counts
   - Level clears when all enemies spawned and array is empty
   - Enemy spawn points: just outside screen edges

**Key code sections:**
- Lines ~50–80: Palette and sprite definitions
- Lines ~100–150: Input and audio setup
- Lines ~150–250: Sprite rendering and entity draw functions
- Lines ~250–400: Entity update logic (player movement, enemy AI, bullet collision, damage)
- Lines ~400–500: Game state machine (menu, playing, level progression)
- Lines ~500–600: Main game loop (update + render)
- Lines ~600–end: State-specific render functions (menu, HUD, game over, etc.)

## Development & Git Workflow

**CRITICAL: Commit and push regularly** — After each meaningful change or completed task, commit to Git with a clean message and push to GitHub immediately. This ensures we never lose work and can always revert if needed.

**Standard workflow for each change:**

```bash
# Make changes to a game file
# Test in browser (open the .html file)

# Stage and commit with descriptive messages
git add <file>
git commit -m "Brief description of change

Detailed explanation if needed:
- What changed
- Why it changed"

# Push to GitHub immediately
git push origin master
```

**Commit message style:**
- First line: imperative, under 50 chars (e.g., "Add runner enemy type", "Fix collision detection")
- Blank line, then detailed explanation (bullets OK for multiple changes)
- Each game is committed separately; UI/gameplay changes are grouped by feature

**Golden rule:** Always push after committing. A local commit is not backed up until it reaches GitHub.

**Remote:** `origin` → `https://github.com/ashlynl8/retro-browser-games.git`

## Testing & Verification

Manual browser testing is the only verification needed:
- Open the `.html` file directly in a browser (double-click or drag into browser tab)
- Test the happy path and edge cases (e.g., win, lose, restart)
- Check browser console for errors (F12 → Console tab)
- No crashes or console errors = tests pass

## Common Tasks

**Important: Web Audio Setup (Shooter)**
- `AudioContext` is created on first user click (browser autoplay policy)
- All `sfx.event()` calls must come *after* initial user interaction
- If audio doesn't play, check that the user has clicked before the SFX call

**Adding a new feature to Shooter:**
1. Identify which section: sprite definitions (lines ~80–120), entity logic (~250–400), state machine (~400–500), or rendering (~600–end)
2. Add sprites if needed (2D string array + `sprite()` call)
3. Add/modify entity update logic
4. Test by opening `shooter.html` in browser
5. Commit and push

**Fixing a bug:**
1. Open game in browser; reproduce the issue
2. Locate the relevant code section (input, collision, state machine, etc.)
3. Fix inline; test immediately in browser
4. Commit with a clear description of what was broken and why it's fixed

**Adding a new level:**
1. Add entry to `WAVES` array with `walkers`, `runners`, `spawnInterval`
2. Test: game should clear when all enemies spawned and defeated
3. Adjust difficulty by tweaking counts and spawn intervals
