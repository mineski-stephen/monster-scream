# Monster Energy — Scream Machine

A booth game for Monster Energy marketing activations. Players scream into a microphone and the game scores how loud they get, all choreographed to a looping backing track. It runs full-screen in **portrait** on a TV, served as plain HTML/JS — no build step, no framework.

The game lives in **`index.html`**. Two tech-demo files document the engines it is built on: `audio.html` (seamless audio + 100 BPM grid) and `paper-scrap-letters.html` (torn-paper text animation).

---

## Feature list

### Title screen
- Black background with the `bg-menu.png` texture fit to the screen.
- A **"fence" frame** built from the `bg-border.png` segment (1318×50), repeated around all four edges and scaling with the screen.
- **Microphone-access notice modal** on load — just the title, the message, and an **OK** button. Pressing OK requests the browser/device microphone permission.
- **Automatic calibration:** as soon as mic access is granted, the calibration screen pops up on its own. The `monster.svg` logo fills with green over 3 seconds, captioned "Stay silent while the microphone calibrates…", with no music playing so it reads true ambient noise. It averages 30 readings and sets that level as the baseline **0**, so the game works in loud booths. **The looping background music (`full.ogg`) starts once calibration finishes.** If access is denied it shows *"Could not calibrate. Microphone access denied."* and the music starts on close.
- **Floating START button** (`start.png`). Hovering zooms it slightly and adds a green tint that is **masked to the button's non-transparent pixels** (it never tints the transparent area), all while it keeps floating.
- The **Monster logo** sits lower on the screen with a strong drop shadow; the **"Scream Machine"** title is smaller and fully white.
- A small **Recalibrate Mic** button is pinned to the bottom to re-run calibration at any time (also muting the music while it runs).

### Audio engine
- `full.ogg` is decoded into a Web Audio buffer and looped with sample-accurate, **zero-gap** seamless playback.
- A constant (inaudible) **100 BPM** beat grid runs across the whole 58-bar loop, derived from the audio clock — everything in the game is timed to it.
- **Touch points** (drum-fill measures that lead into a drop): measures **4, 8, 17, 21, 33, 37, 42, 46, 50**, each reinforced with `riser.ogg`.
- Pressing **START** looks ahead from the current spot to the **next drum-fill measure that still has a free "Get Ready" measure before it** (≥2 measures ahead, wrapping around the loop — e.g. at measure 9 → 17, at 36 → 42, at 49 → 4), then **queues it to play next** (a seamless measure swap), so the countdown always leads into a real drop. A short **spinner** plays, then the Prepare screen appears.
- Tapping **START** fires `scream.ogg` and `squeal.ogg` together as a stinger.

### Prepare screen
- Background: `halftone-loop.mp4` (with `halftone.jpg` as fallback); the Monster logo is retained.
- The **"GET READY TO SCREAM!!"** message (paper-scrap letters falling in one by one) appears promptly after START — no long wait.
- On the queued drum-fill measure, a beat-locked countdown drops on each beat: **3 · 2 · 1 · SCREAM!** The riser builds underneath and resolves into the drop, which starts the action.

### Action screen
- Background: `graffiti-loop.mp4` (with `graffiti.jpg` as fallback); no Monster logo. Text uses **AntonSC**.
- The `monster.svg` icon **fills with green** to the current loudness and **pulses** with the beat.
- A vertical **dB meter on the left** runs 0 → 65 → 130 with a fill that reacts to loudness (gradient green at 0, yellow at 65, red at 130).
- Loudness messages from quiet to loud: **"CAN'T HEAR YOU!" → "GET LOUD!" → "MAXIMUM SCREAM!" → "MONSTROUS!!"**
- The reading, message, and pulse refresh **on the beat**, and refresh faster as you get louder — **every beat, every beat, twice per beat, four times per beat** across the four loudness bands.
- The number is labelled **dB** and runs **0–130** on the meter. Calibrated silence reads **0**, and the scale hits **100% (130 on the meter) at ~95 dB** — a realistic loud scream — rather than at full digital scale, so the meter actually reaches the top.
- A **10-second timer** runs the round, then the results appear.

### Results screen
- Background: `halftone-loop.mp4` (with `halftone.jpg` fallback) behind a `blackboard.png` scoreboard that fills almost the whole screen.
- The Monster logo, the peak dB reading, and the floating **Main Menu** button all sit **inside the blackboard**. Main Menu plays the scream/squeal stinger on press; the music keeps looping the whole time.

### Everywhere
- A **vignette** darkens the edges on the Prepare, Action, and Results screens (not the menu).

---

## Running it

Because the audio is loaded with `fetch()` + Web Audio decoding, the game **must be served over HTTP** — opening `index.html` directly from the file system (`file://`) will not load the audio.

From the project folder, start any static server, e.g.:

```bash
python -m http.server 8000
```

Then open **http://localhost:8000/** in the browser.

### Browser requirements
- A Chromium-based browser (Chrome/Edge) is recommended for the booth.
- Microphone permission must be granted (the on-load modal requests it on **OK**).
- **The microphone only works in a secure context** — `https://` or `localhost`/`127.0.0.1`. Over a plain `http://` LAN address (e.g. testing on a phone via the PC's IP) the browser will silently refuse to show the mic prompt. For mobile testing, serve over HTTPS or use a tunnel.
- Audio and the microphone only start after the first tap (browser autoplay/gesture policy) — that tap is the modal's **OK** button.
- Best displayed full-screen in portrait orientation (the stage is locked to a 9:16 area and scales to the screen).

---

## Assets

| Path | Use |
| --- | --- |
| `img/bg-menu.png` | Title background texture |
| `img/bg-border.png` | Repeating "fence" frame segment |
| `img/monster.svg` | Green-fill meter (calibration loader + action screen) |
| `img/monster-large-noline.png` | Monster logo branding |
| `img/start.png` | START button |
| `img/paper-button.png` | Main Menu button |
| `img/blackboard.png` | Results scoreboard |
| `img/halftone-loop.mp4` / `img/halftone.jpg` | Prepare & Results background |
| `img/graffiti-loop.mp4` / `img/graffiti.jpg` | Action background |
| `audio/full.ogg` | Looping background music (58 bars @ 100 BPM) |
| `audio/riser.ogg` | Riser on drum-fill measures |
| `audio/scream.ogg`, `audio/squeal.ogg` | START stinger |
| `fonts/boston.ttf` | Headers & titles |
| `fonts/AntonSC-Regular.ttf` | Body / action / scrap-letter text |
