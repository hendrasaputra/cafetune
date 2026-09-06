# Cafetune

A café that never closes, playing music that never repeats. Live at **https://cafetune.hensap.id**

Nothing on the page is a recording. A five-piece chiptune band and a room full of talking
people are both synthesised in the browser, one bar at a time, using the Web Audio API.
No libraries, no audio files, no network calls after the page loads.

Part of [Pixelized Thoughts](https://hendrasaputra.com/).

---

## Running it

```bash
python3 -m http.server 8000    # then open http://localhost:8000
```

Everything is in `index.html` — page, styling and audio engine. There is no build step and
nothing to install. The page stays silent until you press **Open the cafe**, because
browsers refuse to start an `AudioContext` without a user gesture.

---

## How it works

The script is one closure split into eight numbered blocks. What follows is the reasoning
behind each; the code has the details.

### 1. The audio graph

One `AudioContext`. Five gain buses — lead, comping, bass, drums, café — all landing in a
compressor acting as a limiter, then the master gain. Nothing is loaded; the two buffers
that exist are generated at startup: two seconds of white noise for percussion and foley,
and a 1.4-second exponentially-decaying stereo noise burst used as the reverb impulse.

The pulse waves aren't `type="square"`. Each duty cycle (50%, 37.5%, 25%, 12.5%, 6.25%) is
built as a `PeriodicWave` from its Fourier coefficients, which is what gives the different
pulse widths their distinct nasal colours.

**The band is dry.** Only the café bus feeds the reverb. Real chips had no reverb — depth
came from re-triggering the same note quieter a moment later, and that is what the melody
does instead.

### 2. Instrument voices

`chip()`, `bassNote()`, `kick()`, `snare()`, `hat()`, `ride()`. Each one creates its own
oscillators and gains, schedules an envelope against an absolute time, starts, stops, and is
garbage collected. Nothing is pooled or reused. A note can switch pulse width partway
through (`dutySeq`) or scoop up into its pitch from below (`slide`) — both tracker tricks.

Percussion is filtered noise: the ride is a narrow bandpass at 5.2 kHz, the snare at 1.85 kHz,
the hat a highpass at 8.2 kHz. The kick is a pitch-swept sine with a noise transient on top.

### 3. Harmony

Five chord qualities (`maj7`, `m7`, `dom7`, `m6`, `m7b5`), each with its own scale, so a
melody note is always chosen from a scale the chord actually belongs to rather than from one
global key. Six progressions are stored as scale degrees relative to `keyRoot`, one entry per
bar, so transposing is one number. When a progression ends, a *different* one is picked — it
never repeats itself back to back.

Above that sits the arrangement clock, which is what stops endless generative music from
becoming tiring. Every 24–40 bars it strips back to bass and drums for four bars, brings the
comping back for four, the lead for four, then returns to full. Every count is a multiple of
four so a change never lands mid-phrase.

### 4. The players

**Bass** walks. Beat one is the root. Beats two and three are chord tones, occasionally
displaced by a step. Beat four deliberately lands a semitone either side of the *next* bar's
root, so the following downbeat resolves. Every pitch is folded back into A1–E3, a real
double-bass register, and kept within a sixth of the previous note so the line doesn't leap.

**Comping** picks one of eight syncopated slot patterns, voices the chord around the fifth
octave, and often drops the root up an octave. Each stab is rolled independently against the
busy knob, so the same pattern is sparser in a quiet room.

**Melody** is the only part with memory. A motif is a rhythm (one of ten sixteenth-note
patterns) plus a contour of scale-degree steps plus two pulse widths. Every other bar the
motif mutates: one note moves, sometimes a note is dropped, sometimes the whole contour
shifts. After four to eight bars it is discarded and a new one is invented. Notes landing on
strong beats are snapped to the nearest chord tone so the harmony still reads through the
mutation. Then the channel echo: the same note again at 40% an eighth later, and sometimes a
third at 16%. At the end of a phrase the density drops so the line breathes, and a fast
six-note run sometimes leads into the next one.

**Drums** play a jazz ride pattern on 1, 2, the swung and-of-2, 3, 4 and the and-of-4.
Everything else — hats, ghost snares, accented snares, kicks — is probabilistic and scaled by
the busy knob. The last bar of each eight gets a snare fill half the time.

### 5. The room

A crowd is not a hiss. Every murmur here is a synthesised utterance: a sawtooth buzz plus a
little breath, pushed through three bandpass filters parked on real vowel formants. An
envelope chops it into roughly four syllables a second, never quite falling to silence
between them, and each syllable slides the filters onto a different vowel. The pitch rises
early then falls — except one time in six, when it rises instead and the sentence becomes a
question.

Each voice gets a pan position and a distance. Distance lowers the level, raises how much
goes to the reverb, closes a lowpass so consonants are lost, and past a certain point drops
the third formant entirely — so far-away people are heard as room rather than as speech. A
laugh is several short utterances falling in pitch.

Glass and china are not noise either. `struck()` sums a handful of sine partials at
inharmonic ratios, with higher partials decaying faster, which is how a real object rings.
The same function makes the clink, the cup on the saucer, the spoon in the mug and the door
bell — only the base frequency, the ratios and the decay differ. Chair scrapes, the steam
wand and the grinder are filtered noise, since that is genuinely what they are.

Underneath all of it are two near-silent noise beds for air handling and traffic outside,
which slowly drift up and down over half a minute so the room feels like it fills and empties.

### 6. The scheduler

This is the only timing mechanism, and the one thing to understand before changing anything.

A `setInterval` fires every 60 ms and asks: does any bar start within the next 400 ms? If so,
`scheduleBar()` advances the chord and the arrangement, then hands every part the bar's start
time and lets each schedule *all* of its notes at absolute `AudioContext` times.

So the interval decides only *when to schedule*; the audio clock decides when things sound.
Nothing is ever played from a timer callback directly — that would inherit the jitter of
`setTimeout` and the music would breathe unevenly. Swing lives in `stepT()`, which pushes
off-beat sixteenths late by a fixed fraction.

The café runs on ordinary timers instead, because a room has no grid. A session counter
guards the recursive callbacks so closing up doesn't leave an old room chattering underneath
a new one.

### 7. The display

A canvas driven by `requestAnimationFrame`, reading an `AnalyserNode` for the spectrum. Bars
are quantised to six-pixel blocks so it reads as pixel art rather than as a smooth meter, and
the frequency index is raised to a power so the low end gets the space it deserves.

The patrons are rectangles; how many stand up follows the busy fader. The beat marker and the
chord and bar readouts come from `timeline`, a ring buffer of the last eight scheduled bars —
the display looks up which bar is playing *now* rather than being told.

Colours are read out of the CSS custom properties, so the theme toggle only has to swap
`data-theme` and call the repaint hook.

### 8. Controls

The faders write into one `mix` object. Five of them are volumes. `busy` is not — it is a
single density number threaded through note probability, comping stabs, drum fills, crowd
burst size, the gap between foley sounds, the room tone level and the number of patrons
drawn. It is the closest thing here to a single "how alive is this place" dial.

Changing key sets `progPos` past the end of the progression, so the switch happens at the
next loop rather than mid-phrase.

---

## Changing it

| Want to | Do this |
| --- | --- |
| Add an instrument | A voice in §2, a player function in §4, one call in `scheduleBar()`. |
| Add a chord quality | Entries in `CH`, `SC` and `NAME` together, or the name readout breaks. |
| Change the feel | `swing` in §6, `RHY` and `COMP_SLOTS` in §4. |
| Change the arrangement | `advanceArrangement()` in §3 — keep bar counts multiples of four. |
| Restyle | The two `:root` blocks at the top of `<style>`, copied from hendrasaputra.com. |

---

## What's in here

| File | What it does |
| --- | --- |
| `index.html` | The whole thing. Page, styling and audio engine in one file. |
| `CNAME` | Tells GitHub Pages the site answers to `cafetune.hensap.id`. Don't rename it. |
| `.nojekyll` | Stops GitHub running the files through Jekyll, which it does by default. |
| `og.png` | The 1200×630 image that shows when the link is shared. |
| `favicon.svg` | Pixel coffee cup, drawn in the site's amber and coral. |
| `robots.txt`, `sitemap.xml` | Lets search engines index the page. |
| `.github/workflows/deploy.yml` | Only used if Pages is set to deploy via GitHub Actions. |
| `LICENSE` | MIT for the code, with the brand and copy carved out. |

Pushing to `main` publishes the site.

---

## Licence

The code is MIT licensed. Read `LICENSE` for the full text. In short: take it,
change it, ship it, sell it — just keep the copyright notice in the source.

The licence covers code and nothing else. These stay © 2026 hendrasaputra.com,
all rights reserved:

- the names "Pixelized Thoughts", "Cafetune", "hensap" and "Hendra Saputra"
- the `hensap@pixelized:~$` mark and the visual identity around it
- `og.png` and `favicon.svg`
- the written copy on the page and in this README

So you are welcome to fork the audio engine and the layout. Give the result
your own name and your own artwork.

The four typefaces load from Google Fonts at runtime and are never copied into
this repository, so nothing here redistributes them. They are IBM Plex Mono,
VT323, Silkscreen and Newsreader, all under the SIL Open Font License.

© 2026 [hendrasaputra.com](https://hendrasaputra.com/)
