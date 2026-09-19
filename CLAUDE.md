# CLAUDE.md — Legible Platformer

## Project Overview

A simple 2D platformer game written in the **Legible** programming language (`.lbl` files), running on the Legible interpreter located at `../legible/`. The interpreter has been extended with SDL2 bindings to support window creation, rendering, input handling, and timing.

## How to Build and Run

```bash
# Build the interpreter (from the legible directory)
cd ../legible && cargo build --release

# Run the game
../legible/target/release/legible run game.lbl
```

## Architecture

```
legible-platformer/
├── CLAUDE.md           # This file
└── game.lbl            # Complete platformer game
```

The game is a single `.lbl` file containing all records, game logic, physics, collision detection, and rendering.

## Game Design

### Records
- `Player` — position, velocity, dimensions, ground state
- `Platform` — rectangular platforms (x, y, width, height, color)
- `GameState` — player, platforms, camera offset, running flag

### Game Loop
Standard fixed-timestep loop:
1. `sdl_poll_events()` — gather input events
2. Update keyboard state (tracked via `sdl_is_key_pressed`)
3. Apply physics — gravity, horizontal movement, jumping
4. Collision detection — player vs platforms (AABB)
5. Render — clear screen, draw platforms, draw player
6. `sdl_present()` + `sdl_delay(16)` for ~60 FPS

### Controls
- **Left/Right Arrow** — move horizontally
- **Space** — jump (when on ground)
- **Escape / window close** — quit

## SDL2 Builtins Available

These functions are built into the Legible interpreter (`../legible/src/interpreter/sdl_builtins.rs`):

| Function | Signature | Purpose |
|----------|-----------|---------|
| `sdl_init` | `(title: text, width: integer, height: integer): nothing` | Initialize SDL2 window + renderer |
| `sdl_poll_events` | `(): a list of text` | Poll all pending events, returns event names |
| `sdl_is_key_pressed` | `(key: text): boolean` | Check if a key is currently held down |
| `sdl_clear` | `(r: integer, g: integer, b: integer): nothing` | Clear screen with color |
| `sdl_fill_rect` | `(x: integer, y: integer, w: integer, h: integer, r: integer, g: integer, b: integer): nothing` | Draw filled rectangle |
| `sdl_present` | `(): nothing` | Present the rendered frame |
| `sdl_delay` | `(ms: integer): nothing` | Sleep for milliseconds |
| `sdl_get_ticks` | `(): integer` | Get milliseconds since SDL init |
| `sdl_quit` | `(): nothing` | Clean up SDL resources |

### Key Names for `sdl_is_key_pressed`
Use SDL scancode names: `"Left"`, `"Right"`, `"Up"`, `"Down"`, `"Space"`, `"Escape"`, `"A"` through `"Z"`, `"Return"`, etc.

### Event Names from `sdl_poll_events`
- `"quit"` — window close button pressed
- `"key_down:KeyName"` — key pressed
- `"key_up:KeyName"` — key released

## Legible Language Constraints

- Functions are limited by a comprehension budget (Halstead volume ≤ 1000, cyclomatic ≤ 10, cognitive ≤ 15, ≤ 9 live bindings), not a line count — split logic into small focused functions
- Every function needs an `intent:` line
- Records are immutable — use `record with { field: value }` for updates
- Mutable variables use `mutable` keyword + `set` for reassignment
- String concat uses `++`, not `+`
- No `break`/`continue` — use functional patterns or flags
- Newlines are statement terminators
