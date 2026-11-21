# Notes on the colour systems

How each system's view is built, and how Wada's colours are placed in it.

## Files

- [architecture.md](architecture.md): how the page is put together (profiles, systems, shared pieces), and how to add a profile or a system.
- [ostwald.md](ostwald.md): Wada's colours placed in Ostwald's colour system as his 1935 supplement draws it, as in the Ostwald view, and how well that fits his own data.
- [ostwald.py](ostwald.py): the script behind ostwald.md. Run `python3 notes/ostwald.py`.
- [munsell.md](munsell.md): Wada's colours placed in Munsell's system, as in the Munsell view, and how they were measured. Its script is `scripts/munsell.py`.
- [pccs.md](pccs.md): Wada's colours placed in PCCS and its tones, as in the PCCS view, and why it's built on Munsell's notation. Its script is `scripts/pccs.py`.

## Sources

- **Wada's 1935 supplement:** 『配色總鑑附說』 ("Supplement to the Haishoku Sōkan"). It was compiled by Wada and published by Hakubisha (博美社) on 20 November 1935, not for sale. NDL scan: https://dl.ndl.go.jp/pid/1903674. Page numbers in these notes are its own.
- **Our data:** `colors.json` (Wada's colours, with OKLCH), and the systems' files (see the README).

Wada's other texts, the notes on his combinations and the Volume 2 data are in [wada-sanzo-colors](https://github.com/adrian729/wada-sanzo-colors).
