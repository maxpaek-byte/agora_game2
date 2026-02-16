# CLAUDE.md

## Project Overview

**Snake Game** is a single-file, client-side snake game built with vanilla HTML, CSS, and JavaScript. Created as a class project for an Agora course. The interface is in Korean.

Players control a snake using arrow keys (or WASD / on-screen D-pad) to eat food and grow longer. The snake speeds up as the score increases. The game ends when the snake hits a wall or itself.

## Repository Structure

```
agora_game2/
├── CLAUDE.md            # This file — AI assistant guide
├── README.md            # Brief project description (Korean)
└── index.html           # Entire application (HTML + CSS + JS in one file)
```

## Tech Stack

- **HTML5 / CSS3 / Canvas / Vanilla JavaScript** — no frameworks, no dependencies
- **Supabase** (`@supabase/supabase-js` v2 via CDN) — leaderboard persistence
- **Google Fonts**: JetBrains Mono (monospace), Space Grotesk (headings/buttons)
- **No build system** — no package.json, no bundler, no transpiler
- **No testing framework** — no unit or integration tests
- **No linter or formatter** configured

## How to Run

Open `index.html` directly in any modern web browser. No server, build step, or installation required.

## Architecture

### Single-File Design

All code lives in `index.html` (~302 lines):
- **Lines 1–7**: HTML head, meta tags, Google Fonts import
- **Lines 8–107**: `<style>` block — all CSS (variables, canvas, D-pad, animations)
- **Lines 109–302**: `<body>` with `<script>` — all game logic

### CSS Custom Properties (`:root`)

| Variable        | Purpose                  | Value             |
|-----------------|--------------------------|-------------------|
| `--bg`          | Page background          | `#07070d`         |
| `--surface`     | Card/panel background    | `#101018`         |
| `--accent`      | Primary accent (teal)    | `#6ee7b7`         |
| `--error`       | Food color (red)         | `#f87171`         |
| `--gold`        | Best score color         | `#fbbf24`         |
| `--tile`        | Canvas background        | `#161626`         |

### Game State Machine

States flow: `idle` → `playing` → `gameover`

| State      | Description                                    |
|------------|------------------------------------------------|
| `idle`     | Start screen with play button                  |
| `playing`  | Snake moves on interval; player controls direction |
| `gameover` | Snake hit wall or itself; final score displayed |

### Key Global Variables

| Variable      | Type       | Description                              |
|---------------|------------|------------------------------------------|
| `state`       | string     | Current game state                       |
| `score`       | number     | Current session score                    |
| `bestScore`   | number     | Highest score (in-memory only)           |
| `speed`       | number     | Current interval delay in ms (starts 150)|
| `snake`       | {x,y}[]   | Array of snake segments (head at index 0)|
| `dir`         | {x,y}      | Current movement direction               |
| `nextDir`     | {x,y}      | Buffered next direction (prevents reversal) |
| `food`        | {x,y}      | Current food position                    |
| `loop`        | number     | setInterval reference                    |
| `playerName`  | string     | Player name (persisted in localStorage)  |
| `leaderboard` | object[]   | Cached top-10 scores from Supabase       |

### Core Functions

| Function            | Purpose                                              |
|---------------------|------------------------------------------------------|
| `render()`          | Rebuilds UI via `innerHTML` based on state; calls `drawCanvas()` when playing |
| `drawCanvas()`      | Draws grid lines, food (circle with glow), and snake (gradient body with glow head) on canvas |
| `spawnFood()`       | Places food at random unoccupied cell                |
| `startGame()`       | Resets all state, spawns food, starts game loop       |
| `setDir(dx, dy)`    | Buffers direction change (prevents 180° reversal)    |
| `tick()`            | Moves snake, checks collisions, handles eating       |
| `gameOver()`        | Stops game loop, switches to gameover state, saves score |
| `fetchLeaderboard()`| Fetches top-10 scores from Supabase                  |
| `saveScore()`       | Inserts score record into Supabase                    |
| `leaderboardHTML()` | Generates leaderboard table HTML                      |
| `updateName()`      | Updates player name and persists to localStorage      |

### Game Mechanics

- **Board size**: 20x20 cells, 17px per cell (canvas: 340x340)
- **Initial snake**: 3 segments starting at (10,10) moving right
- **Initial speed**: 150ms per tick
- **Speed increase**: -3ms per food eaten (minimum 60ms)
- **Scoring**: +10 points per food
- **Controls**: Arrow keys, WASD, or on-screen D-pad (mobile-friendly)
- **Collision**: Wall hit or self-intersection ends the game
- **Best score**: tracked in memory; all scores persisted to Supabase
- **Leaderboard**: top-10 scores fetched from Supabase on load and after each game
- **Player name**: stored in `localStorage` (`snake_player_name` key)

### Rendering

- The game board uses HTML5 `<canvas>` with `CanvasRenderingContext2D`
- Snake body has a gradient from head (bright teal) to tail (darker)
- Snake head has a glow effect (`shadowBlur`)
- Food is drawn as a red circle with glow
- Grid lines are drawn as subtle borders

## Development Conventions

### When Modifying This Project

- All code is in a single file — keep it that way unless the project scope grows significantly
- The UI panels (idle/gameover) render by replacing `innerHTML` of `#app`; the game board uses `<canvas>`
- CSS uses custom properties defined in `:root` — use them for consistency
- The interface language is Korean — maintain Korean strings for user-facing text
- No semicolons are omitted in JS — keep semicolons consistent
- Inline `onclick` / `ontouchstart` handlers are used on HTML elements generated in `render()`

### Code Style

- Compact CSS: single-line declarations for simple rules
- Template literals for HTML generation inside `render()`
- Minimal variable names (e.g., `$` for `querySelector`)
- No modules or imports — everything is in global scope

### Adding Features

- New game states should be added to the state machine and handled in `render()`
- Canvas drawing logic goes in `drawCanvas()` or new helper functions
- D-pad buttons are styled with `.dpad-btn` — add new controls there
- Animations are defined as `@keyframes` at the bottom of the `<style>` block
- Supabase client is initialized at the top of `<script>`; use `supabase.from('scores')` for DB operations

## Supabase Setup

The game uses a `scores` table in Supabase. Create it with the following SQL in the Supabase SQL Editor:

```sql
CREATE TABLE scores (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  player_name TEXT NOT NULL DEFAULT '익명',
  score INT NOT NULL,
  snake_length INT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE scores ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Anyone can read scores" ON scores FOR SELECT USING (true);
CREATE POLICY "Anyone can insert scores" ON scores FOR INSERT WITH CHECK (true);
```

## Deployment

The game can be deployed by serving `index.html` from any static file host. No build step needed.
