# The Adventures of Steven: Road Warrior — Claude Code Reference

## Project Overview

Single-file HTML5 canvas game. Everything lives in `index.html` (~1940 lines). No build system, no dependencies, no modules. Edit the file and open/refresh in a browser to test.

Deployed at: `asharkinwater.github.io/st...` (GitHub Pages, push index.html to repo to deploy)

---

## File Structure

```
index.html          ← the entire game
game.html           ← old backup, do not edit
CLAUDE.md           ← this file
README.md           ← player-facing docs
```

### HTML Structure (lines 35–46)

```html
<div id="wrap">          <!-- CSS-scaled game container -->
  <canvas id="c"></canvas>
  <div class="scanline"></div>
  <div id="ui">
    <div id="hud"></div>
    <div id="chapter-banner"></div>
  </div>
</div>
<div id="dialog-layer">  <!-- OUTSIDE #wrap — never transformed with the canvas -->
  <div id="msg-box"></div>
  <div id="choices"></div>
</div>
```

**Critical:** `#dialog-layer` must stay outside `#wrap`. `position:fixed` inside a CSS-transformed parent breaks — putting it outside means it's never rotated or scaled with the canvas.

---

## Canvas & Layout

- Canvas logical size: **800 × 520** (`W=800, H=520`)
- `layoutGame()` scales uniformly to fill the viewport with no rotation on any device
- `#wrap` gets `transform: translate(tx,ty) scale(s)` — no rotation ever
- `#dialog-layer` is positioned via JS to sit at the bottom of the scaled canvas footprint
- `isCoarse()` — true on touch/small screens (≤820px)
- `isPortrait()` — still defined but no longer used for layout logic

---

## Game Loop

```javascript
GS.t          // frame counter (increments every frame)
GS.frameCount // same as GS.t (alias used in some places)
```

`requestAnimationFrame(gameLoop)` → dispatches on `GS.screen`:
- `'title'` → `drawTitle()`
- `'map'` → `drawMap()`
- `'drive'` → `drawDrive()`
- `'minigame'` → init once on `_lastMG` change, then `update*()/draw*()`
- `'gameover'` → `drawGameOver()`
- `'win'` → `drawWin()`

---

## Game State (GS)

```javascript
GS = {
  screen, minigame,
  chapter,
  health, fuel, chips, money, reputation, miles, day,
  flags: { gainedPowers, lostClothes, ninjasDefeated, huntingDone, llnlVisited, becameGarbageMan },
  currentDest,       // index of destination currently driving TOWARD
  driveProgress, driveTarget, driveSpeed,
  mg: {},            // mini-game or drive state (reused)
  mgScore,
  awaitingChoice,    // true while an event dialog is open
  eventsFired,       // Set of event IDs that have fired (prevents repeats)
  t, frameCount,
}
```

### Destinations

```
Index  Name                  Special
  0    Bothell, WA           (start)
  1    Portland, OR
  2    Sacramento, CA
  3    Livermore, CA         auto-triggers LLNL mini-game on arrival
  4    Los Angeles, CA
  5    Las Vegas, NV
  6    Salt Lake City, UT
  7    Denver, CO
  8    Albuquerque, NM       delivery complete → Chapter 2
  9    Bothell, WA (return)  → Chapter 3 (trash mini-game)
```

`GS.currentDest` = index you are driving **toward** (not where you are).
`arrive()` increments `GS.currentDest` after each leg.

---

## Drive Events

### Trigger points

Each drive leg gets two random event trigger points at ~30% and ~63% of progress:

```javascript
eventPoints: [.3 + Math.random()*.08, .63 + Math.random()*.08]
```

Slot 0 (first point) → `triggerRandomDriveEvent(isFirstSlot=true)` → checks for scripted leg events first.  
Slot 1 → always random pool.

### Scripted (leg-based) events

Events with `leg: N` fire exactly once on the first slot of the drive where `GS.currentDest === N`.

| leg | Event           | Drive leg            |
|-----|-----------------|----------------------|
| 1   | turkey_hunt     | Bothell → Portland   |
| 2   | chp_chase       | Portland → Sacramento|
| 3   | lot_lizard_la   | Livermore → LA       |
| 4   | fire_la         | LA → Las Vegas       |
| 5   | utah_mormons    | Las Vegas → SLC      |
| 6   | snowboard_denver| SLC → Denver         |
| 7   | nm_aliens       | Denver → ABQ         |

