# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## Project: Adam's Dino-style Runner Game

### How to run

No build step. Serve the directory with any static file server, then open in a browser:

```bash
python3 -m http.server 8765
# → http://localhost:8765
```

The game requires being served over HTTP (not `file://`) so the browser can load the MP3.

### What's here

| File | Purpose |
|------|---------|
| `index.html` | Entire game — HTML + CSS + JS in one file |
| `sprites.svg` | Editable reference sprite sheet (not loaded by the game; canvas draws everything programmatically) |
| `SVERDY ADAM BALAGAN HOUSE 30 [New Mix]...mp3` | The song; game duration is tied to its length (~139.8 s) |

### Architecture of `index.html`

Everything lives in a single `<script>` block. There are no modules, no build tools, no dependencies.

**State machine** — `state` is one of `S.{LOADING, READY, PLAYING, DEAD, ENDING, CELEBRATION}`. The `update()` and `render()` functions branch on this.

**Coordinate system** — `y` increases downward (canvas default). Character positions use the *feet* as the anchor (`R.y = GND()`), and drawing functions extend upward from there (negative y offsets).

**`GND()`** — returns the ground y-coordinate as a function of `canvas.height`. Call it fresh each frame; don't cache it across resize events.

**Obstacle types** — two kinds stored in the shared `obstacles` array:
- `{ type:'ars', x, y, count, tall }` — ground obstacles (ערס caricature), `count` 1–3 characters wide
- `{ type:'rocket', x, y }` — flying obstacles at one of three fixed altitudes

**Drawing** — all sprites are drawn with Canvas 2D API calls (no images). `drawRunner`, `drawOneArs`, `drawRocketObs`, `drawHouse` each take a world `x,y` and use `ctx.save/restore` + `ctx.translate` internally.

**End sequence** — at `songDuration - 2` seconds (detected via `audio.currentTime`), `state` flips to `ENDING`, all obstacles are cleared, and the house slides in. When the house door reaches the runner, the runner fades out (`R.alpha`) and `beginCelebration()` fires.

**Audio** — a plain `<audio>` element. Song filename is URL-encoded in the `SONG` constant. Duration is read from `audio.loadedmetadata`; the hardcoded fallback is `139.781`.

**Mobile** — `touchstart` triggers `onTap()`. `canvas.width/height` track `innerWidth/innerHeight` and update on `resize`.

### Key constants to adjust

- `GRAV`, `JV()` — jump feel
- `randSpawn()` — obstacle frequency (frames between spawns)
- `spawnObstacle()` — 28% rocket / 72% ars ratio, and the three rocket altitudes
- `songDuration` fallback (line near top of script) — update if the MP3 changes
