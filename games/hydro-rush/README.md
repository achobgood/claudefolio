# Hydro Rush

A mobile-first arcade powerboat racer in the spirit of 1999's arcade boat racers: boost-pickup hunting, giant ramp jumps, a checkpoint countdown clock, and a fast chase camera. It's a single HTML file built on Three.js, with procedural art, synthesized audio, and no build step.

- `index.html` is the whole game: inline CSS and an ES module.
- `vendor/three.module.min.js` is Three.js r160 (MIT), vendored so the game runs offline and on any static host.
- `PROMPT.md` is the one-shot prompt used to spec the build.

## Play locally

```bash
cd games/hydro-rush
python3 -m http.server 8080
# open http://localhost:8080 (or http://<your-LAN-IP>:8080 on your phone)
```

ES modules don't load from `file://`, so serve the folder over HTTP.

## Deploy

Any static host works: GitHub Pages, Vercel, Netlify, S3 or Cloudflare Pages. Upload `index.html` and the `vendor/` folder together.

## Controls

| Action | Touch | Keyboard |
|---|---|---|
| Steer | Drag anywhere on the left half (floating stick), or **Tilt** mode in settings | ← → / A D |
| Boost | Hold **BOOST** (the ring shows remaining boost) | Space / Shift |
| Hydro Jump | Double-tap **BOOST**, or tap **JUMP** (needs ≥1.5s boost, costs 1s) | J, or double-tap Space |
| Brake / reverse | **BRAKE** | S / ↓ |
| Pause | ❚❚ button (auto-pauses when the tab is hidden) | P / Esc |

Throttle is automatic, as in the arcade game.

## Rules

- 3 laps against 5 AI boats. Each lap has 4 checkpoint gates, and each one adds time to the countdown clock. Bonuses shrink 6% per lap. If the clock hits zero, it's **TIME'S UP**.
- **Blue** pickups add 3s of boost and **red** ones add 6s. Red pickups also hang in the air past every ramp.
- Canyon walls, buoys and logs cost you speed.
- Boats: **Thresher** (balanced), **Barracuda** (top speed, loose handling), **Manta** (grip and a big boost tank).
- Best race and best lap times are saved per boat in `localStorage`.

## Tuning

All gameplay constants live in the `CONFIG` object at the top of the script:
- `timerPace` and `timerStartExtra` set clock difficulty.
- `BOATS` sets boat stats.
- `TRACK_PTS` holds the river spline control points.
- `WAVES` is the wave field, shared by the physics and the water shader.

Lower `timerPace` for a more forgiving clock.

Measured with the built-in autopilot (`__hydro.debug.setAutopilot(true)`):
- A clean 3-lap race takes about 115–135s.
- The clock never drops below about 5s.
- The autopilot finishes with about 12–20s to spare.
- Rivals finish within a few seconds of it.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Blank page, console says module or CORS error | You opened `index.html` via `file://`. Serve it over HTTP (see above). |
| "WebGL is not available" | Enable hardware acceleration in the browser, or try another browser. Every phone from the last ~8 years supports WebGL. |
| No sound on iPhone | Flip the ring/silent switch off. Audio starts on your first tap. |
| Tilt steering does nothing | iOS asks for motion permission when you toggle **Steering: Tilt**, so allow it. Tilt calibrates to how you're holding the phone when you enable it. Toggle off and on to recalibrate. |
| Low frame rate | The game lowers its render resolution automatically (down to 0.5× pixel ratio). Closing other tabs helps on older phones. |
| Page scrolls or zooms on a phone | Shouldn't happen (`touch-action:none`). If you embedded it in an iframe, give the iframe `allow="fullscreen"` and no scrolling. |

## Testing hooks

`window.__hydro` exposes the game object:
- `__hydro.debug.start(boatIndex)` starts a race.
- `__hydro.debug.fastForward(seconds)` runs the fixed-step simulation synchronously.
- `__hydro.debug.setAutopilot(true)` hands the player boat to the AI driver.

These hooks drove the headless Playwright checks: portrait and landscape phone viewports, multi-touch steering plus boost, a full race, the time-up path, and the keyboard flow, all with zero console errors.