**Rule:** no two scripted events may share a leg number. `EVENTS.find()` returns the first match.

### Random pool events

Events **without** a `leg` property go into the random pool. `oneTime:true` events are removed from the pool after firing.

| Event         | oneTime |
|---------------|---------|
| ninja_chips   | yes     |
| lost_clothes  | yes     |
| flat_tire     | yes     |
| weigh_station | yes     |
| radio trivia  | no (deck-shuffled) |

### triggerEvent() double-guard

```javascript
function triggerEvent(ev){
  if(!ev) return;
  if(ev.oneTime && ev.id) GS.eventsFired.add(ev.id); // stamp immediately
  ...
}
```

This prevents re-entry even if `triggerEvent` is called twice for the same event.

### Radio trivia

`_triviaQ` holds the current question until the player answers any choice. `advance()` sets `_triviaQ=null`. Pool reshuffles when exhausted. Questions are never silently consumed.

---

## Mini-Games

All mini-games share the same `GS.mg` object (reset on init). Initialized once via `_lastMG` sentinel.

| Key          | Mini-game            | Trigger                        |
|--------------|----------------------|--------------------------------|
| `'frogger'`  | Lot Lizard Escape    | lot_lizard_la event, choice    |
| `'hunt'`     | Turkey Hunt          | turkey_hunt event, choice      |
| `'llnl'`     | LLNL / laser fusion  | Auto on arrive at Livermore    |
| `'ninja'`    | Ninja Attack         | ninja_chips event, choice      |
| `'chp'`      | CHP Chase            | chp_chase event, choice        |
| `'mormon'`   | Mormon Escape        | utah_mormons event, choice     |
| `'alien'`    | UFO Fight            | nm_aliens event, choice        |
| `'trash'`    | Trash Man            | Auto on return to Bothell      |
| `'fire'`     | Hollywood Fire       | fire_la event, choice          |
| `'snowboard'`| Breckenridge Shred   | snowboard_denver event, choice |

### Snowboard timing

Uses `performance.now()` — not frame count:
- `startT` = `performance.now()` on init
- `endT` = `startT + 22000` (22 seconds)
- Each obstacle hit: `endT = Math.min(endT + 2500, now + 28000)`
- Win when `now >= endT`

---

## Input

### Keyboard
`keys[code]` — held state  
`kd[code]` — pressed-this-frame (cleared at end of each frame)

### Touch / Virtual Joystick
Canvas drag → arrow key simulation via `vj` object.  
Short tap → `Space` / `Enter`.  
`vjUpdate()` maps `vj.dx/vj.dy` directly to arrow keys — no coordinate swapping.  
`setPointerFromClient(x,y)` maps screen coords to canvas coords — no portrait adjustment needed since canvas is never rotated.

---

## Audio

```javascript
beep(freq, duration, type='square', vol=0.15)
successJingle()   // ascending arpeggio
failJingle()      // descending arpeggio
```

Uses Web Audio API (`AudioContext`). Created lazily on first user gesture.

---

## UI Helpers

```javascript
showMsg(text, good, continueFn)   // shows #msg-box with a CONTINUE button
triggerEvent(ev)                  // shows event dialog with choice buttons
showChapterBanner(num, title, sub, cb)  // full-screen chapter overlay, auto-hides after 3s
showHUD() / hideHUD() / updateHUD()
```

---

## Chapter Flow

1. **Chapter 1 — Road Warrior:** Drive Bothell → Portland → Sacramento → (Livermore LLNL) → LA → Las Vegas → SLC → Denver → ABQ. Deliver chips.
2. **Chapter 2 — The Long Road Home:** Auto-starts. Drive ABQ → Bothell.
3. **Chapter 3 — Trash Man:** Auto-starts. Trash mini-game. Win condition: `GS.flags.becameGarbageMan === true` while `GS.currentDest >= 9`.

---

## Key Invariants

- Never rotate `#wrap` or `#dialog-layer` — no `rotate()` in any transform
- `GS.currentDest` is the destination you're driving **toward**, not where you are
- `arrive()` increments `GS.currentDest` — check leg mapping accordingly
- `oneTime` events must be stamped into `GS.eventsFired` in `triggerEvent()` regardless of which code path triggered them
- `resetGame()` must reset `triviaPool`, `_triviaQ`, and `_lastMG`
- Do not add `leg` to random-pool events — `leg !== undefined` excludes them from the pool
