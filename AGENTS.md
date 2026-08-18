# AGENTS.md

Asteroids clone rendered with HTML5 Canvas + vanilla ES6 JS. No bundler, no frameworks, no dependencies, no `package.json`.

## Toolchain

- **There is no build, test, lint, typecheck, or codegen step.** Do not look for or run npm scripts. The only verification is loading the game in a browser and playing it.
- Zero-toolchain is a project constraint. If you add tooling, do it intentionally.

## Running

- Open `index.html` directly in a browser (file:// works — there are no module imports or fetches).
- Or `npx serve .` → http://localhost:3000 (needs Node despite the absence of a package.json).

## Architecture

All logic lives in `game.js` (single ~420-line file); `index.html` only hosts the `<canvas>` and loads the script. No ES modules — everything shares module scope.

- Classes: `Ship`, `Asteroid`, `ShootingStar` (extends `Asteroid`), `Bullet`, `Particle`, `PowerUp`.
- Game state is module-level mutable globals (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`). `initGame()` reinitializes them.
- State machine: `state` ∈ `{'playing','dead','gameover'}`. Ship death → 2s `deadTimer` → respawn via `ship.reset()`. `Space` restarts from `gameover`.

### Canvas size is coupled in two places
`index.html` (`<canvas width="800" height="600">`) **and** `game.js` (`const W = 800; const H = 600;`) must stay in sync — wrap/collision/physics all depend on `W`/`H`. Change both or neither.

### Shooting stars
`ShootingStar extends Asteroid` and lives in the same `asteroids` array, so the wrap/collision/filter loops apply automatically. It overrides `split()` to return `[]` (no fragments) and `draw()` to render a 4-point gold `#ff0` star with the same stroke style. A `Particle` is dropped at its position every frame in `update()` for a comet trail. A module-level `shootingStarTimer` in `update()` spawns one every ~8-15s during `'playing'` from a random screen edge. Each has a `ttl` (~4-7s) that sets `dead = true` and lets the existing filter remove it ("disappears after some time"). Worth `SHOOTING_STAR_POINTS = 300` — handled specially in the bullet-vs-asteroid loop, not via `POINTS[a.size]`. Collides with the ship like a normal asteroid (deadly).

### Input model
`keys` (held-state) + `justPressed` (single-edge). `pressed(code)` **consumes** the edge flag — call it at most once per frame per key, or the press is lost. Keys: `ArrowLeft`, `ArrowRight`, `ArrowUp`, `Space`.

### Loop
`requestAnimationFrame` with `dt` clamped to 0.05s; all motion is `dt`-scaled (frame-rate independent). Space is toroidal: `wrap()` applies to ship/bullets/asteroids but **not** particles (they fly off-screen).

### Rendering
Wireframe only — everything stroked `#fff`, `lineWidth` 1.5, no fills. Ship blink during respawn invincibility is implemented by skipping `draw()` on alternating frames.

## Gotchas

- Ship/asteroid collision uses a 0.82 forgiveness factor (`ship.radius + a.radius * 0.82`) — intentionally lenient, not a bug. Do not "fix" it.
- No audio in the game.