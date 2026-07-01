# Lights Out — Project Progress

A single-screen, browser-based educational mini-game where the player completes a
repeating number **pattern** by tapping the correct switch. Sci‑fi control-panel
theme (robot guide, glowing pipes, power button, energy current).

- **Status:** Playable end-to-end (4 levels → completion screen), **direct gameplay —
  no tutorial**. Visuals match the Figma design (`node 956-7170`) and success screen
  (`node 975-16315`).
- **Stack:** Single file — `index.html` (HTML + CSS + vanilla JS). No build step, no
  dependencies. Fonts (Exo 2, Bebas Neue, Lilita One) load from Google Fonts.
- **Last updated:** 2026-06-25

---

## How to run
Open `index.html` in any modern browser. (For audio to start, the player taps
**Start** — this unlocks the Web Audio context.)

---

## Layout & rendering
- Fixed **1920×1080** design canvas (`#canvas`), positioned top-left and centered
  with a JS `translate() + scale()` so it fits **any display size** (letterboxed,
  aspect preserved). Rescales on resize / orientation change / mobile address-bar
  (`visualViewport`).
- **Responsive extras:** touch handling (`touch-action`, no tap highlight,
  no user-zoom) and a **rotate-to-landscape hint** overlay for narrow portrait phones.
- All element sizes/positions match the Figma frame exactly (panel `1573.9×453`,
  options box `742×256`, text bar `1499w`, switch tiles `128×174`, option tiles
  `112.2×152.5`, power button `183×178`, etc.).

## Screens
| Screen | Purpose |
|---|---|
| `#screen-intro` | Title image + Start button (the tap also unlocks Web Audio) |
| `#screen-level` | Level intro markup (currently **unused** — Start goes straight to gameplay) |
| `#screen-question` | Main gameplay |
| `#screen-complete` | End screen: all 4 completed patterns + Play Again |
| `#transition` | Sci‑fi **blast-door** transition between levels |

---

## Game mechanics
- **Levels** (`ROWS_CONFIG`): 4 levels, each a **7-slot skip-counting sequence**.
  | Level | Sequence | Player fills | Options (distractor) |
  |---|---|---|---|
  | 1 | 2 4 6 8 10 12 14 | 10, 12, 14 | 12 10 16 14 (16) |
  | 2 | 3 6 9 12 15 18 21 | 15, 18, 21 | 21 24 18 15 (24) |
  | 3 | 5 10 15 20 25 30 35 | 20, 30, 35 | 16 20 35 30 (16) — **gap**: 25 prefilled, 20 empty |
  | 4 | 1 3 5 7 9 11 13 | 7, 11, 13 | 13 6 7 11 (6) — **gap**: 9 prefilled, 7 empty |
- Each level = `seq` (the 7 numbers) + `prefilled` (slot indices shown at start) +
  `options` (4 tiles, one a distractor). `TOTAL = 7`. The player taps the correct
  option to fill the non-prefilled slots **in order**; the same render path handles
  both consecutive (levels 1–2) and gapped (levels 3–4) layouts.
- **Scoring:** 15 (first try) / 10 (after a wrong try) / 5 (after a hint).
- **Wrong answer:** shake + red glow pulse + spark burst on the wrong switch; the
  vent bars flash red.
- **Hint** (Figma `node 1009-2060`): after **2** wrong taps the **bulb button** (right
  end of the bot bar) glows (`armed`). Tapping it reveals the *hint screen*: the
  pattern step shown as **"+N" hop arrows** above every switch pair, plus the correct
  option glowing. The bulb then dims (`used`); it re-arms if the player gets stuck on a
  later slot. Hint state resets on each new slot / level / restart (`resetHint`).
  - Art is **picked from `assets/`**: the bulb button is `assets/hint.svg` (navy circle +
    cyan border + bulb icon, per Figma), and the hop-arrow step is baked into each PNG:
    `arrow arc.png` = +2, `arrow arc 3.png` = +3, `arrow arc 2.png` = +5 — mapped to the
    level step in `HINT_ARROW_SRC`. Arrows are sized/positioned in JS (`buildHintArrows` /
    `switchCenterX`) to sit centred over the switch gaps and fit the band above the switches.
- **Correct, completed pattern:** success state (see below) → "Well done!" →
  blast-door transition to the next level (or the completion screen after level 4).

---

## Implemented features

### Entrance (per level, staggered "one by one")
robot bounces in → text bar opens → **main box powers on & opens** → blue pipe +
red power button appear together → switches **cascade/drop in** one by one → bot
gives the instruction ("Tap the correct switch.") → options box deploys out of the
panel → option tiles pop in. Each beat has its own sound.
- Switch cascade timing: `SWITCH_OFFSET_MS = 3850`, `SWITCH_STEP_MS = 340`.

