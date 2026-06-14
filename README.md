# TernInk Movin

TernInk Movin is an early drawing and animation prototype focused on parametric vector strokes, animated brushes, onion skinning, frame holds, and video-reference rotoscoping.

## Install And Run

Use a clean install from source. Do not ship `node_modules`, `.vite`, `dist`, or TypeScript build info files as part of the source package.

```bash
npm ci
npm run dev
```

## Validate

```bash
npm ci
npm run build
npm test
npm run e2e:install
npm run e2e
npm run e2e:ci
npm run package:source
npm run validate:source-zip
```

`npm run e2e:install` downloads the Playwright Chromium browser. In locked-down or offline networks, install the browser on a connected machine/cache before running `npm run e2e`.

## Current Features

- Canvas navigation with pan, zoom, and rotation.
- Pressure-aware vector strokes with deterministic brush snapshots per stroke.
- Animated brush families including clean ink, wiggle, pulse, neon, flow, particle, smoke, sparkle, dash, ribbon, glitch, liquid, fur, and spray.
- Layer opacity, blend modes, and onion-skin fades applied through the renderer.
- Timeline frame holds, blank frames, duplicate frame, duplicate previous and advance, drawing navigation, and draw-on-ones/twos.
- Selection tool with rendered-stroke hit testing, drag threshold, move, arrow-key nudge, Shift+nudge, delete, and Ctrl+D duplicate.
- Eraser with adjustable size and Cut Stroke / Whole Stroke modes.
- Video and image reference layers with metadata, fit/offset/trim controls, missing-media status, relink, and match-project-duration for video.
- `.tnkm` project save/load as a ZIP package with `manifest.json`, `project.json`, brush data, thumbnail manifest, and embedded media assets.
- IndexedDB autosave with recovery prompt; localStorage is used only for small migration/status data.
- Storage panel with project info, package-size estimate, media asset usage, Missing Media Manager, locally stored recent projects, and safe asset cleanup that preserves current/autosave/recent project media.
- Current-frame PNG export, PNG sequence ZIP export, and desktop PNG sequence folder export through the Tauri file adapter.
- Resizable workspace panels with saved layout.
- App-shell layout with TopBar, ToolBar, CanvasStage, InspectorPanel, StatusBar, and contextual ToolOptionsBar.
- Inspector tabs for Brush, Layers, Media, Onion, and Storage, with workspace presets for Default, Rotoscopy, Brush Design, and Minimal.
- In-app toasts, confirm dialogs, modal shell, and export progress dialog instead of browser alert/confirm flows.
- Media timeline clips with thumbnails when available, offset dragging, trim handles, markers/key poses, and loop range.
- Video ghost controls and preview rendering for previous/next reference frames, opacity, offset, and tint.
- Frame-aware video timeline thumbnails with IndexedDB cache and clip-thumbnail fallback.
- Selection transform polish with scale/rotate handles, Shift rotate snapping, flip horizontal/vertical, and Escape cancel.
- Desktop file workflow state for current project path, file name, dirty state, Save, and Save As when running inside Tauri. Save As chooses the target path before packaging media so cancelling does not waste time building a large `.tnkm`.
- Drag and drop for `.tnkm`/legacy project files and supported image/video media.

## Shortcut Notes

- `N` creates a blank frame at the current frame without advancing.
- `Shift+N` creates a blank frame and advances by the active hold duration.
- `D` duplicates the current frame without advancing.
- `Shift+D` duplicates the previous drawing into the current frame and advances.
- Arrow keys step frames unless strokes are selected. With selected strokes, arrows nudge; `Shift` nudges by 10 px.
- `Ctrl+D` duplicates selected strokes.
- `Escape` cancels an active selection transform; otherwise it clears the stroke selection.

## Export

Current-frame export downloads one PNG. PNG sequence export renders deterministically using `frameIndex / fps`, packages frames as `frame_0001.png`, `frame_0002.png`, and so on, and downloads a single ZIP file in the web build.

Export supports 100%, 50%, and 25% scale. Video/image layers are rendered through cloned media elements for export so preview playback does not compete for the same `HTMLVideoElement.currentTime`.

Large PNG sequences still use an in-browser ZIP step, so TernInk Movin warns before very large exports. Use a custom range or lower scale for long shots.

When running inside Tauri, the export dialog can write a PNG sequence directly to a chosen folder. That mode creates a timestamped subfolder, writes `manifest.json` with an `incomplete` status first, writes each PNG frame one by one through the desktop file adapter, and then marks the manifest `complete`. Cancelled or failed exports leave the manifest marked `cancelled` or `failed` so partial output is obvious. ZIP export remains available.

MP4/WebM export is intentionally not part of this stage; frame-accurate PNG sequence export comes first.

## Project Storage

The primary project format is `.tnkm`, a ZIP package:

```text
manifest.json
project.json
assets/
brushes/custom_brushes.json
thumbnails/manifest.json
```

Media layers store `assetId` instead of depending on temporary `objectUrl` values. At load time, TernInk Movin recreates runtime object URLs from packaged or IndexedDB assets. Legacy `.inkmotion`, `.rotobrush`, and `.json` projects can still be opened, but they may require relinking media.

When opening `.tnkm`, the loader materializes only assets referenced by `project.json`. Extra packaged assets are ignored and surfaced as a warning in the app UI.

Recent projects are stored locally in IndexedDB as convenience snapshots in the web build. In Tauri, recent records also keep the `.tnkm` file path and reopen from disk when possible; if the file is missing or unreadable, the Storage panel marks that recent item as missing. The snapshot remains a fallback convenience record, not a replacement for saving a `.tnkm` package.

## E2E And Desktop Prep

`tests/e2e` contains Playwright browser QA. It stays separate from fast unit tests:

```bash
npm run e2e:install
npm run e2e
npm run e2e:ui
npm run e2e:ci
```

`src-tauri` contains the Tauri desktop alpha scaffold with the TernInk Movin window name and offline Vite build wiring:

```bash
npm run tauri:dev
npm run tauri:build
```

Desktop alpha open/save uses Tauri dialog and filesystem plugins when the frontend detects the Tauri runtime. The native File/Edit/View menu emits frontend commands for New, Open, Save, Save As, Export, Undo, Redo, and Reset Workspace. Save writes directly to the current `.tnkm` path after an opened/saved desktop file exists; Save As always opens a native save dialog. The browser build keeps using HTML file inputs and downloads.

Desktop beta bundle configuration is enabled with generated PNG/ICO/ICNS icons and `.tnkm` file association metadata (`application/x-ternink-movin`). The project pins Rust through `rust-toolchain.toml`; use `cargo --version` from the repo root to verify the active toolchain. `src-tauri/Cargo.lock` is committed after desktop dependency resolution.

Use `docs/desktop-qa.md` for the manual desktop alpha checklist covering Tauri launch/build, native menu commands, drag and drop, missing media, recent projects, and desktop folder export behavior. See `docs/desktop-validation-report.md` for the last executed desktop build/dev validation and any pending manual QA.

CI is defined in `.github/workflows/ci.yml` for clean install, build, unit tests, source packaging, and source ZIP validation. Playwright E2E remains available through the local/manual scripts above because browser downloads can be environment-dependent.

Source delivery should be the direct `ternink_movin_source.zip` generated by `npm run package:source`. Do not wrap it in an outer folder or include `node_modules`, `dist`, `.vite`, `test-results`, TypeScript build info, or another ZIP inside it.
