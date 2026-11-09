# Betaflight boot logo generator

A single-page tool for making a custom 288×72 boot logo (splash screen) for Betaflight analog OSDs. You can use text in any language, your own images, or both. The output is a BMP or PNG that Betaflight Configurator accepts as a custom logo.

Everything happens in `index.html`. It needs no build step, server or libraries, and uploaded images never leave the visitor's browser. The only outside dependency is Google Fonts.

## Using it

1. Open `index.html` in a browser, or host it anywhere (see below).
2. Type one or two lines of text and/or upload an image, then adjust them until the preview looks right.
3. Click **Download BMP**.
4. In Betaflight Configurator, open the **OSD** tab and then **Font Manager**. Pick your font preset, click **Select custom logo image**, choose the BMP, check that both green ticks appear, and click **Upload Font**. The logo is stored inside the font, so it's uploaded together with it.

Configurator only accepts pixels that are exactly black `#000000`, white `#FFFFFF` or green `#00FF00`. Green becomes transparent on the OSD. Configurator ignores the alpha channel, so a truly transparent pixel would be read as black. That's why the tool writes green and never alpha.

This works for analog (MAX7456) OSDs. Digital systems like HDZero, Walksnail and DJI/WTFOS use the goggles' own font instead.

## Showing the logo when you arm

By default the logo only appears at boot. Since Betaflight 4.2 it can also appear when you arm. The page has a helper that builds these commands for you, or you can type them into the CLI tab yourself:

```
set osd_logo_on_arming = ON
set osd_logo_on_arming_duration = 20
save
```

| Setting | Values | Default |
| --- | --- | --- |
| `osd_logo_on_arming` | `OFF`, `ON` (every arm), `FIRST_ARMING` (only the first arm after power-up) | `OFF` |
| `osd_logo_on_arming_duration` | 5–50, in tenths of a second (0.5 s to 5 s) | 5 (0.5 s) |

The logo is drawn in the same place as at boot, and the "ARMED" message is still shown below it.

Newer firmware also has `osd_arming_logo` (0–3). It picks which font page the arming logo is drawn from, so it only matters on HD systems whose fonts have several pages, where you can show a different logo on arming than at boot.

## How it works

1. **Text layer.** Text is drawn onto a hidden 288×72 canvas in the chosen font, with a stroke for the outline. Right-to-left scripts (Hebrew, Arabic) are drawn with `ctx.direction = 'rtl'`, so the browser handles letter order and Arabic letter joining. Each line shrinks until it fits the width.
2. **Image layer.** The uploaded image is scaled down to at most 600 px for processing. The most common border color counts as the background. **Touching the edges** flood-fills from the border and removes connected pixels within the tolerance, and **Everywhere** removes every pixel within the tolerance. The cleaned image is then drawn at the chosen scale and offset.
3. **Conversion.** Every pixel becomes clear, white or black. Pixels under 50% alpha become clear. The rest are split by brightness against the threshold, with optional Floyd–Steinberg dithering. A one-pixel black outline can be added around white areas.
4. **Merge.** Text pixels win over image pixels.
5. **Export.** Clear pixels become `#00FF00`. The BMP is written byte by byte as 24-bit with no alpha channel. A PNG export is included too.

## Hosting

- **As its own page:** upload `index.html`, for example as `/tools/bf-logo/index.html`. GitHub Pages works too: enable Pages for this repository and serve from the root of the default branch.
- **Inside an existing page:** embed it with an iframe:

  ```html
  <iframe src="/tools/bf-logo/" style="width:100%;height:1600px;border:0"></iframe>
  ```

  Alternatively, copy the `<style>`, `<main>` and `<script>` blocks into your page. If you do that, rename the CSS classes so they don't clash with your site's styles.
- **Privacy:** loading Google Fonts from Google's servers sends visitors' IP addresses to Google. A German court ruled against a site for this in 2022, so EU-facing sites often self-host the fonts instead. Download them (google-webfonts-helper is handy for that), add your own `@font-face` rules, and change `loadGoogleFont()` to just call `document.fonts.load(...)`. All the listed fonts are under the SIL Open Font License, so self-hosting is allowed.

## Customizing

- **Add a font:** add a line to `FONTS`, for example `['Exo 2', 'Exo+2:wght@400;700', 'latin cyrillic']`. Use the Google Fonts URL name, and include `:wght@400;700` only if the font actually has both weights. If you request a weight that doesn't exist, the font fails to load.
- **Add a language:** add a line to `LANGS` and tag at least one font with the same script key.
- **HD or color logos:** change `W`/`H`, for example to 576×144 for HDZero's 24×36 tiles. Then skip the 3-color classification and export the layer canvas directly as a transparent PNG.
