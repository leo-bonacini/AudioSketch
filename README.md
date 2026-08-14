<img src="assets/logo.svg" width="56" height="56" alt="AudioSketch logo" align="left">

# AudioSketch

Mark regions & events, tag classes, export JSON.

<br clear="left">

AudioSketch is a single-file, static HTML tool for labelling audio as training input for ANN models — sound event detection, speech/diarization segments, audio classification. It has no backend, no build step, no API keys, and no CDN dependencies: everything runs client-side (Web Audio API for decoding/playback, Canvas for the waveform), fully offline.

## Features

- **Import audio**: drag & drop or file picker, any number of clips, nothing uploaded anywhere
- **Two annotation shapes**: region (a time range, for events/segments) and marker (a single timestamp, for instantaneous events)
- **Whole-clip tags**: for classification-style datasets that don't need time-aligned regions, just labels per clip
- **Class manager**: add/remove classes, each auto-assigned a color; pick the active class before drawing
- **Playback**: play/pause (button or Space), click the waveform to seek, drag to scrub, arrow keys to nudge ±1s (±5s with Shift), auto-scrolling playhead
- **Loop & preview**: loop the selected region during playback to check a label by ear, or hit Preview to jump straight to a region's start (or just before a marker) and play
- **Edit after drawing**: with the select tool, drag a region's body to move it, drag either edge to trim it, drag a marker to retime it, right-click any shape to relabel or delete it
- **Zoom, all by mouse**: scroll zooms toward the cursor, fit/100%/±zoom buttons, or the dedicated zoom tool (click to zoom in, click again to flip to zoom out, Alt/right-click for a one-off zoom out); Shift+scroll or the scrollbar pans
- **Import / export JSON**: export writes every clip's tags + annotations (seconds and normalized 0–1 positions); import restores annotations onto matching filenames, or reconstructs clips entirely if the export embedded audio data
- **Export Dataset (ZIP)**: every clip renamed to a content hash, one label file per clip, and a manifest — see below

## Usage

1. **Import Audio** (or drag & drop) to load one or more clips.
2. In the **Classes** panel, add a class name (e.g. `dog_bark`, `speech`, `cough`) — it becomes the active class.
3. Pick a tool: **region** (drag a time range — returns to select after each one) or **marker** (click to drop a point — stays active so you can tag several events in a row; Esc to stop).
4. Click a shape (on the waveform or in the **Annotations** list) to select it — re-tag its class in the sidebar, drag to move/trim it, or delete it. Or **right-click any shape** to relabel/delete from a popup menu.
5. Use **Preview** or the **loop** button (top transport bar) to listen back to a labelled region and confirm it's right.
6. For classification-only labelling, skip drawing and just add **Clip tags** at the bottom of the sidebar.
7. Click **Export Labels JSON** to download everything (check **embed audio** first for a fully self-contained file), or **Export Dataset (ZIP)** for a training-ready package (see below).
8. **Import Labels JSON** re-applies a previous export onto already-loaded clips with matching filenames, or reconstructs clips if the export embedded audio data.

### Keyboard shortcuts

`V` select · `R` region · `M` marker · `Z` zoom tool · `Space` play/pause · `←`/`→` seek ±1s (Shift = ±5s) · `Delete`/`Backspace` remove selected annotation · `Esc` cancel current draw / deselect / close menu · `+`/`-` zoom in/out · `0` fit to window

## Output format

```json
{
  "format": "audiosketch-v1",
  "exported_at": "2026-08-14T12:00:00.000Z",
  "classes": [{ "name": "dog_bark", "color": "#5aa9e6" }],
  "clips": [
    {
      "filename": "backyard.wav",
      "duration": 42.3,
      "sample_rate": 44100,
      "channels": 2,
      "tags": ["outdoor"],
      "annotations": [
        {
          "class": "dog_bark",
          "color": "#5aa9e6",
          "shape_type": "region",
          "start": 3.12, "end": 4.87,
          "start_normalized": 0.0738, "end_normalized": 0.1151
        }
      ]
    }
  ]
}
```

- Times are given both in seconds and normalized 0–1 (position ÷ clip duration).
- Markers carry `time` / `time_normalized` instead of `start`/`end`.
- `audio_data` (a data URL) is only present per-clip when **embed audio** was checked at export time.

## Structured dataset export

**Export Dataset (ZIP)** packages the session for a training pipeline, not for re-importing into AudioSketch:

```
audiosketch-dataset-2026-08-14.zip
├── manifest.json
├── audio/
│   ├── 3f9a0c12e7b4.wav
│   └── 8b21d4f0a655.mp3
└── labels/
    ├── 3f9a0c12e7b4.json
    └── 8b21d4f0a655.json
```

- Every clip is renamed to a 12-character content hash — deterministic from its own bytes, carrying no trace of the original filename/recorder/folder.
- Each clip gets a matching `labels/<hash>.json` (same seconds + normalized fields as above) — one label file per sample.
- `manifest.json` indexes every clip/label pair plus the class list.
- The **"keep original filenames in manifest"** checkbox (on by default) adds `original_filename` for traceability; uncheck it to fully anonymize the archive.
- The ZIP is hand-built in vanilla JS ("store"/no-compression — audio is already compressed) — no library, no network call.

## Notes and limitations

- Nothing is saved automatically: export before closing the tab or reloading the page.
- The waveform is precomputed once at import into a fixed-resolution peak bitmap (not full per-sample data), then scaled to whatever zoom you're at — zooming in gives you far more precise mouse control over boundary placement, but not literally more waveform detail than that base resolution. This keeps memory and import time bounded even for long recordings.
- Very long recordings get less zoom headroom than short clips (the live canvas width is capped for browser/memory safety) — still zoomable, just proportionally less deep.
- Editing is geometric only (move/trim/retime/delete) — there's no undo history yet.
- No third-party format export (e.g. Praat TextGrid, ELAN) yet; the native JSON carries both seconds and normalized positions so it converts easily with a short script.

## Stack

Vanilla JavaScript + Web Audio API (decoding only) + HTML5 `<audio>` (playback) + Canvas (waveform). No frameworks, no build step, no network calls.

## License

MIT
