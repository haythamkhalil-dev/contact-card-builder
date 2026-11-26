# `lib/` — vendored third-party libraries

These files are **committed to the repo on purpose** so the app is fully
self-contained: GitHub Pages serves them from this same site, so there's no CDN
dependency and the app works offline. There is **no `npm install`** and no build
step — the browser loads these `.js` files directly via `<script>` tags.

## Libraries used

| File                  | Library / version       | Used for                                       | Source |
|-----------------------|-------------------------|------------------------------------------------|--------|
| `qrcode.js`           | qrcode-generator 1.4.4  | Building the QR matrix from the vCard text      | https://cdn.jsdelivr.net/npm/qrcode-generator@1.4.4/qrcode.js |
| `jspdf.umd.min.js`    | jsPDF 2.5.2             | Generating the PDF; QR drawn as vector squares  | https://cdn.jsdelivr.net/npm/jspdf@2.5.2/dist/jspdf.umd.min.js |
| `amiri-font.js`       | Amiri Regular (SIL OFL 1.1), subset | Embedded Arabic font so the PDF can render Arabic | https://github.com/google/fonts/tree/main/ofl/amiri |
| `amiri-regular.woff2` | Amiri Regular (SIL OFL 1.1), Arabic subset; exact version not recorded (fontsource `@latest`, downloaded 2026-07-01) | Web font so the live **preview** renders Arabic in the same face as the PDF | https://cdn.jsdelivr.net/fontsource/fonts/amiri@latest/arabic-400-normal.woff2 |
| `readex-pro-latin-wght.woff2`, `readex-pro-arabic-wght.woff2` | Readex Pro (SIL OFL 1.1, see `readex-pro-OFL.txt`) via @fontsource-variable/readex-pro 5.3.0, variable weight 160–700, Latin + Arabic subsets | The app's own **UI font** (not the card), with matching Latin and Arabic designs | https://cdn.jsdelivr.net/npm/@fontsource-variable/readex-pro@5.3.0/files/readex-pro-latin-wght-normal.woff2 (and `-arabic-`) |
| `arimo-font.js`, `arimo-regular.woff2`, `arimo-bold.woff2` | Arimo 1.33 Regular + Bold (SIL OFL 1.1, see `arimo-OFL.txt`), subset to Latin-1, Latin Extended-A, Romanian and Vietnamese | The card's Latin font: embedded in the PDF (`arimo-font.js`) and drawn by the preview (`.woff2`), both built from the same subset | https://github.com/googlefonts/Arimo/tree/22e482e2ce0ebb2e6f88ba49d029d60f558d5d67/fonts/ttf |

`amiri-font.js` is a jsPDF font module: `Amiri-Regular.ttf` was subset (Latin + Arabic
+ presentation forms) with `fonttools`, base64-encoded, and wrapped so it registers
itself as the `Amiri` font on every jsPDF document. To regenerate: subset the TTF, then
base64-encode it into the same wrapper.

`amiri-regular.woff2` is the browser-side counterpart, loaded via `@font-face` in
`style.css` so the preview card matches the PDF. The preview feeds it the same
pre-shaped letter forms jsPDF prints (`jsPDF.API.processArabic`), so its Arabic looks
exactly like the PDF's. A `unicode-range` limits it to Arabic script.

`arimo-font.js` is a jsPDF font module like `amiri-font.js`: it registers `Arimo`
(normal + bold) on every jsPDF document, so the PDF embeds the font instead of naming
jsPDF's built-in Helvetica. That matters twice: a print shop's file check flags fonts
that aren't embedded, and the built-in Helvetica only covers Windows-1252, so names
like "Łukasz" or "Şahin" printed as garbage. Arimo has Helvetica's character widths,
so the card lays out as it did before. `arimo-*.woff2` are the same subset for the
preview card (via `@font-face` in `style.css`). Characters outside the subset (and
outside Amiri) can't print; the app lists them in a warning (`missingGlyphs()`).

To rebuild all three from the source TTFs, with
[fontTools](https://github.com/fonttools/fonttools) (plus `brotli` for `.woff2`), run
this for `Regular` and `Bold`, once without and once with `--flavor=woff2`:

```
pyftsubset Arimo-Regular.ttf --output-file=subset/Arimo-Regular.ttf \
  --unicodes="U+0000-00FF,U+0100-017F,U+0131,U+0152-0153,U+01A0-01A1,U+01AF-01B0,U+0218-021B,U+02BB-02BC,U+02C6,U+02DA,U+02DC,U+0300-0301,U+0303-0304,U+0308-0309,U+0323,U+0329,U+1EA0-1EF9,U+2000-206F,U+20AB,U+20AC,U+20B9,U+20BA,U+20BD,U+2116,U+2122,U+2191,U+2193,U+2212,U+2215,U+FEFF,U+FFFD" \
  --layout-features='' --no-hinting --drop-tables+=GSUB,GPOS,GDEF,kern,DSIG \
  --name-IDs='*' --notdef-outline
```

Then base64-encode the two `.ttf` files into `arimo-font.js` (same wrapper). Without
hinting each `.ttf` is about 36 KB; printing doesn't need hinting.

`readex-pro-*.woff2` style the app around the card — labels, inputs, buttons — in both
languages. They are never used on the card itself, so they can't affect the PDF match.
The OFL text ships alongside as `readex-pro-OFL.txt`, as the license requires.

**Arabic shaping is handled by jsPDF itself** (it shapes contextual forms and applies
right-to-left ordering inside `doc.text`), so we just pass raw Arabic — no separate
shaper library is needed.

## Rules for this folder
- **Pin a specific version** when vendoring; record the version and source URL here.
- Don't edit the library files by hand — re-download to upgrade and note the new version.
- Prefer the unminified build where it's small enough to keep things readable while learning.
