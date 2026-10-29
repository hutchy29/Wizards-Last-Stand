# CLAUDE.md — Wizard's Last Stand

Mobile-first, **landscape** pixel-art survivor / bullet-heaven auto-shooter (inspired by *Deep Rock Galactic Survivor*). You play a medieval Wizard, Warrior or Ranger, dive through procedurally generated caves, auto-fire abilities, mine rock for gold and pickups, level up, and fight bosses.

**Status:** playable, feature-rich, v1 complete. Last feature added: Save data screen (export/import save code) on the title menu. See `docs/ROADMAP.md` for what's next.

## The one important fact
The whole game is **one file: `index.html`** (~790 lines, ~86 KB, vanilla JS + canvas, no build step, no dependencies, no assets). Code is deliberately **dense/minified-style** (short names, many statements per line). Edit surgically; don't reformat or "clean up" the file wholesale.

## Commands
- Play: `npm run serve` then open `http://localhost:8080` (or just open `index.html`; for a phone, serve over LAN).
- Test: `npm i` then `npx playwright install chromium` once, then `npm test` (headless Playwright scripts in `tests/`; screenshots go to `tests/out/`). Run a single one: `node tests/te.js`.
- No build, no lint, no bundler. Don't add one unless asked.

## Hard constraints (don't break these)
- **Single file, no external assets/CDNs.** Sprites are drawn procedurally to canvases; audio is synthesized with Web Audio. Keep it that way.
- **Landscape.** Portrait on touch devices shows a rotate prompt (`rotQ`). Logical height is fixed at **180 px** (`LH`); width derives from aspect (`LW`); CSS upscales with `image-rendering:pixelated`. Don't change the pixel scale model.
- **16-bit audio style.** All sound goes through the `AU` engine (oscillators + noise + bit-crush WaveShaper + echo) on buses `sfx/mon/amb/ui` + master. New sounds: add to `SFX` (and a throttle in `THR`, voice cap is 44). Monster voices in `MV`, per-biome ambience in `AMB`.
- **Max 4 abilities per run.** Level-up pool is gated by this.
- **Gold is deliberately halved** everywhere (pickups `×.5`, quest rewards `×.5`, dive bonus). Don't "fix" it.
- **Rarity is deliberately steep** (weights 60/26/10/3.2/.8). Special upgrades and heart crystals are deliberately rare.
- **Difficulty names are Normal / Heroic / Epic / Legendary / Extreme** — never "Hazard".
- **In-run "Fortune" card** (`BN.know`) and **Tower "Fortune"** (`UP.luck`) share a name by user request. The user declined renaming Tower Fortune to "Luck" — keep both as "Fortune".
- Persistence is `localStorage` key **`wizards-last-stand-v1`** (`SV`). Changing save shape → keep backward compatibility (defaults via `||`), or bump key with a migration. Always wrap storage in try/catch.
- Pause on tab hide; menus hide the HUD via `menu(on)`.
- Hash/noise uses `Math.imul` integer hashing (`hs`, `h01`, `vn`). A previous float-multiply hash produced degenerate noise — don't revert.

## Where things are (search the symbol)
| Area | Symbols |
|---|---|
| Globals / state | `S` (run), `SV` (save), `E` enemies, `PJ` projectiles, `EP` boss projectiles, `ZN` warning zones, `RG2` flame rings, `RC` tile map, `TH` tile HP |
| Audio | `AU`, `SFX`, `THR`, `MV`, `DIE`, `AMB`, `sfx()`, `wantAmb()`, `VOLS`, `options()` |
| Enemies | `ROLE`, `BOSS`, `TY`, `BAB`, `spawn()`, `mkE()`, `bossAct()`, `hit()` |
| World | `BI` (biomes), `HZ` (difficulty), `HN`, `tiles()`, `tile()`, `dig()`, `digR()`, `ORE`, `MHP` |
| Progression | `CL` classes, `IT` abilities, `BN` boons, `RAR`, `UP` Tower upgrades, `QP`/`DIVE` quests |
| Run flow | `newGame()`, `update()`, `showLvl()`, `pick()`, `rollR()`, `apply()`, `openGate()`, `stageClear()`, `clearStage()`, `endRun()` |
| UI | `title()`, `dive()`, `tower()`, `brief()`, `pausePanel()`, `panel()`, `hud()` |
| Render | `draw()`, `spr()`, `portrait()`, `preview()`, `fit()` |
| Input | `jx/jy` floating joystick, `KM` keyboard map |

Full detail: `docs/ARCHITECTURE.md`. Game rules and numbers: `docs/GAME_DESIGN.md`.

## Conventions when changing code
1. Read the surrounding function first (lines are long). Prefer targeted edits over rewrites.
2. Any gameplay number change → update `docs/GAME_DESIGN.md`.
3. Any new feature → add a Playwright test in `tests/` (copy `te.js` as a template: set state via `page.evaluate`, drive UI by selectors, assert, and fail on `pageerror`) and a line in `docs/CHANGELOG.md`.
4. Test at landscape phone viewport **844×390** (and a wider desktop one). Check there are no `pageerror`s.
5. UI is HTML overlays (`#ov` panels) over a canvas; keep tap targets finger-sized.
6. Keep text style: short, plain, game-flavoured.

## Test hooks
Tests drive the game by touching globals: `S.hp=S.mhp=99999`, `S.t=S.s0+150` (jump timer to ready the ley gate), `S.xp=999` (force level-up), `S.x=S.gx;S.y=S.gy+2` (teleport onto gate), `S.rr` (rerolls). Title buttons are `#b1` etc. Keep these globals accessible (script is not a module).

## History of how this file was published
It was originally delivered as a private claude.ai Artifact (https://claude.ai/artifact/KdXxb5czr7GjUtNk7VestQ, v12). The repo copy is the source of truth now. `legacy/` holds earlier prototypes (isometric dungeon crawler, early pixel version) for reference only — don't extend them.
