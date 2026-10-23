# Tofu gallery copy

This repository is Tofu's copy of **storytold/photocraft** (licence: MIT OR Apache-2.0, at your option), listed in Tofu's app gallery as **PhotoCraft**.

- `main` mirrors the upstream repository and is never committed to by Tofu.
- `tofu` carries Tofu's own changes; tags named `pin/<date>` mark the versions the gallery points to.
- A visitor who chooses this app gets a private snapshot of the pinned version in their own Tofu account. Their copy does not follow upstream updates.

## What the app is

PhotoCraft is an open-source image editor in the style of Photoshop: layers, masks, adjustment layers, layer styles, type, vectors, brushes and real PSD files. Upstream calls it early alpha. It is written in Rust, and upstream also publishes a browser version: the same code compiled to WebAssembly.

The browser version runs entirely on the visitor's device. Open uses the browser's file picker (or drag and drop), Save and Export download files, and preferences stay in the browser's localStorage. There is no server, no account and no document storage: unsaved work is lost when the page is closed or reloaded. It needs a browser with WebGPU or WebGL2; over HTTPS it uses WebGPU when the browser has it.

## How Tofu runs it

| Setting | Value |
|---|---|
| Framework preset | `static` |
| Root folder | `site` |
| Install command | none |
| Build command | none: `site/` is already built (see below) |
| Output directory | `site` itself, served as it is |
| Environment variables | none |
| Database | none |

Tofu cannot build Rust or WebAssembly, so `site/` holds the finished browser build, made from the upstream commit below with upstream's own packaging script:

- `index.html` and `photocraft-web-boot-<hash>.js` / `photocraft-web-boot-<hash>_bg.wasm`: the page and the small module that starts the app.
- `app/photocraft-web-<hash>.part-aa`, `app/parts.txt`, `app/photocraft-web.js`: the app's WebAssembly (25,959,125 bytes) gzipped into one part of 11,848,986 bytes, the list of parts, and the app's loader. Upstream ships the app this way so that no file comes near the 25 MiB per-file limit of static hosts.
- `_headers`, `.htaccess`, `HOSTING.md`: upstream's sample server settings and its hosting notes (`packaging/web/README.md`). A host that does not read them is unaffected.
- `README.md`, `LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE`, `ATTRIBUTION.md`, `licenses/`: upstream's readme, licences and notices, and the licences of the third-party material compiled into the app.

Every URL in the page is relative, so the site works at any address. The app also asks for `fonts/manifest.txt` (extra fonts a host may add); the 404 it gets is expected and changes nothing.

### How `site/` was built

