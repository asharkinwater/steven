# The Adventures of Steven: Road Warrior

A retro pixel-art browser game. Steven is a truck driver hauling 500 bags of potato chips from Bothell, WA to Albuquerque, NM — and everything that can go wrong will.

**Play it:** [asharkinwater.github.io](https://asharkinwater.github.io/st...)

---

## How to Play

Use the **map screen** to see your route, then hit **ENTER / tap** to start driving each leg. Events fire during drives — read the story and pick a choice.

### Controls

| Action | Keyboard | Mobile |
|--------|----------|--------|
| Move / steer | Arrow keys or WASD | Drag anywhere on canvas |
| Action / confirm | Space or Enter | Tap (short) |
| Navigate menus | Arrow keys | Tap buttons |

---

## The Route

```
Bothell, WA
    ↓
Portland, OR
    ↓
Sacramento, CA
    ↓
Livermore, CA  ← automatic stop: visit the National Ignition Facility
    ↓
Los Angeles, CA
    ↓
Las Vegas, NV
    ↓
Salt Lake City, UT
    ↓
Denver, CO
    ↓
Albuquerque, NM  ← deliver the chips!
    ↓
(Chapter 2: drive home)
    ↓
Bothell, WA  ← Chapter 3: Trash Man
```

---

## Stats

- **Health** — hits zero → game over
- **Fuel** — drains while driving; refueled on arrival at each city
- **Chips** — your cargo; some events cost bags
- **Money** — earn $150 per delivery leg, spend on repairs and fines
- **Reputation** — affects nothing mechanically yet, but matters spiritually

---

## Events

Two random events fire during each drive leg. Some are scripted to specific legs; others are drawn from a random pool. All oneTime events fire once per playthrough.

### Scripted Events (guaranteed per leg)

| Leg | Event |
|-----|-------|
| Bothell → Portland | Wild Turkey Spotted |
| Portland → Sacramento | CHP Chase |
| Livermore → LA | LA Truck Stop Incident |
| LA → Las Vegas | Hollywood Is On Fire! |
| Las Vegas → SLC | Missionaries On Your Bumper |
| SLC → Denver | Rocky Mountain Break (Snowboard!) |
| Denver → ABQ | UFO on I-25 |

### Random Pool Events (fire once)

- **Ninja Attack** — potato chip bandits drop from an overpass
- **Wardrobe Malfunction** — Steven loses every piece of clothing he owns
- **Blowout** — rear tire detonates on the highway
- **Weigh Station Drama** — the inspector has questions about those bags

### Radio Trivia

A shuffled deck of trivia questions — win cash by answering correctly, or focus on driving.

---

## Mini-Games

| Mini-game | How to reach it |
|-----------|----------------|
| Turkey Hunt | Choose "Hunt that turkey!" |
| Lot Lizard Escape (Frogger) | Choose "RUN AWAY!" at the truck stop |
| CHP Chase | Choose "Floor it!" |
| LLNL Laser Fusion | Automatic at Livermore |
| Ninja Fight | Choose "Fight them off!" |
| Mormon Escape | Choose "RUN!" |
| Hollywood Fire | Choose "FIGHT THE FIRE!" |
| Breckenridge Snowboard | Choose "Shred the mountain!" |
| UFO / Alien Fight | Choose "FIGHT THEM!" |
| Trash Man (final) | Automatic — Chapter 3 ending |

---

## Chapters

**Chapter 1 — Road Warrior**  
Haul 500 bags of potato chips across the American West.

**Chapter 2 — The Long Road Home**  
Chips delivered. Steven drives back to Bothell.

**Chapter 3 — Trash Man Extraordinaire**  
Steven becomes Seattle's greatest garbage collector. This is the ending.

---

## Tech

Single HTML file, no libraries, no build step. HTML5 Canvas + vanilla JS + Web Audio API. Works in any modern browser, on desktop and mobile.
