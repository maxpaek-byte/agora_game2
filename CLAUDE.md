# CLAUDE.md

## Project Overview

**Memory Grid** is a single-file, client-side memory pattern game built with vanilla HTML, CSS, and JavaScript. Created as a class project for an Agora course. The interface is in Korean.

Players memorize highlighted tile positions on a 4x4 grid and reproduce the pattern by clicking tiles. Difficulty increases each level with more tiles and shorter display times.

## Repository Structure

```
agora_game2/
├── CLAUDE.md            # This file — AI assistant guide
├── README.md            # Brief project description (Korean)
└── index.html           # Entire application (HTML + CSS + JS in one file)
```

## Tech Stack

- **HTML5 / CSS3 / Vanilla JavaScript** — no frameworks, no dependencies
- **Google Fonts**: JetBrains Mono (monospace), Space Grotesk (headings/buttons)
- **No build system** — no package.json, no bundler, no transpiler
- **No testing framework** — no unit or integration tests
- **No linter or formatter** configured

## How to Run

Open `index.html` directly in any modern web browser. No server, build step, or installation required.

## Architecture

### Single-File Design

All code lives in `index.html` (~294 lines):
- **Lines 1–7**: HTML head, meta tags, Google Fonts import
- **Lines 8–147**: `<style>` block — all CSS (variables, grid layout, animations)
- **Lines 149–294**: `<body>` with `<script>` — all game logic

### CSS Custom Properties (`:root`)

| Variable        | Purpose                  | Value             |
|-----------------|--------------------------|-------------------|
| `--bg`          | Page background          | `#07070d`         |
| `--surface`     | Card/panel background    | `#101018`         |
| `--accent`      | Primary accent (teal)    | `#6ee7b7`         |
| `--error`       | Failure indicator (red)  | `#f87171`         |
| `--gold`        | Combo/best score color   | `#fbbf24`         |
| `--tile`        | Default tile color       | `#161626`         |

### Game State Machine

States flow: `idle` → `showing` → `input` → `success`/`fail` → (repeat or `gameover`)

| State      | Description                                    |
|------------|------------------------------------------------|
| `idle`     | Start screen with play button                  |
| `showing`  | Pattern tiles glow for memorization            |
| `input`    | Player clicks tiles (supports deselection)     |
| `checking` | Validating player input                        |
| `success`  | Correct answer — brief feedback then next level|
| `fail`     | Wrong answer — lose a life, shake animation    |
| `gameover` | All 3 lives lost, final score displayed        |

### Key Global Variables

| Variable      | Type     | Description                          |
|---------------|----------|--------------------------------------|
| `state`       | string   | Current game state                   |
| `level`       | number   | Current level (starts at 1)          |
| `pattern`     | number[] | Tile indices to memorize             |
| `playerInput` | number[] | Tiles the player has clicked         |
| `score`       | number   | Current session score                |
| `bestScore`   | number   | Highest score (in-memory only)       |
| `lives`       | number   | Remaining lives (starts at 3)        |
| `combo`       | number   | Consecutive correct answer streak    |
| `timer`       | number   | setTimeout reference                 |

### Core Functions

| Function            | Purpose                                              |
|---------------------|------------------------------------------------------|
| `genPattern(lvl)`   | Generates random non-repeating tile indices; count = `min(3 + lvl, 12)` |
| `render()`          | Rebuilds entire UI via `innerHTML` based on current state |
| `startGame()`       | Resets all state and starts level 1                  |
| `startLevel(lvl)`   | Generates pattern, shows it, then transitions to input after timeout |
| `cellClick(i)`      | Handles tile toggle (select/deselect); auto-checks when count matches |
| `checkAnswer()`     | Compares sorted arrays; updates score, combo, lives  |

### Game Mechanics

- **Grid size**: 4x4 (16 tiles)
- **Pattern length**: `min(3 + level, 12)` tiles
- **Show duration**: `max(900ms, 2400ms - level * 100ms)` — gets faster each level
- **Scoring**: `level * 100 * (1 + combo * 0.2)` points per correct answer
- **Lives**: 3 total; wrong answer costs 1 life and resets combo
- **Best score**: tracked in memory only (not persisted across sessions)

## Development Conventions

### When Modifying This Project

- All code is in a single file — keep it that way unless the project scope grows significantly
- The UI renders by replacing `innerHTML` of `#app` on every state change (no virtual DOM, no diffing)
- CSS uses custom properties defined in `:root` — use them for consistency
- The interface language is Korean — maintain Korean strings for user-facing text
- Game messages are in the `messages` array: `['완벽해요!','훌륭해요!','대단해요!','천재!','놀라워요!']`
- No semicolons are omitted in JS — keep semicolons consistent
- Inline `onclick` handlers are used on HTML elements generated in `render()`

### Code Style

- Compact CSS: single-line declarations for simple rules
- Template literals for HTML generation inside `render()`
- Minimal variable names (e.g., `$` for `querySelector`, `lvl` for level, `i` for index)
- No modules or imports — everything is in global scope

### Adding Features

- New game states should be added to the state machine and handled in `render()`
- New CSS classes for tile states go alongside `.cell.active`, `.cell.selected`, `.cell.correct`, `.cell.wrong`
- Animations are defined as `@keyframes` at the bottom of the `<style>` block
- To persist best scores, consider `localStorage`

## Deployment

The game can be deployed by serving `index.html` from any static file host. No build step needed.
