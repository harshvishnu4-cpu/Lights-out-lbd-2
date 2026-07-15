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
| `#screen-intro` | Title image + **PLAY** button (first launch) OR **Play Again** button (after a completed run). The tap unlocks Web Audio. |
| `#screen-level` | Level intro markup (currently **unused** — Start goes straight to gameplay) |
| `#screen-question` | Main gameplay |
| `#screen-complete` | Completion **video** + a **happy `sfxWin` fanfare & confetti burst** as it appears; when the video ends the game returns to the title and reveals **Play Again** (`goToTitleWithPlayAgain`) |
| `#transition` | Sci‑fi **blast-door** transition between levels |

---

## Game mechanics
- **Levels** (`ROWS_CONFIG`): 4 levels, each an **8-slot skip-counting sequence**
  (5 prefilled + 3 to fill). The distractor is the "next" number just past the sequence.
  | Level | Sequence | Player fills | Options (distractor) |
  |---|---|---|---|
  | 1 | 2 4 6 8 10 12 14 16 | 12, 14, 16 | 16 12 18 14 (18) |
  | 2 | 3 6 9 12 15 18 21 24 | 18, 21, 24 | 24 18 27 21 (27) |
  | 3 | 5 10 15 20 25 30 35 40 | 25, 30, 35 | 35 25 45 30 (45) — **gap**: 40 prefilled at the end, slots 4–6 empty |
  | 4 | 1 3 5 7 9 11 13 15 | 11, 13, 15 | 15 11 17 13 (17) |
- Each level = `seq` (the 8 numbers) + `prefilled` (slot indices shown at start) +
  `options` (4 tiles, one a distractor). `TOTAL = 8`. The player taps the correct
  option to fill the non-prefilled slots **in order**; the same render path handles
  both consecutive (levels 1, 2, 4) and gapped (level 3) layouts. Every fill value has an
  `audio/<n>.ogg` clip so each correct pick is spoken.
- **Scoring:** 15 (first try) / 10 (after a wrong try) / 5 (after a hint).
- **Wrong answer:** feedback shows **only on the tapped option** — gentle shake + **amber**
  glow pulse + amber spark burst + a soft SFX. The wrong number is **not** placed in a slot
  and the switch panel doesn't react (no vent flash, no panel shake), and **no spoken line
  is replayed** (the bot text stays put) so the player can immediately try again. After 2
  wrongs the hint bulb glows, with a soft magical shimmer SFX (`sfxHintGlow`). No red anywhere.
- **Inactivity nudge**: if the player is idle ~10s during their turn, the 4 option tiles
  **bounce in a wave for ~5.6s** (`idleBounce` loops 4×1.4s / `#options-area.idle-nudge`,
  class removed after 5.9s) with a soft blip to draw them back. Only a real **tap/keypress**
  (`pointerdown`, `keydown`, `touchstart`) resets the ~10s countdown — **not mouse movement**,
  so idly moving the cursor doesn't cancel it. `resetIdle` is also called at the top of every
  turn from `renderQuestion` so the countdown reliably (re)starts. It only fires when the
  options are tappable and the row isn't full, is **not shown during the Level 1 tutorial**
  (`currentRow !== 0` — the +2 arc already guides there), and is **suppressed while the hint
  bulb is armed** (one thing pulses at a time).