### Switches & vent
- Tiles render as a metal rocker (cropped from `tile.png`) + a **Lilita One** number.
- Empty slots: cyan neon border; the active "next" slot has a soft cyan glow.
- **Vent bars** (top of panel) recolor by state: cyan (idle), green (correct),
  red (wrong).

### Power button & current (success feedback)
- Power button: **red** (off) → **green** (on) + pulse when the pattern is complete.
- On completion the panel border turns **green** (Figma success screen), the **vent
  bars flow green** (left→right toward the pipe), then a **green current** sweeps
  along the pipes — accompanied by an electric **surge SFX**. Held ~2.4s.

### Celebration & transition
- "Well done!" with a dancing robot, then a **blast-door** power-down/up transition
  swaps in the next screen.

### Dialogue
- Bot lines **typewriter** out with soft talk blips; options lock while the bot
  "speaks". Instruction is spoken right when the switches finish appearing.
- **No tutorial:** every level (including the first) just prompts "Tap the correct
  switch." then a randomized win line. The old Level-1 guided narration
  ("let us read the pattern together", option-highlight, `readPatternLvl1`) was removed.
- On a correct tap the placed number is spoken if an `audio/<n>.ogg` clip exists
  (`playNumberVoice` — clips present for 3, 4, 5, 7, 10, 11, 13; others silently skip).

### Audio (all synthesized via Web Audio API; master gain + compressor)
hover, click, correct, wrong, row-complete, entrance pops, panel power-on, options
deploy whoosh, power-down/up (transition), bot-bar open/close, switch-appear,
talk blips, the **current-flow surge**, and **menu-button hover/press** (PLAY and Play
Again now click like the switch/option tiles). Optional number-voice `.ogg` files are
referenced and degrade gracefully if missing.

---

## Assets
**All raster art is WebP, all video is WebM, vectors stay SVG.** The old PNG/GIF/MP4
originals have been removed (converted 2026-07-01 via `sharp` + `ffmpeg-static`; the
`robot dance` GIF became a 25-frame **animated WebP** — 702 KB → 254 KB — and
`end-video` is now VP9+Opus WebM — 1.37 MB → 0.96 MB). Total `assets/` ≈ 1.6 MB.
- `assets/` — `background.webp` (sci-fi panel-wall game background, 1920×1080).
- `assets/figma/` — `robot.webp`, `connector.webp` (pipes), `tile.webp`,
  `panel.svg`, `panel-green.svg` (success), `options-box.svg`, `textbox.svg`.
- `assets/` — `Red button.webp`, `Green button.webp`, `robot dance.webp` (animated),
  `title screen.webp` (3D title art), `play button.svg` (title-screen **PLAY** button),
  `hint.svg` (hint bulb button), `arrow arc.webp` / `arrow arc 2.webp` / `arrow arc 3.webp`
  (hint "+N" hops), `end-video.webm` (completion video). The end-screen **Play Again**
  button is built in CSS (themed chamfered cyan plate to match).
- SVGs are kept as vectors (converting them to WebP would rasterize and lose crispness).
- `audio/` — number/voice `.ogg` clips are referenced by name but **not currently
  present**; the game runs fine without them (synth SFX cover everything).

---

## Pending / ideas
- [ ] Add the number-voice `.ogg` files (or remove the references) for spoken numbers.
- [ ] On-screen score/streak display (score is tracked but not shown).
- [ ] Persist player progress / best score (e.g. `localStorage`).
- [ ] More levels / difficulty curve; optional timer.
- [ ] Audit/remove unused legacy assets in `assets/`.

---

## Recent changelog
- **Switched to skip-counting sequences + removed the tutorial (2026-06-30):** the 4
  levels are now 7-slot sequences (×2, ×3, ×5, odds) taken from the new design; play is
  direct (Start → Level 1 gameplay, no guided narration). Unified the render path
  (`seq` + `prefilled`) so consecutive and gapped levels share one code path; number
  voice is now generic (`playNumberVoice`). `TOTAL` 8 → 7.
- Implemented full Figma layout at native 1920×1080 with exact element sizes.
- Made the game responsive across all display sizes (+ portrait rotate hint).
- Added playful staggered entrance + per-beat SFX; typewriter dialogue.
- Power button: lightning-panel iteration → reverted to the **red/green** button per
  the latest Figma.
- Success: green panel border, **green** vent flow + pipe current (confined to the
  exact pipe shape), held longer and sped up for visibility.
- Added the **current-flow electric surge SFX** as an attention grabber.
