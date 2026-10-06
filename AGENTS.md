# Paper Trench · 纸面战壕

A doodle-on-graph-paper real-time strategy game. Single deliverable, no build step, no
dependencies, no network. Open the HTML file and it plays.

## Hard constraints

These are requirements, not preferences. Breaking one breaks the deliverable.

- **One self-contained file.** `paper-trench.html` must have zero `src=`, `href=`, or
  `http` references. Verify after every change:
  `(Select-String -Path paper-trench.html -Pattern 'src=|href=|http://|https://' -AllMatches).Count`
  must be `0`. No CDN, no webfont, no image file — art is drawn with Canvas 2D and the
  font stack `"Comic Sans MS","Segoe Print","Bradley Hand"`.
- **ES5 syntax only.** `var`, no arrow functions, no template literals, no `let`/`const`.
  The whole script is one `(function(){ "use strict"; … })();` IIFE.
- **Bilingual UI.** Every user-facing string is Chinese then English
  (`"部队已满 40 · Unit limit reached"`). Do not add English-only text.
- **`preview-*.png` are outputs, not inputs.** They are screenshots of the current build;
  regenerate them when a change is user-visible, and never make the game load them.

## File map

`paper-trench.html` (~2,770 lines) is organized by banner comments — jump by searching
for the banner, not by line number:

```
constants (31) · state (64) · helpers (107) · doodle drawing (143) · sizing (240)
unit sprite (354) · data (447) · lifecycle (496) · spawning (541) · projectiles (684)
combat (735) · fx (799) · update (815) · AI (1034) · input (1157)
rendering (1364) → battlefield (1490) HUD (1783) bottom bar (1895) menu (2176)
  hotkey help overlay (2552) game over (2658)
loop (2729) · boot (2750)
```

Repo root also holds `preview-menu.png`, `preview-gameplay.png`, `preview-guide.png`,
`preview-hotkeys.png`. There is no git repo, no package.json, no test directory.

## Verifying changes

There is no test runner. Two complementary techniques were used throughout development;
use them instead of guessing.

### 1. Headless logic harness (fast, for balance and rules)

Extract the `<script>` body, then **rewrite the boot tail** so the IIFE exposes its
internals instead of starting the render loop:

```js
// replace exactly this tail:
//   G.ink = 10;
//   requestAnimationFrame(frame);
// with:
G.ink = 10;
window.__PT = { G: G, L: L, update: update, render: render, resize: resize,
                deployCard: deployCard, spawnUnit: spawnUnit, UNITS: UNITS,
                DIFFS: DIFFS, ORDER: ORDER, countSide: countSide, W: function(){return W;},
                H: function(){return H;} };
// do NOT call requestAnimationFrame
```

Everything a test needs goes through `deployCard(key)` — the same single entry point the
cards and the hotkeys use. There is no separate "click on the field" path to call any
more, so a rule test cannot accidentally exercise a code path the player never touches.

Stub the canvas and 2D context with a `Proxy` that returns a no-op function for any
method and swallows any property set, so `render()` runs without a DOM. Three methods
must return a real object rather than a no-op, or the boot sequence throws:
`createLinearGradient` / `createRadialGradient` → `{ addColorStop(){}, }`,
`createPattern` → `{}`, `measureText` → `{ width: 10 }`. Give the canvas stub
`clientWidth`/`clientHeight` (e.g. 1032×582, the reference layout) since `resize()` reads
them. Then drive the simulation by calling `__PT.update(dt)` in a loop.

Run the whole thing under `vm.runInNewContext` so the extracted IIFE gets its own global
scope; expose the sandbox as `window`. A working smoke test is ~50 lines and is worth
writing first — it catches a stale boot tail immediately, before any balance run.

Node: `C:\Users\Mr Chen\.dsh\dsh-runtimes\dsh-primary-runtime\dependencies\node\bin\node.exe`

### 2. Real browser via Chrome DevTools Protocol (for input and layout)

Chrome: `C:\Program Files\Google\Chrome\Application\chrome.exe`

Drive it over CDP with **real** `Input.dispatchMouseEvent` / `Input.dispatchKeyEvent`.
Synthetic `element.dispatchEvent(new MouseEvent(...))` has hidden bugs before and is not
proof that a click works.

## Testing pitfalls that have already cost time

Do not rediscover these.

- **Headless virtual time leaves `dt ≈ 0` for rAF callbacks.** Under plain
  `--headless=new` with no virtual time policy the game loop runs on the wall clock and
  `sleep()` really does advance the match (that is how the previews are shot). The moment
  you add `--virtual-time-budget`, frames stop carrying a usable `dt` and mid-match states
  must be reached by calling `PT.update(dt)` directly instead.
