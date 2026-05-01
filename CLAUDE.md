# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the project

There is no build system. Serve the files with any static HTTP server and open in a browser:

```bash
python3 -m http.server 8080
# or
npx serve .
```

Then open `http://localhost:8080/main.html`. The app requires camera permission and a physical **Hiro marker** in view to trigger the AR scene.

## Architecture

The entire application lives in `main.html` as a single HTML file with inline JavaScript. There is no bundler, no package.json, and no framework beyond the CDN-loaded libraries.

**Libraries (CDN):**
- A-Frame 1.4.0 — declarative 3D/WebXR scene graph
- AR.js (aframe-ar.js) — webcam-based marker tracking (Hiro marker preset)

**Scene structure (`<a-scene>`):**
- `#hiro-marker` — A-Frame AR.js marker; its child `#source-cell` is the main cell anchored to the printed marker
- `#bloodstream` — vessel shell + RBC/platelet particles flowing in world space
- `#background-cells` — 7 floating target cells in world space (not marker-anchored)
- `#immune-actors` — antibody entities spawned over time

**Simulation layers and state machines:**

| Array | Spawned by | States |
|---|---|---|
| `exosomes[]` | `setInterval(spawnExosome, 420ms)` | `travel` → `fusion` (absorbed by cell) or `blocked` (neutralized by antibody, then flows away) |
| `antibodies[]` | `setInterval(spawnAntibody, 980ms)` | `seeking` → `bound-exosome` (rides a blocked exosome) or `attached` (binds to a cell briefly) |
| `bloodParticles[]` | `createBloodstreamFlow()` once | continuous loop; reset to back of vessel when `pos.z > 2.1` |

**Main loop:** `requestAnimationFrame` with capped delta time (max 50 ms). Calls `updateBloodFlow`, `updateExosomes`, `updateAntibodies` each frame.

**Cell construction (`buildCell`):** Procedurally assembles A-Frame sphere elements (outer membrane, inner membrane, cytoplasm, nucleus, nucleolus, vesicles, surface bumps) and stores references in `entity.__cellParts` for later color signalling.

**Cell signalling (`setCellSignal`):** Temporarily shifts cell part colors to an `exosome` (blue) or `antibody` (pink) palette for 820 ms using `setMaterialColor`, then reverts. Triggered on fusion/attachment events.

**Camera selection:** On load, `getUserMedia` is called to get camera permission, then `enumerateDevices` populates a `<select>`. `startAR()` passes the chosen `deviceId` to AR.js and begins the simulation.

## Assets

- `human_cell.glb` — 3D cell model (not currently used in `main.html`; scene objects are built procedurally)
- `antibody.glb` — antibody model (same — antibodies in `main.html` are procedural cylinders)
- `exosome.blend` / `exosome.blend1` — Blender source files

`test.html` is a separate minimal MindAR prototype (image-target AR, requires a `targets.mind` file and `model.glb` not present in the repo).
