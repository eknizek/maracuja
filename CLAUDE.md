# Mini Golf

A browser-based top-down 2D mini golf game built collaboratively with **David, the user's 10-year-old son** (no prior coding experience). Dad has coding experience and writes most of the code; David owns creative/design decisions (colors, hole layout, what to add next, tuning numbers).

## Collaboration notes

- David is learning by seeing code → immediate visual feedback. Keep changes small and observable.
- Favor beginner-readable code: clear variable names, minimal abstractions, no build step, no frameworks.
- When suggesting next steps, prefer ones David can drive (tweak constants, design a new hole, pick colors) over big invisible refactors.
- The `HOLES` array is intentionally simple data so David can add or modify holes by editing numbers.

## Tech setup

- Single file: `index.html`
- p5.js loaded from CDN (no install, no build, no package.json)
- Run: double-click `index.html`, or `open index.html` from the project directory

## Architecture

Everything lives in `index.html`. The `<script>` block has three parts: data (the `HOLES` array + tunable constants), state (current hole, ball, strokes, etc.), and p5 functions (`setup`, `draw`, plus helpers).

### Holes are data

Each hole is an entry in the `HOLES` array near the top of the script. Each entry has:

- `par`, `canvas: { w, h }`
- `tee`, `teeBox`
- `cup`, `green`
- `fairway` — array of line segments (each `{ x1, y1, x2, y2, width }`). Multiple segments = dogleg.
- `sand` — array of circles (`{ x, y, r }`)
- `water` — array of ellipses (`{ x, y, w, h }`)
- `wind` *(optional)* — `{ dx, dy }`: per-frame acceleration applied to the moving ball while its speed is above `WIND_THRESHOLD`. Only present on holes that should have wind.

The helper `hole()` returns `HOLES[currentHoleIndex]`. All draw and physics functions read from it. **To add a new hole, append a new entry to `HOLES`** — nothing else needs to change. Canvas resizes automatically between holes.

### Current holes

- **Hole #1** — Par 3, 800×500 canvas. Diagonal fairway from bottom-left tee to top-right green. Fairway bunker mid-course, water above the green, greenside bunker.
- **Hole #2** — Par 4, 1200×700 canvas. L-shaped dogleg-right fairway. Water hazard in the inside of the dogleg punishes cutting the corner. Fairway bunker at the dogleg, greenside bunker.
- **Hole #3** — Par 5, 1400×700 canvas. Long hole that bends up-and-to-the-right across three fairway segments. Eight sand traps scattered along the way plus a greenside bunker. **Has wind** (`{ dx: -0.012, dy: 0.010 }`) — a gentle quartering headwind that pushes the moving ball back toward the tee.
- **Hole #4** — Par 4, 1200×700 canvas. Narrow zig-zag (S-shape) fairway across three segments, width 80 (about half the other holes). Six water hazards: four on the outside of the bends and around the green, plus two large vertical ponds filling the inside-of-bend wedges so the player can't cut the corners.

### Physics constants (tunable)

- `FRICTION = 0.985` — fairway / green (default)
- `ROUGH_FRICTION = 0.955` — anywhere not in fairway, green, or sand
- `SAND_FRICTION = 0.94` — inside any sand trap
- `GREEN_PENALTY_FRICTION = 0.92` — applied when the ball is on the green AND the in-flight shot was hit with Driver/Iron/Wedge (forces use of Putter on the green)
- `WIND_THRESHOLD = 1.2` — wind only nudges the ball while its speed is above this (in px/frame). Prevents wind from sustaining infinite drift past terminal velocity.
- `BALL_R = 8`, `STOP_SPEED = 0.1`, `SINK_SPEED = 3.0`, `MAX_DRAG = 150`, `POWER_SCALE = 0.12`

### Clubs

Defined in the `CLUBS` object — each has a `power` multiplier:

- Driver `1.1`, Iron `1.0`, Wedge `0.55`, Putter `0.25`

`currentClub` is the player's selection (driven by the buttons below the canvas). `shotClub` is the club used for the in-flight shot — captured in `mouseReleased`, cleared when the ball stops. The green-penalty friction reads `shotClub`, not `currentClub`.

### Hazards & rules

- **Water** — ball returns to `lastShotPos`. **No stroke penalty** (David's design choice — keeps the game kid-friendly).
- **Out of bounds** (ball touches any canvas edge) — same as water: returns to `lastShotPos`, no stroke penalty.
- **Sand** — slows the ball (uses `SAND_FRICTION`). No penalty.
- **Rough** — slows more than fairway, less than sand (uses `ROUGH_FRICTION`). No penalty.
- **Wind** — per-hole optional. Applied as a per-frame velocity nudge only while `speed >= WIND_THRESHOLD`, so the ball can still come to a natural stop on slow rolls.
- **Sinking the ball** — ball center inside cup AND speed below `SINK_SPEED`. On sink, the current-hole win screen shows; clicking anywhere advances to the next hole (or restarts after the last).

### UI

- Aim: click on the ball, drag *away* (slingshot), release to shoot. Red arrow shows direction and power.
- HUD (top-left of canvas): Hole #, Par, Strokes, Club, Total (after hole 1).
- HUD (top-right of canvas, only on holes with wind): "Wind" label + a blue arrow showing the wind direction.
- Flag at each cup: a wooden pole with a red triangular flag. On holes with wind, the flag points in the wind direction with a gentle `sin(frameCount)` wave; on holes without wind it hangs limp.
- Club buttons (below canvas): Driver / Iron / Wedge / Putter — selected club is highlighted green.
- **"Skip Hole →" dev button** (dashed purple border): testing-only, jumps to the next hole without counting strokes. Safe to remove later if shipping.

### Win screen

After sinking the ball, a dark overlay shows the score label (big text), the per-hole stroke count, and a "click to continue" hint. Labels come from `scoreLabel(strokes, par)`:

- `strokes === 1` → **"Hole in One!"** — also triggers `drawFireworks()`, an animated 5-burst confetti display behind the text.
- `-3` or better (vs par) → "Albatross!"
- `-2` → "Eagle!"
- `-1` → "Birdie! 🎉"
- `0` → "Par 😊"
- `+1` → "Bogey 😢"
- `+2` → "Double Bogey 😢"
- `+3` → "Triple Bogey 😢"
- `+4` or worse → `"+N 😢"`

The `strokes === 1` check runs first, so a 1-stroke finish always shows Hole in One even if it would technically also qualify as Eagle/Albatross.

## Conventions

- Constants: `SCREAMING_SNAKE_CASE`. Runtime state: `camelCase`.
- Prefer named `function` declarations for top-level functions; arrow functions only inline.
- Comments are sparse and only used when the *why* isn't obvious from names (e.g., the perpendicular-vector math for tee markers).
- No git repository currently.
