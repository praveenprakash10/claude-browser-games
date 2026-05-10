# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the games

Both games are self-contained HTML files with no build step. Open directly in a browser:

```bash
open shooter.html
open tictactoe.html
```

No server, no dependencies, no install step. After edits, hard-refresh the browser (`Cmd+Shift+R`).

## Repository layout

| File | Description |
|---|---|
| `shooter.html` | Cyber Siege — top-down shooter, ~960 lines, all logic in one `<script>` |
| `tictactoe.html` | Tic Tac Toe — 2-player + minimax AI, ~250 lines |

## shooter.html architecture

The entire game lives in a single `<script>` tag, sectioned with `// ── SECTION ──` banners. Reading order matters — each section depends on the ones above it.

**Execution order / dependency chain:**
```
Canvas + resize globals (W, H)
  → Constants (PLAYER_SPEED, FIRE_RATE, color palette C{})
  → Input state (keys{}, mouse{})
  → Particle class + burst() helper
  → Pure draw functions (no state, take entity as arg)
  → Entity classes: Player, Enemy, Bullet
  → LEVELS[] config array
  → G{} game state object + loadLevel() / startGame()
  → update(dt) — mutates G
  → render() — reads G, calls draw functions
  → rAF game loop
```

**Global state — everything lives on `G`:**  
`G.state` drives what `update()` and `render()` do. Valid states: `'MENU'`, `'PLAYING'`, `'LEVEL_COMPLETE'`, `'GAMEOVER'`, `'VICTORY'`.  
`G.queue[]` is the shuffled spawn queue for the current level — enemies are shifted off it on a timer.

**Sprite drawing is pure / stateless:**  
`drawPlayer(p)`, `drawGrunt(e)`, `drawRunner(e)`, `drawTank(e)`, `drawBoss(e)` receive the entity and draw relative to `(0,0)` — callers do `ctx.save(); ctx.translate(e.x, e.y); ... ctx.restore()`. Never read from `G` inside draw functions.

**Adding a new enemy type:**
1. Add a `drawXxx(e)` sprite function
2. Add a branch in `drawEnemySprite(e)`
3. Add a config entry in `ENEMY_CFG` (`hp`, `spd`, `r`, `dmg`, `pts`)
4. Reference the type string in a level's `pool` array

**Adding a new level:**  
Append an object to `LEVELS[]`. Shape: `{ name, interval, pool: string[], hasBoss: bool }`. `hasBoss: true` means the boss spawns after all `pool` enemies are dead; the boss type is always `'boss'`.

**Physics / timing:**  
All movement is `position += velocity * dt` (delta-time in seconds, capped at 0.05 to survive tab-blur). `fireCd`, `iframes`, `bossShootCd` are all in seconds, decremented by `dt` each frame.

**Collision:** circle–circle only — `circHit(ax,ay,ar, bx,by,br)`. Enemy contact radius is scaled to `e.radius * 0.78` to feel fair.

## Git workflow

Every meaningful change should be committed and pushed:

```bash
git add <files>
git commit -m "descriptive message"
git push
```

Remote: `https://github.com/praveenprakash10/claude-browser-games` (main branch).
