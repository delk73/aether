# Wallpaper Rendering Architecture

Last updated: 2026-11-20

## Scope

This note captures the source selection, preview rendering, export, blur, crop,
eyedropper, and palette-extraction invariants for future wallpaper experiments.
Detailed implementation remains in the source files listed below.

## Source-of-truth paths

- Original wallpaper: `theme.svelte.ts` `wallpaperPath`
- Desktop-only blurred variant: `theme.svelte.ts` `blurredWallpaperPath`
- Full-resolution image cache: `imagecache.svelte.ts`
- Live canvas filters: `canvas-filters.ts`
- Editor UI: `wallpaper-editor/WallpaperEditor.svelte`
- Filter controls and presets: `wallpaper-editor/FilterControls.svelte`
- Crop overlay: `wallpaper-editor/CropOverlay.svelte`
- Hero preview and eyedropper: `editor/WallpaperHero.svelte`
- Blur variants: `internal/wallpaper/blur.go`
- Export: `App.SaveDataURLToFile`

## Data flow

1. A wallpaper path is selected and stored by `theme.svelte.ts`.
2. `WallpaperHero` distinguishes the extraction source from the desktop display
   source: extraction uses the original path, while `blurredWallpaperPath` is
   shown and applied for a blurred desktop variant.
3. The editor loads the original path through `loadFullImage` and caches its
   full-resolution data URL. It does not edit the blurred variant.
4. `canvas-filters.ts` derives preview and export dimensions from the original
   image. `renderToCanvas` uses a downscaled preview and writes directly to the
   bound canvas. `applyFilters` renders full resolution unless `maxSize` is set.
5. `WallpaperEditor` exports a JPEG data URL through `SaveDataURLToFile`,
   replaces the original wallpaper path with the filtered cache file, and caches
   that file for immediate preview.
6. The filtered path becomes the palette pipeline source. The editor must not
   silently retain the old original path after Apply.
7. Blur variants are cache-addressed separately and do not modify the palette
   extraction source.

## Rendering invariants

- Browser CSS filters are not used for image processing; canvas rendering keeps
  behavior consistent across supported WebKitGTK versions.
- The preview renderer and export renderer share `renderInternal`. Spatial
  effects scale by preview/output dimensions.
- Crop values are normalized source percentages.
- Crop mode renders the original image; an active filter canvas must not shift
  or resize the source underneath the crop overlay.
- Transforms use a temporary working canvas. No-transform renders write directly
  to the destination canvas for live-preview efficiency.
- `applyStrength` scales active filter values before rendering.
- Export returns `image/jpeg` at quality 0.92 and saves under
  `~/.cache/aether/filtered`; PNG data URLs retain PNG handling.
- Full-resolution export requires a loaded, ready source image.
- Blur cache writes are atomic and never modify the source file.

## Eyedropper invariants

- Sampling uses an in-image canvas, not `window.EyeDropper`.
- `mapEventToSource` accounts for object-cover, object-contain, letterboxing,
  and browser zoom.
- Full-resolution cache entries are same-origin data URLs, avoiding CORS-tainted
  canvas reads.

## Undo and history

Filter changes form an independent 50-state undo chain for one editor session,
with debounced 400 ms snapshots. Palette undo and editor shortcut handling remain
separate.

## Experiment protocol

For a wallpaper rendering experiment:

1. Establish a representative source image and repeatable expected result.
2. Change one isolated filter or control path first.
3. Add focused Vitest/jsdom coverage for deterministic calculations or state.
4. Run `npm run check` and `npm test`.
5. In `wails dev`, manually verify preview, original comparison, undo/redo,
   crop coordinates, full-resolution Apply, re-extraction, blur behavior, and
   both light and dark UI.
6. Update this file if the experiment establishes a durable invariant.

## Known verification gaps

- Preview/export pixel parity is not covered by a dedicated automated suite.
- Filter formulas and canvas edge behavior have limited jsdom coverage.
- Native WebKitGTK behavior requires manual verification under `wails dev`.
