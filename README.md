# Lottie Forge

Turn images, GIFs, videos, SVGs and existing Lottie files into optimised, **fully vector** Lottie animations: for the web, for apps, and for microcontrollers through [LottieMCU](https://github.com/jobitjoseph/LottieMCU).

Lottie Forge runs entirely in your browser, works offline, and never uploads your files anywhere.

---

## Features

- **Many inputs:** PNG, JPG, WebP, GIF, MP4, WebM, SVG, Lottie `.json` and `.lottie`. Several images at once become a frame sequence.
- **Pure vector output:** shape layers only (paths, fills, strokes, transforms), no embedded bitmaps.
- **Two tracing engines**
  - **Classic:** smooth curves, good for illustrations and artwork.
  - **Line art:** straight lines and sharp corners, made for icons and logos. It can also turn thin lines into real stroked paths.
- **Clean colours:** a k-means palette keeps small accent colours. Anti-aliased edge pixels snap to their neighbouring colours instead of becoming sliver shapes. Any colour can be merged or deleted with one click.
- **Frame sequences** (GIF, video, image sets)
  - **Flipbook:** every frame becomes its own vector layer. Exact, and the default.
  - **Smart motion:** shapes are tracked across frames and stored as keyframes. Moves, rotations and scales become a few transform keys, and changing shapes morph. Usually much smaller.
- **Existing Lottie files**
  - Image layers are traced into vector shapes, keeping their position, timing and parenting.
  - Hidden layers, tiny paths and redundant keyframes are removed.
  - Curves can optionally be refitted.
  - Flipbook-style files can be rebuilt as smart motion.
- **Motion presets** for still images and SVGs: spin, pulse, float, bounce, wiggle, shake, blink, pop-in and fade-in. Apply one to the whole artwork or to individual parts.
- **Output size:** longest side 240–800 px, or any custom width × height (presets for common TFT displays). The artwork is fitted and centred.
- **Exact-size preview** at actual pixels, with zoom, a timeline scrubber, and transparent, white or black backgrounds.
- **Live stats:** JSON and gzipped size (with change against your previous settings), vertices, keyframes, layers, paths, and the Lottie features used.
- **Exports:** `.json`, `.lottie` (dotLottie), and for LottieMCU a C header `.h` or raw binary `.lmc`.
- **Batch convert:** several files with the same settings, saved as one `.zip`.

## Getting started

1. Download `lottie-forge.html` from this repository (or from the Releases page).
2. Open it in a current browser. Chrome or Edge is recommended.
3. Drop a file anywhere on the page, or click **Open file**.

### Host it on GitHub Pages

To serve it from GitHub Pages:

1. Rename `lottie-forge.html` to `index.html`.
2. In the repository, go to **Settings → Pages** and enable Pages.

The tool then runs at `https://<your-user>.github.io/<repo>/`.

## How to use

The left panel has four tabs. Hover over any control to see what it does.

| Tab | What it does |
|---|---|
| **01 Source** | Frame sampling for GIF, video and Lottie rebuilds: sample rate, frame limit, start and end trim, merge similar frames, or use every frame at the source's own rate. |
| **02 Vectorise** | Profile, output size, tracing engine, colour count, curve tolerance, minimum path size, pre-blur, background removal, edge cleaning, lines as strokes, and the colour swatches (click one to merge or delete it). |
| **03 Motion** | Still images and SVGs: presets for the whole artwork or for a part (pick parts in the **Parts** view). Frame sequences: flipbook or smart motion. Lottie files: rebuild as smart motion. |
| **04 Export** | Frame rate, decimals, name stripping, Lottie optimisation options, downloads, the LottieMCU report and batch convert. |

### Profiles

| Profile | Use it for |
|---|---|
| Balanced | Web and app animations |
| LottieMCU lean | Small screens on ESP32-class boards: 240 px, 4 colours, fewer frames, names stripped |
| High detail | Large, detailed artwork |

### Tips for small files

- **Fewer colours.** Usually the biggest single saving.
- **Raise curve tolerance** for fewer points per shape.
- **Raise "Drop paths under"** to remove specks.
- **Line art** for icons and logos, with **lines as strokes** for outline drawings.
- **Smart motion** instead of flipbook when shapes move rather than change completely.
- **Lower the sample rate** or set a frame limit for GIFs and videos.
- **Decimals 0 or 1** and **strip names** for the final export.

## LottieMCU

Lottie Forge writes the same `LMC1` binary as LottieMCU's `tools/lottie2mcu.py` (format 1.1.0). The output is byte-identical to the Python converter on the rlottie sample animations and on files exported from this tool, so no Python step is needed.

1. Choose **LottieMCU lean**, or set a custom size matching your display (for example 320 × 240).
2. In **04 Export**, set the array name and click **.h**.
3. Use it in your sketch:

```cpp
#include <TFT_eSPI.h>
#include <LottieMCU.h>
#include "my_anim.h"

TFT_eSPI tft;
LottieMCU lottie;

void flush(int16_t x, int16_t y, uint16_t w, uint16_t h, const uint16_t *px, void *) {
  tft.pushImage(x, y, w, h, (uint16_t *)px);
}

void setup() {
  tft.init();
  tft.setSwapBytes(true);
  lottie.begin(my_anim, tft.width(), tft.height());
  lottie.setFlush(flush);
}

void loop() {
  lottie.update(millis());
}
```

Use **.lmc** instead to load animations from LittleFS or SD at runtime (see LottieMCU's `ESP32_LittleFS` example).

The LottieMCU panel shows:

- the binary size,
- the largest path, so you can check `Config::maxPathVerts` (default 256, small profile 32),
- the number of draw items, against `Config::maxItems`,
- any warnings from the converter.

For complex animations, run LottieMCU's **Benchmark** sketch at your real display size to check the remaining limits.

## Browser support

Chrome and Edge support every feature. Other current browsers handle images, SVG, Lottie and video, with two caveats:

- **Animated GIF frames** need the browser's `ImageDecoder` API. Without it, only the first frame is used and the tool says so.
- **Video** decoding depends on the formats the browser supports. WebM and H.264 MP4 work in Chrome and Edge.

Tracing runs in a background worker so the page stays responsive.

## Privacy

Everything runs locally in your browser. No file is uploaded, no network request is made, and the tool works with the network switched off.

## Limitations

- Raster tracing approximates the source. Photographs become stylised, posterised vector art.
- SVG import replaces gradients with their average colour and skips text. Convert text to outlines in your editor first.
- Smart motion works best on flat, cartoon or icon-style footage with clear shapes.
- Lottie image layers can be vectorised only when the image is embedded in the `.json` or included in the `.lottie`.

## Third-party software

Bundled inside `lottie-forge.html`:

- [lottie-web](https://github.com/airbnb/lottie-web) 5.12.2: MIT License, © 2015 Bodymovin. The full licence text is included in the file.
- [imagetracerjs](https://github.com/jankovicsandras/imagetracerjs) 1.2.6: public domain (Unlicense).

## Author

**Jobit Joseph**: [github.com/jobitjoseph](https://github.com/jobitjoseph)

## Copyright

© 2026 Jobit Joseph. All rights reserved.
