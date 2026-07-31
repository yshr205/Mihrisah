# Mihrişah — build record

Two pages, no frameworks, no build step. Plain HTML/CSS/JS in single files so
nothing can break in transit.

- `index.html` — the main page
- `sky.html` — the castle
- `photos/`, `voices/` — everything they load

---

## What the site does, page by page

### index.html

1. **Intro overlay** — the first tap unlocks audio (iPhones block sound until you touch the screen).
2. **Hero** — her name, the date, the 21, fireworks. Four hidden stars are scattered here and further down.
3. **Bouquet** — lit from above, petals falling, motes drifting through the beam.
4. **The letter** — a 3D envelope with a wax seal that opens on tap, then the letter itself with photos beside the paragraphs, a table of polaroids you can pick up, and the video.
5. **The party** — the five Monster High girls around a goth cake with 21 candles. They speak in order with the real voice clips, subtitles underneath, and they don't start until you actually reach the scene.
6. **The wishes** — the six "may…" lines, then the director line.
7. **Closing**, the link to the castle, and the footer.

Hidden: find all four stars and something happens. Tap the cake past 21 times and something else does.

### sky.html

Dracula's castle under a blood moon. Gargoyles you can wake, fireworks, drifting
fog and ash, an organ playing Toccata in D minor, secrets to find, and the jump
scare at the end.

Above the tallest spire: the Eye of Sauron.

---

## The Eye — how it's built

Your generated file was a 30-frame animation. Everything below sits on top of it.

**Placement.** Not guessed. I put a coordinate grid over `castle.jpg` and read the
needle tip off it: **x 0.627, y 0.398** of the image. I also measured where the flame
sits inside its own file, because the artwork has transparent padding — aiming at
the file's box instead of the fire is what made the first attempts float too high.
Both numbers are named constants in `sky.html`:

```js
const TIPX = .627, TIPY = .398, GAP = .008;   // GAP = clearance above the tip
const VB = .799, HC = .480;                   // where the flame sits inside the PNG
```

Change `GAP` alone to raise or lower it. It holds on any screen size because
everything is a fraction of the displayed image, not pixels.

**The gaze.** The source animation only slid the pupil ±6% of the iris, which read
as a twitching line rather than an eye. I tried retouching the artwork to drive a
pupil of my own and every attempt left a ghost slit where the original had been, so
I threw that out. Instead the whole eye now swings in 3D on a vertical axis — the
pupil genuinely travels across the fire and the far side foreshortens the way an
eyeball does. It holds, darts, holds. 12-second cycle. The artwork is untouched.

**The layers, back to front:**

| Layer | What it does |
|---|---|
| `eyeCast` | wide wash that lights the clouds, so it reads as a light source |
| `eyeBeam` | searchlight raking down over the castle, sweeping on a 19s cycle |
| `eyeHalo` | blurred heat haze drifting and rotating off the edges |
| `eyeF` | the artwork, with the 3D gaze and an off-beat flame flicker |
| `eyeGlow` | close orange glow, pulsing |
| `eyeFlare` | every 13s it blazes white-hot for a moment, then settles |
| `eyeSparks` | nine embers rising at different speeds |

All of it switches off automatically if the phone starts dropping frames.

---

## Problems that came up, and what was actually wrong

**Black box around the eye.** `mix-blend-mode: screen` was supposed to erase the
black background, but the breathing animation created its own compositing layer, so
the image was blending against the eye's own container instead of the sky. Fixed by
baking real transparency into the file.

**Faint square around the eye.** `mix-blend-mode` + `filter: blur()` + `will-change`
on the same element. iOS Safari gives up on blending that and composites the layer's
bounding rectangle instead — a visible box. Removed the blend mode from every layer
of the eye (it was doing nothing anyway) and rebuilt the light beam as SVG.

**Voices too quiet, then silent.** First the silence-trimming filter had destroyed
the clip volumes — one was at −55 dB. Reprocessed from the originals. Then ducking
the piano was silencing the girls too, because both ran through the same output.
Voices now have their own bus.

**The party never started.** The scene is taller than the screen, so the trigger
that waited for "half of it visible" never fired. Now it fires as soon as any of it
enters, and tapping the cake starts it too.

**Stars wouldn't shrink.** Three separate causes stacked: the ✦ glyph renders far
larger than its font size, the class name collided with the credits styling, and a
CSS shorthand placed after a related property was silently resetting it. Rewritten
as a vector star with an explicit size.

**Fireworks in slow motion.** A frame-skip I'd added was halving the physics, not
just the drawing. Every particle system now runs on the real clock, so it looks the
same at 60fps or 30.

**Music dead after coming back from the castle.** Safari restores the page from
memory with the volume still turned down. Added restore handlers plus a watchdog
that turns it back up if nothing is speaking.

**Photos blocking scroll.** They were swallowing every touch. Now: swipe up or down
scrolls the page, a sideways flick drags the photo, and holding for about a second
picks it up. The letter says so.

**Video wouldn't play on iPhone.** It was 10-bit H.264, which iOS won't decode.
Re-encoded to baseline.

**Voices clipping.** They're boosted 2.6× to be audible on a phone speaker, with
nothing catching the peaks — the loud syllables would have crackled. Added a limiter.

**The Show.** A 90-second cinematic on the castle page. It looked bad. Removed, and
the page restored from backup.

---

## Final check

| | index.html | sky.html |
|---|---|---|
| JavaScript | valid | valid |
| Duplicate ids | none | none |
| Duplicate animation names | none | none |
| Animations used but never defined | none | none |
| Animations defined but never used | none | none |
| Broken inline SVG | none | none |
| Unbalanced tags | none | none |
| Missing files | none | none |

No collisions between the new code and the old. The Eye's styling all lives under
`.eye`, its animation names are all prefixed `eye…`, and nothing else on the page
uses those names.

**Weight:** index 7.3 MB total, but only the top of the page loads up front — 30 of
31 images load as she scrolls to them. Castle page 0.9 MB.

**iPhone:** audio unlocks on first tap and plays with the ringer on silent, video
plays inline, touch listeners are passive so scrolling stays smooth, the page
survives being backgrounded, and quality drops automatically if frames start
dropping.

---

## Before sending

1. Delete `photos/eye1.*`, `eye2.*`, `eye3.*`, `moon.jpg` — unused. I couldn't
   remove them myself; the folder blocks deletions.
2. Upload `index.html`, `sky.html`, `photos/`, `voices/`. Give Pages two minutes.
3. Open the live link on your phone **from inside an Instagram DM to yourself** —
   that's the exact browser she'll use.
4. Check: sound with the ringer on silent, fireworks smooth, envelope opens, photos
   lift on a long press, video plays, the girls are all audible, the Eye sits on the
   spire and moves, the scare fires, and the music is still playing when you come
   back from the castle.

`voices/scream.mp3` doesn't exist and doesn't need to. If it's ever added it gets
used; otherwise the scream is synthesized. The 404 in the log is expected.
