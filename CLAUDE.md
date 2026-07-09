# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A small self-contained HTML slide deck for a **Socratix AI office-hours session on July 10, 2026**. The slides exist to **foster dialogue**, not to present: the session's goal is to draw out problems leaders have that AI could solve — or to spark broad, imaginative thinking about what AI could do — so Miguel can take that input, run research spikes, and come back with a way through.

The narrative arc (six slides): cover → **how the hour works** (they talk, we listen, Miguel builds) → **warm-up question** → **two directions to think in** (blue ocean = imagine broadly / red ocean = fix what already hurts) → **the four questions** and → **what happens next**.

The centerpiece is slide 5, a two-column blue-ocean / red-ocean grid built from four discussion questions: the **left/blue** column holds the imaginative, broad prompts ("Wait, AI can do that now?", "The Magic Wand"); the **right/red** column holds the known-problem, optimization prompts ("The Persistent Pain", "The Visibility Black Hole"). Slide 4 introduces that same blue/red framing at a lower altitude. If you edit the questions, keep the blue = imaginative / red = optimization split intact — it's the point of the slide.

There is **no build system, no framework, no dependencies, and no tests** — everything is vanilla HTML/CSS/JS in `index.html`. The only external asset is `assets/socratix-logo.png`.

The deck deliberately reuses the styling and interaction engine of the sibling talk deck at `../presentation/index.html`. When in doubt about a visual convention, look there — it's the source of the shared design language (theme variables, stage scaling, step reveals, HUD).

## Running it

Open `index.html` in a browser. It works from `file://`, but serve over HTTP if you add anything that fetches (the logo loads fine either way):

```
python3 -m http.server   # then open http://localhost:8000
```

Navigation: `→ / ↓ / Space / Enter / PageDown` advance, `← / ↑ / PageUp` go back, `Home`/`End` jump to first/last, `f` toggles fullscreen. Clicking the left 22% of the screen goes back, anywhere else advances. Elements with class `noadvance` (and `<video>`/`<audio>`) swallow clicks so they don't advance.

## Architecture (`index.html`)

Three regions in one file: a `<style>` block in `<head>`, the slide markup, and one strict-mode IIFE `<script>` at the bottom.

- **Stage**: a fixed `1600×900` `#stage` scaled to the viewport by `fit()`. **Design everything against those pixel dimensions** — not viewport units.
- **Scenes**: each slide is a `<section class="scene" id="sN">`. `scenes` is the array of all `.scene` elements **in document order — order in the DOM is the slide order.**
- **Step reveals**: elements marked `[data-step]` appear one at a time on each advance (`next()` adds `.on`). `[data-anim="left|right|down|pop|stamp|none"]` sets the entrance transform. No `data-anim` = default rise-up.

### Key difference from the `../presentation` deck

This deck's engine is intentionally **generic** — there are no hardcoded per-scene animation branches. `go`/`next`/`prev`/`resetScene`/`revealAll` only toggle `[data-step].on`; they don't know about individual scene IDs. That makes adding, removing, and reordering slides safe — no scene-specific code will silently break.

The **one** thing that is index-coupled is the section-dot HUD: `SECTION(i)` maps a scene index to how many of the three dots are lit via numeric thresholds. If you reorder or change the slide count, update those thresholds (and the `#counter` / `SECTION` logic reads `scenes.length`, so the "NN / NN" counter self-adjusts).

If you port a scene-specific animation over from `../presentation` (e.g. a chart draw or a canvas sim), wire its trigger through a new `onReveal(el, sceneId)`-style hook rather than reintroducing ID branches into `go()`.

## Conventions

- Theme colors are CSS custom properties on `:root` (`--bg`, `--amber`, `--blue-lt`, `--red-lt`, `--ink`, `--muted`, `--faint`, `--line`, etc.), copied verbatim from `../presentation`. **Reuse them rather than hardcoding hex** so the two decks stay visually identical.
- Reusable slide-type classes already defined: `.cover`, `.sec` (+ `.secnum`, `.agenda`), `.qscene` (+ `.badge`, `.bigq`, `.prompts`), `.cols`/`.col` (two-column compare, with `.blue`/`.red` variants — used on slide 4), `.qgrid`/`.qcol` (the four-question blue/red grid with `.ohead`/`.oname`/`.otag` headers and stacked `.qblock` items of `.qk`/`.qq`/`.qh` — used on slide 5), `.closing`. Type scale: `.kicker`, `.display`, `.sub`, `.foot`. Prefer composing these over inventing new ones.
- Blue-ocean columns use `--blue`/`--blue-lt`, red-ocean columns use `--red`/`--red-lt` — this color coding carries meaning (imaginative vs. optimization), so don't recolor it casually.
- The ember canvas background (`#embers`) and cover word-rise animation are copied from `../presentation`; leave them as-is unless changing the shared look.
- Timed animations must go through the `later`/`clearTimers` helpers inside the IIFE so `go()` can cancel them on scene change.
