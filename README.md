# Gesture Draw — Air Gesture Note-Taking

A webcam-based whiteboard where you draw in the air with your hand. Raw ink appears on the canvas exactly as you trace it — no mouse or touchscreen drawing required.

Built for the [Devpost OpenAI Build Week](https://openai.devpost.com/) hackathon. The product keeps **raw air-drawn ink** as the source of truth; a future GPT-5.6 "Clean up this note" action will interpret finished notes without replacing the original drawing.

## Why this exists

Quick visual note-taking should work anywhere a webcam is available: open the board, gesture in the air, and keep what you drew. The interaction is intentionally physical — pinch to put the pen down, move your hand, release to lift — so the demo stays reliable under hackathon conditions.

## Current functionality

### Drawing modes

| Mode | How to draw |
| --- | --- |
| **Thumb–index (default)** | Pinch thumb and index finger together to start a stroke; separate them to finish. Pinch over header controls to click buttons. |
| **Keyboard index** | Click **Thumb-Index Mode** to switch. Hold **Space** or **Enter** and move your index fingertip over the board to draw; release the key to finish the stroke. |

### Core features

- Browser webcam capture with single-hand tracking via [MediaPipe Hands](https://developers.google.com/mediapipe/solutions/vision/hand_landmarker).
- Live 21-landmark hand overlay in a compact camera preview (top-right).
- Full-bleed responsive whiteboard canvas that fills the available viewport.
- Pinch distance normalized by hand size, with separate pen-down / pen-up thresholds (hysteresis) to reduce flicker.
- One Euro Filter + rolling average + adaptive cursor gain for smooth, responsive fingertip tracking.
- Camera safe margin (inner 74% of the frame maps to the full board) so corners stay reachable without leaving the camera view.
- Soft release threshold stops accepting new ink as fingers begin separating, reducing end-of-stroke hooks.
- Incremental live ink rendering during strokes (full canvas rebuild only when a stroke finishes).
- Adjustable stroke thickness (1–10 px), four preset ink colors plus a custom color wheel picker, eraser mode, undo, and clear.
- Canvas zoom (50%–225%) via CSS transform — toolbar and camera stay fixed.
- Save board snapshots as note thumbnails in the footer filmstrip.
- On-screen framing guidance when a hand is too close, too far, near the camera edge, or when a busy background (e.g. your face) makes tracking jittery.
- ROI hand tracking: once a hand is found, later frames crop to the hand region so background clutter interferes less.
- Frame preprocessing (contrast/saturation boost) and temporal landmark smoothing for steadier tracking.
- Mobile detection popup recommending desktop/laptop use; landscape mode supported on phones.

### Not active in the current build

- **Text mode / local OCR** — Tesseract.js is bundled but disabled (`textModeActive = false`). Handwriting is kept as raw ink.
- **GPT-5.6 clean-up** — planned as a separate post-drawing action via a minimal backend endpoint (see below).

## Run locally

The app is a single static HTML file. MediaPipe and Tesseract load from CDNs on first visit.

1. Open a terminal in this project folder.
2. Start a local server:

   ```powershell
   python -m http.server 8000
   ```

3. Open [http://localhost:8000](http://localhost:8000).
4. Click **Start Camera** and allow webcam access.
5. Keep one hand in view and draw using either mode above.

**Requirements:** A current Chromium-based browser, a working webcam, and internet access on first load (CDN scripts + MediaPipe model assets).

## Quick test guide (for judges)

1. Load the hosted or local URL in Chrome or Edge.
2. Grant camera permission when prompted.
3. **Pinch mode:** Pinch thumb + index, move hand, release — ink should appear smoothly without freezing.
4. **Keyboard mode:** Click **Thumb-Index Mode** in the header to switch, hold Space, move index finger, release Space.
5. Try **Undo**, change color/thickness, toggle **Eraser**, and **Save to notes** to confirm toolbar controls work.
6. Pinch over a header button (e.g. **Clear**) to verify pinch-to-click on controls.

## Gesture tuning (current values)

These are defined in `index.html` and tuned for live webcam testing:

| Setting | Value | Purpose |
| --- | ---: | --- |
| Pen-down threshold | `0.24` | Normalized thumb–index distance to start a stroke. |
| Pen-up threshold | `0.30` | Ends stroke after fingers separate. |
| Soft release | `0.27` | Stops adding points as fingers begin opening. |
| Pen-down confirmation | 1 frame | Fast pen-down after a firm pinch. |
| Pen-up confirmation | 1 frame | Fast pen-up after release. |
| Camera safe margin | `13%` | Inner frame maps to full board edges. |
| Cursor deadband | `0.8 px` | Ignores sub-pixel jitter when nearly still. |
| Min stroke spacing | `1.5 px` | Minimum distance between recorded points. |

## Planned GPT-5.6 integration

GPT-5.6 will power a **"Clean up this note"** workflow, separate from the real-time drawing loop:

1. User finishes a note and taps **Clean up this note**.
2. The canvas image is sent to a minimal server endpoint (API key stays off the client).
3. GPT-5.6 returns a handwritten-text transcription, a short title, and a one-line summary.
4. Results display alongside the unchanged original ink.

This keeps the low-latency gesture experience independent from AI latency.

## How Codex collaboration shaped the project

Codex has been the primary implementation partner for the core frontend, split into testable milestones:

- **Project framing:** Preserved raw air-drawn ink as the core product decision.
- **Tracking milestone:** Webcam + MediaPipe Hands with live landmark overlay.
- **Gesture milestone:** Normalized pinch detection, hysteresis, and confirmation frames.
- **Interaction refinement:** 2D pinch metric, object-fit crop correction, safe margin mapping, One Euro Filter smoothing.
- **Design iteration:** Light-mode responsive layout, full-bleed board, compact camera preview.
- **Drawing performance:** Incremental live segments during strokes to avoid full-canvas rebuilds every frame (fixes UI freeze in keyboard-index mode).

The human collaborator drives product calls: minimal UI, natural pinch feel, live hardware testing, and mode selection for different drawing preferences.

## Development log

| Milestone | Change |
| --- | --- |
| 1. Webcam tracking | MediaPipe Hands, landmark overlay, tracking status HUD. |
| 2. Real air ink | Index fingertip mapped to canvas; pinch = pen down/up. |
| 3. Reliable pinch | Hand-size normalization, hysteresis, confirmation frames. |
| 4. Edge mapping | Corrected for `object-fit: cover` crop in camera preview. |
| 5. Visual refinement | Light mode, viewport-responsive board, compact preview. |
| 6. Jitter and guidance | Smoothing filters and on-screen hand-framing hints. |
| 7. Precision | Safe camera margin, adaptive cursor gain, deadband. |
| 8. Stroke fidelity | Geometry-preserving rendering, soft release, straight-line assist. |
| 9. Keyboard index mode | Hold Space/Enter + index finger as an alternative to pinch drawing. |
| 10. Live ink performance | Incremental segment rendering during active strokes; full redraw only on stroke finish. |
| 11. Tracking + color | ROI crop tracking, frame preprocessing, landmark smoothing for busy backgrounds; custom color wheel picker. |
| 12. Color wheel reliability | Moved the picker into viewport-aware positioning, synchronized its marker/lightness with active ink, and prevented toolbar clipping. |

## Next steps

1. Add canvas export for the GPT-5.6 clean-up action.
2. Create a minimal secure backend endpoint and integrate GPT-5.6 vision.
3. Persist saved notes (IndexedDB or similar) with title/summary in a gallery.
4. Further tune pinch thresholds and stroke smoothing from live testing.
