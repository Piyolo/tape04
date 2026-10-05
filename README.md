# TAPE_04: THE SUB-LEVELS

A first-person psychological horror game that runs entirely in the browser. You play David, an
archivist who queued a corrupted 1987 VHS tape into deck four — and ended up trapped inside the
tape's live feed.

Everything is generated in code: no image, audio, model or font files ship with this project.

```
tape04/
├── index.html    the entire game (self-contained)
├── vercel.json   static host config + security headers
└── README.md
```

---

## Play it

Open `index.html` in any modern browser. An internet connection is required on first load,
because the two Three.js library files are pulled from `cdn.jsdelivr.net`.

### Desktop

| Input | Action |
| --- | --- |
| Click **▶ CLICK TO INSERT TAPE** | Start (also captures the mouse and starts audio) |
| Mouse | Look |
| `W` `A` `S` `D` / arrows | Walk and strafe |
| `E` | Recover a floppy disk / insert disks at the terminal |
| `Esc` | Release the mouse → pause |

### Mobile

Rotate to **landscape** — the game shows a rotate prompt in portrait.

| Input | Action |
| --- | --- |
| Tap **▶ CLICK TO INSERT TAPE** | Start (this is also what unlocks audio on iOS) |
| Drag on the **left half** | Analog stick — how far you push sets your speed |
| Drag on the **right half** | Look around |
| **E** button (bottom right) | Interact; it glows green when something is in reach |
| **II** button (top right) | Pause / resume |

Both halves accept input at the same time, so you can back away while watching the corridor.

---

## Objective

Three glowing floppy disks are hidden in dead ends far from the spawn room. Look at one and press
`E` (or the **E** button), then carry all three back to the CRT terminal and interact with it to
end the game.

Two things to keep in mind:

- Your flashlight is the only light source. Corridors past roughly 12 m are pitch black.
- The concrete halls **rearrange themselves** while you are not looking at them, so backtracking
  geometry will not necessarily match what you remember. Every rearrangement is validated so the
  terminal and any uncollected disk always remain reachable.

---

## Deploying to Vercel

### Option A — Vercel CLI (fastest, no GitHub needed)

```bash
cd tape04
npx vercel          # preview URL
npx vercel --prod   # production URL
```

The CLI prompts you to log in on first run.

### Option B — GitHub, then Vercel (auto-deploys)

1. Create a repository and push this folder to it.
2. Go to **vercel.com/new** → **Import** the repo → **Deploy**.
3. Every `git push` now redeploys automatically, and pull requests get their own preview URL.

No build step and no framework: Vercel serves the files as static content.

> Deploying from the repo root works if `index.html` sits at the root. If you nest this project in a
> subfolder, set the project's **Root Directory** to that folder (e.g. `tape04`).

---

## How it works

- **Rendering** — Three.js r128 via CDN. Scenes render to a `WebGLRenderTarget` at a fraction of
  screen resolution with `NearestFilter` for chunky PS1-style pixels, then a fullscreen fragment
  shader applies the VHS look: chromatic aberration, a rolling tracking band, scanlines, grain,
  head-switch noise, vignette, and a red-flood mode used by the jumpscare.
- **Lighting** — Ambient light is zero. A narrow-angle `SpotLight` is parented to the camera and
  casts hard shadows (`BasicShadowMap`). Wall blocks are lazily instantiated and hidden by
  `visible` rather than added/removed, so cells can be toggled without reallocating geometry.
- **Maze** — 21×21 recursive-backtracker maze on a 3.5 m grid. A shifting-hall routine opens one
  wall and closes one passage per cycle, only for cells entirely outside the camera frustum, and
  every close is BFS-validated before it is allowed to stick.
- **Stalker AI** — A cluster of wireframe boxes with flickering red/purple emissive materials. When
  it is outside the camera frustum it BFS-paths through the maze toward a point behind your head;
  when it enters view it stops and creeps. Inside 3 m it triggers the jumpscare.
- **Audio** — 100% Web Audio synthesis: a 55 Hz sawtooth drone through a resonant lowpass with a
  slow LFO, a heartbeat built from a 150→40 Hz sine sweep whose interval tracks the stalker's
  distance, and white noise generated into an `AudioBuffer` from random floats for static bursts.
- **Performance** — Phones and tablets start on a lighter tier (512² shadow map, lower internal
  resolution, disk lamps off). If the average frame time stays above ~33 ms the internal
  resolution steps down; if it recovers it steps back up. Only resolution changes, so there are no
  shader recompiles mid-play.

---

## Requirements

- A browser with WebGL and Web Audio (Chrome, Edge, Firefox, Safari 15+).
- Audio starts only after your first click/tap, as browsers require.
- Tested against Three.js r128, pinned to an exact version so a future CDN release cannot change
  behaviour under you.