On 2026-10-23, on macOS (arm64), from upstream commit `ff4738810134d7f7e68b49f2e4f231b686282ff3`, with Rust 1.98.1 (stable) and its `wasm32-unknown-unknown` target, and Trunk 0.21.14 (the version upstream's CI and releases use). Trunk fetched wasm-bindgen 0.2.129 (the version in `Cargo.lock`) and wasm-opt (binaryen `version_123`).

```sh
export PHOTOCRAFT_BUILD_SHA=ff4738810134d7f7e68b49f2e4f231b686282ff3 PHOTOCRAFT_BUILD_DATE=2026-10-23
(cd apps/photocraft-web && trunk build --release --locked)
(cd apps/photocraft-web-boot && trunk build --release --locked)
packaging/web/package.sh --skip-build
rsync -a target/web-package/photocraft-web-0.14.0/ site/
cp NOTICE ATTRIBUTION.md site/
mkdir site/licenses
cp assets/fonts/OFL-*.txt assets/icons/LICENSE-*.txt assets/dict/LICENSE-*.txt \
  crates/ui-egui/src/i18n/LICENSE-translations.txt crates/algo/data/LICENSE-libfacedetection.txt site/licenses/
```

These are the steps of upstream's CI job "web build and wasm size gate": `packaging/web/package.sh` packages the two apps that Trunk builds with the `wasm-release` profile, the `heif` feature and `wasm-opt -Oz`. The two `PHOTOCRAFT_BUILD_*` values only label the About dialog with the upstream commit and the build date. Like upstream's own web build, it does not embed the optional craft-fonts fonts. To move the pin, merge the new upstream commit into `tofu`, run the same commands and replace `site/`.

## What Tofu changed and why

- Added `site/`, the finished browser build described above, because Tofu's hosting cannot run the Rust and WebAssembly toolchain. No upstream source file was changed.
- Added `NOTICE`, `ATTRIBUTION.md` and `licenses/` to `site/`, so the notices and licences of everything compiled into the app travel with the site (upstream's packaging script copies only the readme and the two licence files). They are the files upstream's `Dockerfile` copies into its web image, plus libfacedetection's BSD-3-Clause licence.
- Rewrote this file. Its first version said the licence was Apache-2.0 alone and that this copy shipped a web build while the pin held only the Rust sources.

## Upstream

- Repository: https://github.com/storytold/photocraft
- Commit: `ff4738810134d7f7e68b49f2e4f231b686282ff3` ("Release 0.14.0", 2026-10-22), PhotoCraft 0.14.0

## Licence and caveats

- PhotoCraft's code and its own assets are licensed under the MIT License or the Apache License 2.0, at your option (`LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE`). These files are unchanged, at the repository root and in `site/`. Neither licence is copyleft, so no source offer is needed; the full source is in this repository.
- Third-party material compiled into the app: the Inter and JetBrains Mono fonts (SIL Open Font License 1.1), Lucide icons (ISC; those derived from Feather also MIT), one toolbar icon adapted from the Noun Project (CC BY 3.0, credited in its licence file), the SCOWL English word list (SCOWL licence) and libfacedetection's face-detection weights (BSD-3-Clause). Their licence texts are in `site/licenses/`; `ATTRIBUTION.md` lists every file.
- Trademarks: the ArtCraft name, wordmark and logos in `docs/brand/` belong to the ArtCraft Team and are not covered by the open-source licence (`docs/brand/LICENSE-brand.txt`). The site does not include those files; the app's welcome screen links to PhotoCraft's website, GitHub, Discord and ArtCraft. Do not present this copy as an ArtCraft product or as endorsed by the ArtCraft Team. Anyone who publishes a modified version must remove the ArtCraft marks and must not call it ArtCraft.

## Verification

2026-10-23 (UTC), before tagging `pin/2026-10-23.1`:

1. Build: the commands above finished without errors. `package.sh` reported the app's wasm at 25,959,125 bytes, shipped as 11,848,986 bytes of gzip in 1 part, and "largest file in the site: 11848986 bytes (limit 25165824)".
2. Serve: a copy of `site/` made with the paths Tofu's import drops left out (`rsync -a --exclude node_modules --exclude dist --exclude build --exclude target --exclude .cache`; the copy was identical to `site/`) was served with `python3 -I -m http.server 60460 --bind 127.0.0.1 --directory <copy>`. `curl` got HTTP 200 for `/`, `index.html`, both boot files, `app/parts.txt`, `app/photocraft-web.js` and the `.part-aa` file (`application/wasm` for the boot `.wasm`, `application/octet-stream` for the part).
3. Browser (Chromium-based, WebGPU available, against 127.0.0.1 only): the app started about 2 seconds after the page loaded. A 320 x 200 PNG dropped onto the page opened as a document; Image > Adjustments > Invert inverted its colours; File > Save produced `tofu-check.psd` (229,098 bytes, starting with the PSD signature `8BPS`) and the status bar said "Saved tofu-check.psd". The download was caught in the page and never written to disk. No console errors. Every request returned 200 except the optional `fonts/manifest.txt` (404, expected). Two console warnings from eframe say the web backend ignores the window-title and theme commands.
