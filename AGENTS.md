# Agent Guide: Wallpaper Rendering

Use repo-local documentation before implementation:

- `CLAUDE.md` for architecture and conventions
- `docs/wallpaper-editor.md` for user-facing behavior
- `docs/wallpaper-rendering.md` for rendering invariants
- `docs/color-extraction.md` for the downstream palette pipeline

## Wallpaper experiment workflow

For wallpaper edit, rendering, crop, preview, eyedropper, or blur tasks:

1. Identify whether the change affects source selection, desktop display/blur,
   live canvas preview, full-resolution export, palette extraction, or native
   desktop behavior.
2. Preserve separation between the original palette source and the optional
   blurred desktop variant.
3. Reuse `renderInternal` for preview/export logic. Do not introduce a separate
   CSS or canvas filter path without a measured compatibility reason.
4. Keep crop coordinates normalized in source space and transform behavior
   deterministic.
5. Add focused Vitest/jsdom tests for pure calculations or state transitions.
   Use `wails dev` for native canvas, image loading, eyedropper, and desktop
   wallpaper verification.
6. Run `npm run check` and `npm test` for frontend changes. Run the smallest
   relevant Go test package for backend/export or blur changes.
7. Update `docs/wallpaper-rendering.md` when an experiment establishes a
   durable invariant or verification gap.
