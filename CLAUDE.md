# Riverland Yards — The Muster

A single-file Three.js game (r128 via cdnjs, no build step). Player rides a black-and-white
horse (Fant) mustering 8 cattle into stock yards in the rain, set on a Wairarapa riverland
farm in NZ. Includes a "Boss fight" second phase: on-foot chase to catch a ewe before she
reaches the river.

## Run / deploy
- Open `index.html` directly in a browser, or `npx serve` for a local server.
- Deployed via GitHub Pages (root `index.html`). Push to main = live.
- Only external dependency: Three.js r128 from cdnjs. No npm, no bundler. Keep it that way
  unless explicitly asked.

## File layout (all in index.html)
- `<style>` — HUD, intro card, win overlay, boss cutscene (`#boss`, SVG portrait of "Unc"),
  boss end overlay, mobile joystick (`#stick`, shown via pointer:coarse media query).
- `<script>` sections in order, marked with `// ============ NAME ============` banners:
  SCENE → TERRAIN → NATIVE BUSH → RIVER STONES → ATV WHEEL RUTS → PEN → build helpers →
  FANT + RIDER → CATTLE → ON-FOOT PLAYER & THE EWE → INPUT → PEN COLLISION → GAME LOOP →
  BOSS FIGHT → resetGame.

## Key architecture
- **Terrain**: `terrainH(x,z)` is the single source of truth for ground height everywhere
  (entities, bush, ruts all sample it). River channel is carved inside it via `riverX(z)`.
  ATV track path = `trackZ(x)`. Terrain colours are per-vertex, set once at build
  (grass/mud/shingle/track corridor).
- **Yards**: axis-aligned box, consts `PX1,PX2,PZ1,PZ2`, gate gap `GX1..GX2` on the +z wall.
  `collidePen(oldX, oldZ, entity)` blocks wall crossings (zeroes vx/vz if present);
  `inPen(x,z)` tests containment. Gate mesh swings shut on win via `gateTarget`.
- **Cattle AI** (in GAME LOOP, `// --- cattle ---`): boids-style — fear of horse (radius 24,
  scaled by horse speed `spdF`), cohesion/separation, river avoidance, fence repulsion
  (margin 2.5), and a gate funnel that only pulls when `pressured` by the player. Cows
  settle (damped) once inside. Tune difficulty with fear radius, funnel strength, `max` speed.
- **Vegetation**: fully instanced — four `InstancedMesh` pools (trunkPool, blobPool,
  conePool, frondPool) + per-instance colours. Species are placement functions
  (`manukaTree` dominant, `cabbageTree`, `ponga`, `totara`, `scrub`). Stones are one
  InstancedMesh (900). When adding vegetation, use the pools; don't add per-tree Groups.
- **Modes**: global `mode` = `'ride'` | `'boss'`. The game loop branches movement, camera,
  and breath on it. Boss flow: `startBoss()` (cutscene) → `beginChase()` → `updateFoot()` /
  `updateEwe()` → `endChase(win)`. Ewe: grazes until player within 18m, then flees toward
  `riverX` with jinks + panic burst; caught at dist < 1.7; escapes at `x < riverX(z)+6`.
- **Atmosphere**: rain = LineSegments respawning around camera with wind; breath = pooled
  sprites (`puff()`); fog = FogExp2 0.0085.

## Conventions / gotchas
- Three.js r128: no CapsuleGeometry, no OrbitControls import — stick to primitives.
- All state resets go through `resetGame()`; if you add state, reset it there.
- Timer is gated by `timerRunning`, not by mode flags.
- Everything is metric-ish world units ≈ metres; horse max 15, foot 7.6, ewe 5.9(+1.7 panic),
  cattle max 8.
- Mobile: single virtual joystick (throttle = y, steer = x). Test any control change against
  both keyboard and touch paths (`handleTouch`, `keys{}`).
- z-fighting: avoid coplanar box faces on the character models (rear horse patch was moved
  +0.03 for this reason).

## Backlog ideas (owner: Nick)
- Sound: rain loop, hoofbeats in mud, cattle, gate clunk (Web Audio, no assets = synth or
  small local files).
- Heading dog with whistle commands.
- Gallop camera shake; puddle reflections; lens rain.
- Difficulty tuning pass on cattle funnel + ewe chase.
