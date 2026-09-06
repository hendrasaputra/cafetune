# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single static page: `index.html` holds the markup, CSS and the whole audio engine.
There is no build step, no package manager, no dependencies, no tests. Everything else in
the repo is deployment furniture (`CNAME`, `.nojekyll`, `og.png`, `sitemap.xml`).

## Running it

```bash
python3 -m http.server 8000    # then open http://localhost:8000
```

`open index.html` works too, but serve over HTTP if you touch anything that cares about
origin. Audio is gated behind the "Open the cafe" button because browsers block
`AudioContext` until a user gesture — the page will always be silent until that click.

Verification is by ear and eye: click Open, listen, watch the canvas, toggle `--mode=`,
reload to confirm the theme persisted in `localStorage` (`pt-theme`).

## Deploying

Pushing to `main` publishes to `https://cafetune.hensap.id`. Two mutually exclusive routes:
Settings → Pages → *Deploy from a branch* (in which case `.github/workflows/deploy.yml` is
dead weight), or *GitHub Actions* (in which case the workflow does it). Don't enable both.

## Architecture

The script is one IIFE split into eight numbered `/* === N. NAME === */` blocks. Grep for
the number to jump.

1. **Audio graph** — one `AudioContext`, five gain buses (lead/comp/bass/drums/cafe) into a
   `DynamicsCompressor` limiter into `master`. Pulse waves are built as `PeriodicWave`
   partials; the reverb impulse is generated noise. The band is deliberately **dry** — only
   the cafe bus feeds the convolver, because real chips faked depth with delayed re-triggers
   (see the echo in `melody()`), not reverb.
2. **Instrument voices** — `chip()`, `bassNote()`, `kick()`/`snare()`/`hat()`/`ride()`.
   Every voice creates its nodes, schedules its envelope at an absolute time `t`, and stops.
   Nothing is pooled or reused.
3. **Harmony** — chord qualities (`CH`), their scales (`SC`), and `PROGS`, a list of bar-wise
   progressions in scale degrees relative to `keyRoot`. `advanceArrangement()` is the
   long-form shape: every 24–40 bars it strips to bass+drums and rebuilds in 4-bar steps, so
   the music doesn't sit at one intensity. Counts are multiples of 4 on purpose.
4. **The players** — `walkingBass`, `comping`, `melody` (motif + `mutate()`), `drums`.
   Each takes the chord and the bar's timing and emits notes.
5. **The room** — not noise. Each murmur is a real `utterance()`: sawtooth glottal source
   through three moving formant filters (`VOWELS`), chopped into syllables, panned and sent
   to reverb by distance. Plus foley (`clink`, `grinder`, `doorBell`, …).
6. **Scheduler** — the only timing mechanism. A `setInterval(tick, 60)` lookahead calls
   `scheduleBar()` for any bar starting within 400ms, which schedules *everything* for that
   bar at absolute `AudioContext` times. **Never schedule audio from a timer directly** —
   `setInterval` only decides *when to schedule*, the Web Audio clock decides when to sound.
   `stepT()` applies swing.
7. **Display** — canvas driven by `requestAnimationFrame` off the `AnalyserNode` plus the
   `timeline` ring buffer. Colours are read from CSS custom properties via `refreshColors()`,
   so the theme toggle calls `window.__cafetuneRepaint`.
8. **Controls** — faders write into the `mix` object. `mix.busy` is not a volume: it is the
   density knob threaded through note probability, drum fills, and crowd rate.

Adding a new instrument means a voice in §2 and a player function in §4 called from
`scheduleBar()`. Adding a chord quality means entries in `CH`, `SC` and `NAME` together.

## Style

Vanilla ES5-flavoured JS (`var`, `function`), no modules, no transpiling — it ships as-is.
Comments in the script explain *why* a musical or acoustic choice was made; match that
register rather than annotating what the code does.

The two `:root` blocks in `<style>` are copied from hendrasaputra.com. If the parent site
restyles, update both here.

## Licence boundary

Code is MIT. The names ("Cafetune", "Pixelized Thoughts", "hensap"), `og.png`,
`favicon.svg`, and the written copy on the page are reserved — don't treat them as
freely reusable when refactoring or when generating derivative assets.
