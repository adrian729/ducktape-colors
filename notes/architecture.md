# Architecture: profiles and systems

How the page is put together, and how to add a profile or a colour system. Everything is in `index.html` unless named.

## The pieces

- **`PROFILES`:** every profile a view can be limited to, in menu order (the most used first), each `{ id, kind, name, short?, about, ... }`.
  - Screen (`kind: "screen"`): `primaries` and `white` (CIE xy). `rgbMatrix` works out linear sRGB to its linear RGB (Bradford to D65 if its white isn't). `matrix` overrides it where CSS Color 4 gives one (Display P3, Rec. 2020).
  - Print (`kind: "print"`): `file`, the stem of `profiles/<file>.icc` and `gamuts/<file>.json`.
  - `PROFILE_DEFAULTS`: each kind's default, its first (from `SYSTEM_DEFAULTS`, which must name the same). `WADA_PRESS`: Japan Color 2011 Coated, the press Wada's colours and Ostwald's system are tied to.
- **`gamut(id)`:** a profile's edge, made once: `most(L, h)`, `darkest`, `cusp()`, `widest(k)`, `NAMED`, `prepare()`. From `rgbGamut` (a screen, worked out) or `tableGamut` (a press, from its gamut file, which `need` fetches first).
- **`SYSTEMS`:** every system, in System switch order, each `{ id, name, file, model, make, switches }`. `fitChoices`, `filesFor`, `showLayout` and the System switch read it.
- **Views:** `makeWheel` (OKLCH), `makeNotation(colors, sys)` (Munsell, PCCS) and `makeOstwald`. Each returns `{ box, adopt, camera }`.
- **Shared by the views:**
  - `switchBar` (the groups: System, View, Profile, Options) and `profileSwitch` / `profileControl` (the Profile group, loading a press's gamut before applying it);
  - `viewSwitches`, `optionSwitch`, `viewStore` / `viewPrefs` (saved choices), `labelsSwitch`, `peelLine`, `cakeDial`, `viewCaption`, `atlasFrame` / `atlasCells` / `wadaMarks` (Ostwald and the notations);
  - `fieldSurface` (smooth fields), `solidSlot`, and the WebGL 2 solids `oklchSolidSteps` / `ostwaldSolidSteps`.
- **Saved choices:**
  - shared by the systems, under `systems-`: `range` (screen or print), `screen` and `print` (the profile chosen for each), the view, the slice, the cut and `v-<view>-<option>`;
  - each system's own, under its prefix: the level, the pages.
- **CMYK estimates:** `estimateCMYK(oklch, id = chosenPrint())`, through the print profile's B2A1 table. `loadPrintTable(id)` fetches it once, as a byte range of the `.icc`. Ostwald's use `WADA_PRESS`.

## Adding a screen profile

1. An entry in `PROFILES`, `{ id, kind: "screen", name, about, primaries, white }`, where its usefulness puts it.
2. Check that it fits the scale. The views draw every profile at the widest screens' scale: the OKLCH rim is chroma 0.47 (Rec. 2020's most), and the notations' `JMAX` is the most rings any screen profile reaches. A profile far wider, such as ProPhoto (imaginary primaries, chroma 1.49), would need a new scale. A sliver beyond, as Display P3's and NTSC's reds are, is fine.

## Adding a print profile

1. Put the ICC file in `profiles/`, unaltered, with `<file>.LICENSE.txt`: source, provider, printing condition, SHA-256 and the licence's words. The licence must allow sharing it unaltered. PSO Coated v3's, for one, doesn't.
2. It needs 16-bit lut16 (ICC v2, `mft2`) A2B1 and B2A1 tables. Both the script (`cmyk_to_oklch.Lut3`) and the page (`loadPrintTable`) read only those. ICC v4 profiles (`mAB` / `mBA`, e.g. GRACoL 2013, SWOP 2013) need a parser added to both first.
3. Run `python3 scripts/print_gamut.py profiles/<file>.icc` (about a minute). It writes `gamuts/<file>.json` and reads the ink limit from the profile.
4. An entry in `PROFILES`, `{ id, kind: "print", name, short, about, file }`, where its usefulness puts it.
5. Check that it fits the scale (as for a screen).

## Adding a system

1. An entry in `SYSTEMS`, where its usefulness puts it in the switch, with:
   - `file`: what its view can't do without, fetched first;
   - `model()`: its own colours, worked out once from that file;
   - `make(colors, model)`: its view;
   - `switches`: whether it has a profile, its slices, its views, and those with a space switch.
2. Its view:
   - **A notation** (pages of hue round a grey axis, levels, rings out by chroma or saturation, as Munsell and PCCS are): reuse `makeNotation`. Its model is `notationCells(colors, key, chips, greys, PAGES)` (cells, profile fits, scale, nearest chip), plus the fields `makeNotation` reads. `munsellSystem` is the model to copy:
     - `key`, `title`, `solid`, `solidShort`, `source`, `ownWhat`, `ringWord`;
     - `PAGES`, `HOME`, `TURN`, `LEVELS`, `levelWord`, `levelLabel(i)`, `levelText(i)`, `levelAria`;
     - `DISC`, `DIAL`, `SOLID` (the hues named in each place), `pageName(k)`, `pageTitle(k)`, `hueTitle(k)`;
     - `lines(c)`, `ownLines(o, where)` (the info panel);
     - `flat` (has a Flat view), `tones` (PCCS's tone map).
     - Each of Wada's colours gets `c[key] = { k, i, lv, j, hue, share, code, exact }`.
   - **Anything else:** a maker of its own returning `{ box, adopt, camera }`, built from the shared pieces above. Use `switchBar` for its switches, and `profileSwitch()` with `profileControl` if it has a profile, or `profileSwitch(why)` to show it off.
3. Its data script in `scripts/`, and its notes in `notes/`.

## Review: what's shared, what isn't

- **Shared enough:** the switches, saved choices, profile handling, captions, dials, peel, the solids' builder and renderer, the smooth fields.
- **Still repeated in each maker:**
  - `setRange` / `setView` / `adopt` / `choose` / `showLook` / `loadLook`;
  - the 3D solid's making and keeping (`solidMade`, `remakeSolid`, `makeSolid3d`);
  - the level control and its outline (`halfWidth`).

  They follow one pattern with different names, and could become one view controller taking hooks. That's worth doing before a fourth kind of view, not for a notation (which reuses `makeNotation`).
- **Not to generalise:** the scripts. Each system's data comes from its own source and maths, run once, offline.
- **Performance:**
  - A screen gamut is worked out on first use: under 50 ms to switch to one the first time, then cached. The shown one is prepared while the page is idle.
  - A press's gamut file (350 to 440 KB) is fetched only when it's first chosen.
  - The estimates' table is a byte range of the `.icc` (340 to 460 KB). A server without range requests, such as `python3 -m http.server`, sends the whole file (2 to 8.6 MB). GitHub Pages supports ranges.
