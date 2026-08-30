# LocalMark

English | [中文](README.md)

A local-only image/PDF watermarking tool — everything runs in your browser, nothing is ever uploaded, and it works fully offline.

🔗 Try it live: https://krstic-c.github.io/local-mark/

## Features

- **Zero upload**: all image/PDF processing happens in browser memory via the Canvas API — no server, no API calls
- **Offline-ready**: double-click `index.html` to use it fully offline; fonts and PDF libraries are bundled locally, no CDN dependency
- **Text watermark**: multi-line text, color/size/opacity/stroke/blend mode, one-click insert of date or date+time
- **Image/Logo watermark**: upload a local image (company logo, confidential stamp, etc.) as the watermark
- **Two layout modes**: tiled (adjustable angle and density, brick-style staggered rows to avoid gaps) or single-point placement via a 9-grid picker
- **Batch processing**: drop in multiple files, apply the same watermark settings, export everything as a ZIP
- **PDF watermarking**: applies the watermark to every page and exports a watermarked PDF
- **Presets**: save named watermark configurations for quick reuse; automatically remembers your last settings

## Usage

1. Open the [live version](https://krstic-c.github.io/local-mark/), or clone this repo and double-click `index.html`
2. Drag images (JPG/PNG) or PDF files into the left panel
3. Set your watermark text or upload a watermark image, then adjust color, size, opacity, angle, density, etc. — the preview updates live on the right
4. Click "Download Current File" to export a single file, or "Batch Export ZIP" to export everything at once
5. Save frequently used watermark setups under "Presets" for one-click reuse later

## Local development

This is a static single-page app with no build step.

```bash
git clone https://github.com/Krstic-c/local-mark.git
cd local-mark
python3 -m http.server 8000
```

Then open `http://localhost:8000` in your browser.

You can also just double-click `index.html` to use it fully offline (some browsers restrict font loading over the `file://` protocol — if styling looks off, serve it via a local static server instead).

## Project structure

```
index.html          single-file app: all HTML/CSS/JS
vendor/
  pdf-lib.min.js     bundled PDF processing library
  jszip.min.js       bundled ZIP packing library
  fonts/             bundled DM Sans / DM Mono font files
```

## Privacy

This tool never uploads, stores, or transmits your image/PDF content in any form. It only uses browser `localStorage` to remember your watermark parameter presets (color, size, etc.) — never the file content itself.