- **`--window-size` is the outer window, not the canvas.** Headless Chrome reports
  `innerWidth = width - 16` and `innerHeight = height - 95`, so the reference 1032×582
  layout needs `--window-size=1048,677`. Check `innerWidth+'x'+innerHeight` before
  trusting any screenshot's geometry.
- **A measurement harness must let the front line move.** Pinning `frontX` to a fixed x
  gives one side the 50% home-soil bonus and denies it to the other, which silently
  doubles one army's effective HP and makes every duel favour the defender. Let
  `updateFront()` run.
- **`update()` decrements the 180 s clock.** A long sequence of duels in one harness
  process will trip `endGame` and then silently simulate nothing. Reset
  `G.screen = "playing"; G.time = 9999;` before each independent scenario.
- **Stagger duel spawn x-coordinates** (`470 + i*26` vs `810 - i*26`). Stacking units at
  one x makes a single splash hit the whole stack and fakes an AoE result.
- **An ink-splash assertion must count kills plus survivors.** The blot often kills the
  entire squad, and dead units are spliced out — counting only survivors reads 0 and
  looks like a failure.
- **The update loop must stop the moment a match is decided.** Keep the
  `if (G.screen !== "playing") return;` guard after `updateProjectiles`. Its absence
  produced a visible HUD-vs-result mismatch (HUD 91%, result screen 92%).

## Design invariants

- **Blue is the player, red is the enemy.** Sprites are drawn once per `type|side` into
  `spriteCache`; red is the blue sprite mirrored with `scale(-1,1)`.
- **Combat is 1-D along x.** Range checks are `|ax - bx|`, not distance. The HQ is a
  valid target for both sides.
- **Splash inverts the 1-D assumption deliberately.** `splashDist()` squashes the y axis
  by `k = min(SPLASH_SQUASH, 0.85*radius/(L.field.h*0.5))` so a blast always inks the
  whole trench column at any window aspect ratio. A plain `Math.hypot` circle made
  artillery appear to always miss — do not "simplify" this back.
- **50% damage reduction on your own soil** is a spec requirement, not a tuning knob.
  It applies to whichever side is behind `frontX`.
- **Both armies deploy from their own defence line**, `DEPLOY_LINE = 68` px behind the
  HQ. **Deploying is a single action**: the cards, `1`–`5` and `Space` all call
  `deployCard(key)` and the unit is on its way — there is never a second click on the
  battlefield. Keep the battlefield click-free; a click that silently places a unit at
  the pointer is the bug this design replaced.
- **Where the unit comes out is a setting, not a hard-coded rule.** `G.mode` is
  `"manual"` (default) or `"auto"`; `targetLane()` / `targetBlot()` are the only two
  functions that read it, and `deployCard()` calls nothing else. *Manual* reuses
  `G.aim` — the **last** point the player aimed at, never the live pointer — because
  choosing a card drags the pointer down to the tray and that must not move the lane.
  `G.aim.kb` marks an aim that came from the arrow keys. *Auto* ignores `G.aim`
  completely and falls through to `autoLane()` / `autoBlotTarget()`.
- **Auto mode is the fallback, not an error state.** In manual mode with nothing aimed
  at yet (`G.aim.set === false`, e.g. the very first deploy after `Enter`), `targetLane()`
  returns `autoLane()`. The on-field chevron always shows the real answer, so this can
  never surprise the player.
- **`autoLane()` must stay deterministic and y-only.** It scores nine candidate lanes by
  "distance from the enemy's ink-weighted push" minus "crowding from our own units", and
  `spawnUnit(side, type, x, preferY)` honours an explicit lane exactly (only the AI, which
  passes none, lets `pickLane()` scatter). A lane that shifts between frames reads as a
  bug, and a half-honoured `preferY` blend reintroduces the "my click did nothing"
  complaint.
- **Range is position.** `updateUnits()` advances a unit *only* while it has no target in
  range, so a unit's range alone decides how far back it forms up. That is the whole
  mechanism behind "the MG and the tank don't walk into the front rank" — do not add a
  separate stand-off rule for it, and do not hand the ranged units a range above
  `HQ_RANGE` (195) unless you mean them to shell the watchtower for free. Artillery (285)
  is the deliberate exception.
- **Balance invariant: `Q = hp * dps / cost²`** must be roughly equal across units, so
  two armies spending equal ink trade evenly. Q was the metric that exposed artillery
  being unplayable (Q 138) and the tank being unkillable (Q 909). Re-derive Q whenever
  you touch the `UNITS` table.
- **The counter-triangle is intentional and test-locked.** Soldiers are the most
  ink-efficient and beat tanks/artillery in numbers (but must cross the tank's 96 to get
  there); MG shreds infantry and out-ranges the tank 168 to 96; the tank closes on the MG;
  artillery cracks armour and is the only unit out-ranging the HQ watchtower (285 vs 195);
  Ink Splash answers clumped pushes. The duel matrix in the harness must keep producing
  exactly these winners after any range change — the 2024 range bump (mg 122→168,
  tank 44→96, artillery 260→285) left all 13 recorded outcomes unchanged and only narrowed
  the margin of "4 soldiers beat a tank" from 3 survivors to 1.
