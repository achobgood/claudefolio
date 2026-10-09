# One-Shot Prompt: "Hydro Rush" — a mobile-first Hydro Thunder–style boat racer

Copy everything below the line into a capable coding model. It's written to produce a finished, playable game in a single pass with no follow-up questions.

---

You are a senior web game developer. Build **Hydro Rush**, an original arcade powerboat racer inspired by the *feel* of Midway's Hydro Thunder (1999): absurd speed, boost-pickup hunting, giant ramp jumps, a checkpoint countdown timer, and a fast chase camera. Do **not** use the Hydro Thunder name, logos, boat names, track names, or any copyrighted assets. All art is procedural, all audio is synthesized.

## Deliverable

- One self-contained `index.html` (inline CSS + JS ES module) plus `vendor/three.module.min.js` (Three.js r160, MIT), imported via an import map: `{"imports":{"three":"./vendor/three.module.min.js"}}`.
- No build step, no other dependencies, no external network requests at runtime. It must work from any static host (GitHub Pages, S3, Vercel) and from `python3 -m http.server`.
- Production quality: no console errors, defensive `try/catch` around every browser API that can be missing or throw (audio, vibration, fullscreen, orientation lock, device orientation, `localStorage`).

## Core gameplay (the Hydro Thunder feel)

1. **Race format:** 3 laps on a closed river-canyon circuit against 5 AI boats. Final standings 1st–6th.
2. **Checkpoint timer:** A countdown clock starts at roughly the time needed to reach the first checkpoint. There are 4 checkpoint gates per lap, and each one adds time ("+12.4s" flash). If the clock hits zero, it's **TIME'S UP** and the run is over. Tune bonuses so a clean run finishes with about 5–15s to spare and a run with lots of wall crashes is in danger.
3. **Boost economy:** Boats start with 3s of boost. Floating glowing pickups: **blue = +3s, red = +6s**, capped at 12s. Boost raises top speed and acceleration, adds an FOV kick, flames from the engine, and a whoosh sound. Pickups respawn after 10s.
4. **Hydro Jump:** Double-tap boost (or press the JUMP button / `J` key) with at least 1.5s of boost to launch the boat into the air (costs 1s). Use it to hop obstacles and reach airborne pickups.
5. **Ramps:** Wedge ramps placed along the river launch the boat in proportion to its speed. Some pickups hang in the air above ramps, which rewards lining up jumps.
6. **Obstacles:** Buoys and floating logs that bump and slow the boat on contact. Canyon walls bounce the boat back with a speed penalty, screen shake, spray, a thud and a vibration.
7. **Boat select:** 3 boats with visibly different stats (Top Speed / Handling / Boost Capacity bars):
   - **Thresher**: balanced
   - **Barracuda**: fastest, drifts more
   - **Manta**: best grip and turning, lower top speed
8. **AI:** Opponents follow the racing line with per-boat lateral offsets and sway, hit ramps, use boost occasionally, and rubber-band mildly (faster when far behind the player, slower when far ahead) so races stay close. Boat-to-boat collisions push boats apart.
9. **Results screen:** Final position, total time, best lap, a per-boat standings table, and best-time records per boat saved in `localStorage`.

## Physics (arcade, not simulation)

- 2D planar physics (x/z) with heading, forward thrust, quadratic drag, and **lateral grip** so boats slide and drift like they're on water. Steering authority scales with speed and drops sharply while airborne.
- Vertical axis: boats float on an analytic wave function `h(x,z,t)`, a sum of 3–4 sines that **the water shader shares**, so boats visibly ride the swells. Pitch and roll follow the wave gradient plus steering lean. In the air: gravity, nose-up pitch, and a splash with a speed dip on landing.
- Track-relative coordinates: precompute ~1500 samples of a closed Catmull-Rom spline (position, tangent, right vector, cumulative distance, half-width). Each boat keeps a sample index and searches a small window for the nearest sample to get **progress** and **lateral offset**. Use those for walls, laps, checkpoints, ramps, race positions, wrong-way detection and AI steering.
- Fixed-timestep simulation (e.g. 120 Hz with an accumulator, max 5 substeps) decoupled from rendering.

## Visuals (procedural, stylized, 60fps on mid-range phones)

