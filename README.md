# Ducktape Colours

A static site for exploring colour systems: Ostwald's colour solid (in Wada Sanzō's notation), OKLCH, Munsell's solid and PCCS's (with its tones). Each is shown flat (every colour at once by hue and chroma), as 2D slices (across the axis, one level, or through it, two opposite hues' pages) and as a 3D solid, with a smooth or gridded background, faded or at full strength, and a Screen or Print range. Sanzo Wada's colours, from *A Dictionary of Color Combinations*, are plotted on every system as a dataset to compare against.

It started from the Colour systems page of [wada-sanzo-colors](https://github.com/adrian729/wada-sanzo-colors).

## Run locally

It's a single `index.html` that loads JSON files (`colors.json` at once, the others when a view first needs them), so it has to be served over HTTP:

```sh
python3 -m http.server
```

Then open http://localhost:8000.

## Data

- `colors.json`: Wada's 171 colours (159 from Volume 1, and 12 only in Volume 2, marked `"volume": 2`), each with its name as printed in the book, its CMYK as given in the book, and OKLCH.
  - 36 of them also have `atlas`: where Plate 8 of the 1935 colour atlas puts them (see Sources). It gives one of the plate's 12 hues (`hue`, translated), its tone there (`tone`: tint, full colour or shade), and the plate's Japanese name and its reading (`ja`, `reading`). Our only copy of the plate is a black-and-white scan, so the colours are matched by name. Its Grayish Lavender is our Grayish Lavender - A, the one whose hue fits. Its purple-blue shade, Coolie Blue (花色), has no colour of that name, so it is matched by us and marked `inferred`, with the plate's English name in `plateName`. It is Violet Blue: Wada's own chips (Seigensha's Volume 2) call Violet Blue 花色, and it is the one candidate that fits the plate, a shade as dark as the full colour or darker, with the black content of the other shades, between the blue and purple shades in hue, and black in the scan (see notes/ostwald.md). Where it gives a second spelling or a qualifier (鴾色 (鴇色), 萠木色 (裝束の)), only the first spelling is kept.
- `print-gamut.json`: what the press prints (Japan Color 2011 Coated, at most 350% ink): the most chroma at each OKLCH lightness and hue, for the OKLCH view's Print range, and the full colours and tint edge the Ostwald view measures Wada's colours against (`scripts/print_gamut.py`; see `notes/ostwald.md`).
- `ostwald-cmyk.json`: CMYK estimated for every cell of the Ostwald view, by its Ostwald code, for the cells none of Wada's colours fill (`scripts/ostwald_cmyk.py`).
- `munsell.json`: for the Munsell view, each colour's Munsell notation, and the system's own colours, the 1943 renotation's, in our space (`scripts/munsell.py`; see `notes/munsell.md`).
- `pccs.json`: for the PCCS view, each colour's PCCS notation and tone, and the system's own colours and tones, converted from Munsell's (`scripts/pccs.py`; see `notes/pccs.md`).

OKLCH is computed from CMYK through the Japan Color 2011 Coated profile (`scripts/cmyk_to_oklch.py`). The Ostwald view's estimated CMYK goes the other way, through the same profile (`scripts/ostwald_cmyk.py`); re-run it after changing `colors.json`, `print-gamut.json` or the Ostwald computation (`notes/ostwald.py`). The OKLCH view's, for any point of its slices or cell of its solid, is estimated the same way by the page itself, from the profile's table. `print-gamut.json` only changes with the profile or the ink limit; `scripts/print_gamut.py` takes a few minutes. Re-run `scripts/munsell.py` after changing `colors.json`; it needs colour-science and scipy, which uv fetches: `uv run -q --python 3.12 --with colour-science --with scipy python scripts/munsell.py` (about half a minute). Then re-run `scripts/pccs.py` the same way (about two minutes).

## Colour ranges

- **The OKLCH, Munsell and PCCS views** show one of two ranges (in the Munsell and PCCS views, the range is which of the system's own colours they show):
  - Screen (the default): Rec. 2020, the widest standard screen gamut, which holds all of Wada's colours and everything the press prints;
  - Print: what the press prints, as Wada's colours were printed. The press's darkest colour is at lightness 0.226, so the darker levels are empty.
- **Nothing is clipped before the screen.** The canvases and the 3D views draw in 16-bit floating point, which holds any colour, so each reaches the screen as CSS gives it, and a screen that shows fewer colours clips the rest itself. A browser without it gets Display P3, else sRGB, which clip what they don't hold.
- **Faded backgrounds** (the Strength switch) show the system's colours at a third of their strength over the page. Colours far in lightness from the page would all fade to one grey, so their chroma is raised first, the more the further they are: on the light page the dark colours, on the dark page the light ones (`fadeBoost` in `index.html`).

## Sources

- Wada's colours, Volume 1: the dataset from [mattdesl/dictionary-of-colour-combinations](https://github.com/mattdesl/dictionary-of-colour-combinations) (MIT, see `DATA-LICENSE.txt`), itself based on [dblodorn/sanzo-wada](https://github.com/dblodorn/sanzo-wada), checked against the book: names corrected to the printed spelling and a few CMYK values fixed.
- Wada's colours, Volume 2: read from the National Diet Library's scans of his 配色総鑑 B篇 (1934–35), which it marks public domain, and checked against the printed book (see wada-sanzo-colors' `vol2/`).
- Plate 8 ("Colour names and their hues") of 標準色彩圖集 (*Hyōjun shikisai zushū*, a standard colour atlas, 1935) by Hama Yaohiko and Wada Sanzō, read from the National Diet Library's scan. Its names are those of Wada's 色名總鑑 and 色名帖. It even shares the book's misprints, Rosolanc Purple and Antwarp Blue.
- The Munsell renotation data (Newhall, Nickerson and Judd, 1943), as colour-science has it (BSD licence), which also converts our colours to Munsell notation.
- PCCS's conversion from and to Munsell's notation, and its tones' representative colours and coordinates: Mituo Kobayasi and Kayoko Yosiki (小林光夫, 吉識香代子), "Mathematical Relation among PCCS Tones, PCCS Color Attributes and Munsell Color Attributes" (PCCSトーン, PCCS三属性値, およびマンセル三属性値間の数学的関係), *Journal of the Color Science Association of Japan* 25(4), 2001, pp. 249–261, from the National Diet Library's digital collections. PCCS is the Japan Color Research Institute's.
- The Japan Color 2011 Coated ICC profile, from the ICC Profile Registry, shared unaltered under its licence (`profiles/`).
- The DM Mono font, under the SIL Open Font License (`fonts/OFL.txt`).
- The Noto Serif JP font, under the SIL Open Font License (`fonts/OFL-NotoSerifJP.txt`).