- **Ink is the only resource.** Cap 10, regen `PLAYER_REGEN`. A unit that cannot spawn
  (40-per-side cap) must not consume ink — spawn first, deduct only on success, and show
  a hint.

## Tuning reference

Current values, kept here because they are the ones the balance was verified against.

| unit | cost | hp | dmg | rate | range | speed | Q |
|---|---|---|---|---|---|---|---|
| soldier | 1 | 62 | 11 | 0.58 | 34 | 30 | 1176 |
| mg | 2 | 92 | 6 | 0.20 | 168 | 14 | 690 |
| tank | 3 | 200 | 34 | 1.05 | 96 | 20 | 720 |
| artillery | 4 | 74 | 100 | 2.6 | 285 | 10 | 178 |
| ink | 5 | — | 175 | instant | radius 122 | — | — |

`DIFFS` — recruit `{think 2.4, regen 0.34, wave 9, stat 0.90, smart 0}`, sergeant
`{1.5, 0.56, 6, 1.00, 0.55}`, general `{0.95, 0.80, 5, 1.06, 1}`.
Match `DURATION = 180`, `HQ_HP = 3000`, `MAX_UNITS_PER_SIDE = 40`,
`PLAYER_REGEN = 0.82`.

Stand-off distances fall straight out of the range column and are asserted in the
harness: soldier 34, mg 168, tank 96, artillery 285 px from a target that cannot move.

Difficulty ladder was verified over 5 seeded full matches per difficulty: recruit
5/5 wins, sergeant 2/5, general 0/5, most decided on territory at 180 s. Re-run that
ladder after any change to `UNITS`, `DIFFS`, or `PLAYER_REGEN` — difficulty drifts
easily and one income retune already had to be walked back.

The ladder is also the A/B rig for **any** balance change, because a harness with a fixed
`Math.random` stream is not comparable to an earlier run with a different one. Seed the
sandbox only when comparing two arms inside one process; for absolute win rates leave
`Math` alone and use ≥20 matches per difficulty. Worked examples:

- **One-tap deployment** (same proxy, only the placement rule swapped): auto-lane 8/10
  recruit, 2/10 sergeant, 0/10 general versus `pickLane` scattering at 9/10, 1/10, 0/10 —
  indistinguishable, so auto-placement did not move the balance.
- **The range bump.** Win counts alone were too noisy (3/30 → 1/30 at sergeant), but the
  *mean blue territory at the end* was not: 27% → 20%. Longer ranges amplify whoever has
  the better composition, which at sergeant is the AI. Raising `PLAYER_REGEN` 0.75 → 0.82
  put it back at 29% / 2-30 wins, and recruit (already meant to be winnable) went 18/20 →
  20/20. When a change moves a metric, prefer the same proxy re-measured over raw win
  counts, which saturate at n≈30.

## Conventions

- **Layout scales from one factor.** `L.ui = clamp(min(W/1032, H/582), 0.62, 1.55)`.
  Multiplied into speeds, ranges, and every offset. Do not hardcode pixel offsets in
  rendering or hit tests; multiply by `L.ui`.
- **Buttons are rebuilt every `render()`** into the `buttons` array via `pushButton()`,
  and hit-tested by scanning backwards in `hitButton()`. Set `BTN_ON = false` to suppress
  registration for an inactive screen (the game-over screen relies on this).
- **Seeded jitter, never `Math.random()` in rendering.** `mulberry32` + `hashStr` give
  doodle wobble that is stable across frames. Randomness per frame makes the art crawl.
- **Scratch files get a leading underscore** (`_harness.js`, `_probe.js`, `_crop.png`) and
  are deleted before presenting anything. The tree must contain only
  `paper-trench.html`, `AGENTS.md`, and `preview-*.png`.
- **Keyboard and mouse are both first-class.** Every action reachable by click must be
  reachable by keyboard, and both routes must funnel through `deployCard(key)` so they
  cannot drift apart. `G.lastKey` remembers the last card used; `Space` repeats it, which
  is how a player holds a push together. Arrow/WASD aim the lane in manual mode
  (`moveAim`), and in auto mode they raise `DEPLOY_HINT` so the old muscle memory gets an
  explanation instead of silence.
- **One-line captions shrink with `fitFont()`.** Canvas does not wrap or auto-shrink, and
  a clipped hint is worse than a small one. Any single-line label that carries a variable
  string (menu blurbs, the keyboard legend, the tray tip) goes through
  `fitFont(text, maxW, startPx)`; `wrapText` is for genuinely multi-line prose.
