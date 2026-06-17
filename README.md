# Monster Energy — Scream Machine

A booth game for Monster Energy marketing activations. Players scream into a microphone and the game scores how loud they get, all choreographed to a looping backing track. It runs full-screen in **portrait** on a TV, served as plain HTML/JS — no build step, no framework.

The game lives in **`index.html`**. Two tech-demo files document the engines it is built on: `audio.html` (seamless audio + 100 BPM grid) and `paper-scrap-letters.html` (torn-paper text animation).

---

## Feature list

### Title screen
- Black background with the `bg-menu.png` texture fit to the screen.
- A **"fence" frame** built from the `bg-border.png` segment (1318×50), repeated around all four edges, inset slightly from the screen edge and scaling with the screen.
- **Microphone-access notice modal** on load — just the title, the message, and an **OK** button. Pressing OK requests the browser/device microphone permission.
- **The looping background music (`full.ogg`) starts the moment the mic modal closes** — it keeps playing even while calibration is running.
- **Automatic calibration:** as soon as mic access is granted, the calibration screen pops up on its own. The `monster.svg` logo fills with green over 3 seconds, captioned "Stay silent while the microphone calibrates…". It averages 30 readings and sets that level as the baseline **0**, so the game works in loud booths. If access is denied it shows *"Could not calibrate. Microphone access denied."* and the title appears on close (the music is already playing).
- **Floating START button** (`start.png`). Hovering zooms it slightly and adds a green tint that is **masked to the button's non-transparent pixels** (it never tints the transparent area), all while it keeps floating.
- The **Monster logo** sits lower on the screen with a strong, near-black drop shadow; the **"Scream Machine"** title is plain white with a drop shadow.
- A small **Settings** button is pinned to the bottom. It opens a settings card with: **Calibrate Mic** (re-runs calibration any time), a **Music** mute toggle, and a **Microphone** mute toggle.

### Audio engine
- The backing track is driven by a **song config** at the top of the script — a `SONGS` array of `{ file, bpm, drumfills, riser }` entries (e.g. `{ file: 'audio/full.ogg', bpm: 100, drumfills: [4, 8, 17, 21, 33, 37, 42, 46, 50], riser: 'audio/riser.ogg' }`). `SONG` points at the active entry; add more entries and repoint `SONG` to swap the track later.
- The song file is decoded into a Web Audio buffer and looped with sample-accurate, **zero-gap** seamless playback.
- A constant (inaudible) beat grid at the song's **BPM** runs across the whole loop, derived from the audio clock — everything in the game is timed to it.
- **Touch points** (drum-fill measures that lead into a drop) come from the song's `drumfills` list, each reinforced with the song's riser.
- Pressing **START** looks ahead from the current spot to the **next drum-fill measure that still has a free "Get Ready" measure before it** (≥2 measures ahead, wrapping around the loop — e.g. at measure 9 → 17, at 36 → 42, at 49 → 4), then **queues it to play next** (a seamless measure swap), so the countdown always leads into a real drop. A short **spinner** plays, then the Prepare screen appears.
- Tapping **START** fires `scream.ogg` and `squeal.ogg` together as a stinger.

### Prepare screen
- Background: `halftone-loop.mp4` (with `halftone.jpg` as fallback); the Monster logo is retained.
- The **"GET READY TO SCREAM!!"** message (paper-scrap letters falling in one by one) appears promptly after START — no long wait.
- On the queued drum-fill measure, a beat-locked countdown drops on each beat: **3 · 2 · 1 · SCREAM!** The riser builds underneath and resolves into the drop, which starts the action.

### Action screen
- Background: `graffiti-loop.mp4` (with `graffiti.jpg` as fallback); no top Monster logo. Text uses **AntonSC**, with the loudness message in a brushed-metal gradient and a heavy drop-shadow. The message sits in a fixed-height box so the logo never shifts as the text changes.
- The large `monster.svg` icon (its paths loaded from the file at runtime, with a built-in fallback) is a **bucket** that **fills with green ink over time**: while your instantaneous loudness is **at or above 50%** the bucket **rises and never drops**; below 50% it **decays**. So the goal is to *sustain* a loud scream — go quiet and you start losing fill. The rising ink surface has a gooey **ink-drip** effect, and the silhouette carries a thick near-black outer glow. It pulses (glow only, no movement) on the beat.
- A vertical **dB meter on the left** shows **instantaneous loudness** so you can see the 50% fill threshold. It's a rectangular meter on a dark-gray panel, divided into **13 segments (10 dB each) from 0 to 130**, with a gradient fill (green low → yellow mid → red high). It only registers sound **above the calibrated ambient floor** (plus a small deadzone), so it reads empty at the calibration minimum and only starts tracking once you're clearly louder than the room.
- Hype messages escalate with how **full the bucket is** (not instantaneous loudness): **"CAN'T HEAR YOU!" (empty) → "GET LOUD!" → "MAXIMUM SCREAM!" → "MONSTROUS!!" (near full)**, refreshing **on the beat** (faster the fuller it gets — every beat → twice → four times per beat across the bands).
- **Scoring (points):** every **quarter-beat**, the level the bucket "stays at" earns points — **floor(fill% ÷ 10)** per quarter-beat (0% → 0, 10% → 1, 20% → 2 … 100% → 10) while the bucket is filling or held, and **−1** per quarter-beat while it's decaying (never below 0). The running total is shown live as **PTS** under the logo. So you score fastest by filling the bucket high *and keeping it there*. A **10-second timer** runs the round, then the results appear.
- **Cookie-clicker fallback:** when there is no usable microphone — access denied, an insecure/unsupported context, or the mic muted in Settings — the action screen becomes a **tap game** instead. The dB meter is hidden, the prompt reads **"TAP LIKE CRAZY!"**, and each tap (pointer **or spacebar**) counts toward a score (shown as **TAPS**) while filling the logo as a decaying mash-meter. The same 10-second timer applies.

### Results screen
- Background: `halftone-loop.mp4` (with `halftone.jpg` fallback) behind a `blackboard.png` scoreboard that fills almost the whole screen.
- Headed **"Your Score"**, it shows the Monster logo, the final score (**PTS**, or **TAPS** after a clicker round), a small **"Loudest: N dB"** line beneath it (the highest instantaneous reading of the round; hidden in the tap fallback), and the floating **Main Menu** button — all **inside the blackboard**. Main Menu plays the scream/squeal stinger on press; the music keeps looping the whole time.

### Everywhere
- A **vignette** darkens the edges on the Prepare, Action, and Results screens (not the menu).
- **Editable on-screen text** lives in a `UI_TEXT` object at the top of the script — the menu `title`, the `startButton` label, the `prepareMessage` ("Get Ready" text), and the `backMenuButton` label — so copy can be changed in one place.

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
- Best displayed full-screen in portrait orientation. On a **landscape** display the stage is locked to a centered 9:16 area; on a **portrait** display it fills the whole viewport and the layout flexes so nothing is clipped.

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