- **Water:** a camera-following, grid-snapped subdivided plane with a custom `ShaderMaterial`: vertex waves from the same sine sum, deep/shallow color blend, fresnel sky reflection, sun specular, procedural foam streaks, and distance fog.
- **Canyon:** extruded rock walls along both sides of the spline with noisy jagged profiles and vertex-colored sandstone strata. Instanced low-poly pine trees on the rim. Gradient sky dome with a sun glow. Fog matched to the horizon.
- **Boats:** built from primitives (a tapered hull from an extruded shape, cockpit canopy, tail fins, racing stripe, engine pod), with a unique livery per boat.
- **Effects:** pooled particle systems (Points with additive blending) for the wake, rooster-tail spray, boost flames, wall-hit spray and landing splashes. Checkpoint arches glow; the start/finish arch has a checkered banner.
- Chase camera with spring smoothing, speed-based FOV, boost FOV kick, and shake. In **portrait**, widen the vertical FOV so horizontal visibility stays about the same as in landscape.

## Mobile-first controls and UX (the most important section)

- Viewport `width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover`. `touch-action:none`, `overscroll-behavior:none`, `user-select:none`, `-webkit-tap-highlight-color:transparent`. No scroll, rubber-banding, pinch-zoom, double-tap zoom, long-press callout or context menu.
- Use **Pointer Events** with per-`pointerId` tracking so steering and boost work at the same time with multiple fingers.
- **Steering (default):** a floating virtual stick. Touch anywhere in the left ~50% of the screen to set its origin; horizontal drag of about 70px gives full lock. Show the base and knob where the thumb landed.
- **Steering (optional): Tilt.** A settings toggle that calls `DeviceOrientationEvent.requestPermission()` on iOS from the tap handler and uses the correct axis for `screen.orientation.angle` (portrait vs. landscape-left/right), with a dead zone and smoothing.
- **Right side:** a large **BOOST** button (with a ring that shows the boost meter), a **JUMP** button, and a **BRAKE** button. Throttle is automatic, as in the arcade. All buttons sit within thumb reach and respect `env(safe-area-inset-*)`.
- Desktop fallback: Arrows/WASD to steer, Space/Shift to boost, `J` to jump, `S`/Down to brake, `P`/Esc to pause.
- HUD: position ("2nd/6"), lap, countdown timer (pulses red under 5s), speed in MPH, race clock, boost bar, minimap (pre-rendered track path plus live boat dots), center-screen callouts ("CHECKPOINT +11.2s", "FINAL LAP", "WRONG WAY", "HYDRO JUMP!").
- 3-2-1-GO countdown with beeps. Pause menu (Resume / Restart / Quit). Auto-pause on `visibilitychange`.
- On the first tap: unlock `AudioContext`, try `requestFullscreen` and `screen.orientation.lock('landscape')` (both optional and caught). Show a dismissible "rotate for best experience" hint in portrait, but keep the game fully playable in portrait.
- **Performance:** cap `devicePixelRatio` at 2, no shadow maps, a modest polygon count, `InstancedMesh` for trees and buoys, and pooled particles. **Adaptive resolution:** measure frame time over rolling 1s windows and step the render scale between 0.5 and the cap to hold about 55–60fps. Handle `resize` and `orientationchange`.

## Audio (WebAudio, all synthesized)

- Engine: two detuned oscillators (sawtooth + square) through a low-pass filter, with pitch and cutoff tracking speed and boost.
- Water rush: looped filtered noise whose gain tracks speed.
- One-shots: pickup arpeggio, boost whoosh (noise sweep), wall thud, landing splash, checkpoint chime, countdown beeps, finish fanfare. Master mute toggle, saved in `localStorage`.

## Code organization (in the single file)

Write clear sections with header comments: `CONFIG` (all tuning constants in one object), `UTIL`, `TRACK` (spline, samples, geometry), `WATER`, `SCENERY`, `BOATS` (mesh factory and stats), `PHYSICS`, `AI`, `PICKUPS/RAMPS/OBSTACLES`, `PARTICLES`, `AUDIO`, `INPUT`, `HUD`, `GAME STATE` (title → select → countdown → race → paused → finished/timeup), and `MAIN LOOP`. Use no globals beyond one `game` object. Expose `window.__hydro = game` for automated testing.

## Acceptance checklist (verify before you finish)

- [ ] Loads with zero console errors in Chromium at 390×844 portrait and 844×390 landscape.
- [ ] Title → boat select → countdown → race → results works using only touch input.
- [ ] Steering and boost work at the same time (multi-touch).
- [ ] Pickups, ramps, Hydro Jump, walls, obstacles, checkpoints, laps, TIME'S UP, and the results screen all work.
- [ ] Page never scrolls or zooms. HUD and controls stay clear of notches and home indicators.
- [ ] Holds about 60fps on a mid-range phone, and adaptive resolution steps down on slow devices.