- **Hint** (Figma `node 1009-2060`): the yellow bulb button (right end of the bot bar) is
  **hidden by default** (never on Level 1). It only appears once the player struggles: after
  **2** wrong taps on a slot, `armHint` **pops it in** (`hb-reveal`/`hbPopIn` — the same
  appearance animation) already `armed` → **bright, pulsing + glowing** (`hintArmed`: scale +
  yellow glow), with the `sfxHintGlow` shimmer. (Guarded by `hintArmed` so later wrong taps
  don't re-pop or re-play the sound.) Tapping it reveals the *hint screen*: just the **"+N"
  hop arrows** above the switch pairs — it does **not** glow the option placeholders. The bulb
  then dims (`used`). `resetHint` **hides it again** (`hb-hidden`) on each new slot / level /
  restart, so it re-pops the next time the player gets 2 wrongs.
  - Art is **picked from `assets/`**: the bulb button is `assets/hint-yellow.svg` (yellow
    bulb + border), and the hop-arrow step is baked into each PNG:
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
- Empty slots: plain cyan neon border (the "next" slot is no longer glow-highlighted).
- **Vent bars** (top of panel) recolor by state: cyan (idle), green (correct),
  red (wrong).

### Power button & current (success feedback)
- Power button: **red** (off) → **green** (on) + pulse when the pattern is complete.
- On completion the panel border turns **green** (Figma success screen), the **vent
  bars flow green** (left→right toward the pipe), then a **green current** sweeps
  along the pipes — accompanied by the recorded current-flow SFX (`sfxCurrentFlow`
  plays `audio/electricity.mp3`, a sharp zap, layered with `audio/energy.mp3`, a
  sustained ~12s surge). The long energy clip is cut by `stopCurrentFlow()` inside
  `playTransition` so it never bleeds into the next level. Held ~2.4s.

### Celebration & transition
- "Well done!" with a dancing robot, then a **blast-door** power-down/up transition
  swaps in the next screen.

### Dialogue
- Bot lines **typewriter** out with soft talk blips; options lock while the bot
  "speaks". Instruction is spoken right when the switches finish appearing.
- **Level 1 is a full guided tutorial** (levels 2–4 skip straight to "Tap the correct
  switch."). The choreography:
  1. "Let us start fixing the switches." → "These switches are in a pattern." → "Let us
     read the pattern together." — `readPattern` highlights each prefilled switch
     left-to-right and speaks its number (`read-pop` + `NUMBER_VOICES`: 2, 4, 6, 8, 10).
  2. "We are adding 2 at each step." — the `+2` hop arrows pop in **one by one**
     (`revealArrowsSeq(0,4)`): the 4 over the read pattern **plus the arc into the first blank
     slot**, so the blank arc reveals as part of the same animated sequence (it does **not**
     pop in on its own later).
  3. "We keep adding the same number!" — `glowArrows(0,3)` makes the 4 pattern arcs **pulse
     together** (the blank arc stays static for now).
  4. "Tap the correct switch to continue the pattern." — the pattern-arc glows switch off and
     the **already-visible +2 arc into the first blank slot lights up & glows** as the guide
     (`showBlankArc` just moves the glow onto it — no pop). The glow advances to the arc into
     the next blank slot on each correct pick, so it walks across the answer slots.
  - The tutorial arrows stay up for the whole level (`tutorialArrowsLocked` makes
    `resetHint` skip clearing them until the level is left).
  5. The **+2 hop ARC that points into the current blank slot** is the tap guide
     (`showBlankArc` — an arrow *arc*, not a plain arrow). At the tap prompt it is revealed and
     **glows** ("the answer goes HERE, +2"), while the pattern arcs behind it stay static. After
     each correct pick the glow **moves to the arc into the next blank slot** automatically, then
     clears when the level is solved. Level 1 only; it does not affect scoring. The options
     themselves **no longer glow** — the glowing arc at the blank is the single guide.
  - **Only one thing pulses at a time.** The pattern arcs are static; only the single +2 arc
    hopping into the *current* blank slot glows (`showBlankArc` clears every other arc's glow
    before glowing that one). The hint bulb is hidden on Level 1 entirely, so it can't compete.
    (On levels 2–4 the bulb pulses alone after 2 wrongs; the revealed hint shows only arcs — no
    option glow.)
- On a correct tap the placed number is **spoken** (`playNumberVoice` → `audio/<n>.ogg`).
  Clips are present for 1–16, 18, 20, 21, 24, 25, 30, 35 (`NUMBER_VOICE_FILES`), which
  covers every correct answer across all four levels; any unmapped number silently skips.

### Audio (all synthesized via Web Audio API; master gain + compressor)
hover, click, correct, wrong, row-complete, entrance pops, panel power-on, options
deploy whoosh, power-down/up (transition), bot-bar open/close, switch-appear,
**switch-flip-ON for a correct pick (`sfxSwitchOn` — clunk + upward toggle + electric
spark)**, talk blips, the **current-flow surge**, **menu-button hover/press** (PLAY
and Play Again click like the tiles), and a **happy "ta-da!" win fanfare (`sfxWin`) on
completing the whole game — plays with an extra confetti burst**. The wrong-answer SFX is a
**soft two-note triangle "aw"** (not a harsh buzz). Optional number-voice `.ogg` files are
referenced and degrade gracefully if missing.
- **No background music** — the game runs on SFX only (the synth ambient BGM was removed).
  The completion `end-video.webm` still carries its own audio.

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
  `title screen.webp` ("LIGHTS OUT!" title art), `play button.svg` (title-screen **PLAY**
  button — a bold GSAP breathing pulse; a **spark burst** fires from it on tap, then the
  blast-door transition starts the game), `hint-yellow.svg` (yellow hint bulb button),
  `arrow arc.webp` / `arrow arc 2.webp` / `arrow arc 3.webp`
  (hint "+N" hops), `end-video.webm` (completion video), `play again.svg` (the **Play Again**
  button shown on the title screen after a completed run — same pulse + click-spark as PLAY).
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
- Added the **current-flow SFX** (recorded zap `electricity.mp3` + sustained
  `energy.mp3` surge) as an attention grabber; stopped on level transition.
