# Air Gesture Note-Taking

A webcam-based whiteboard that lets you write or draw in the air. Pinch your thumb and index finger together to put the pen down, move your hand to create raw ink, then separate them to lift the pen. No stylus, touchscreen, or mouse drawing is required.

> The project name is intentionally still a placeholder while the product is being developed.

## Why this exists

This project is being built for the Devpost OpenAI Build Week hackathon. The core idea is to make quick visual note-taking accessible wherever a webcam is available: open a board, gesture in the air, and preserve exactly what was drawn.

The product deliberately prioritizes raw ink over handwriting replacement. A future AI clean-up action will interpret the finished note without modifying the original drawing.

## Current functionality

- Browser webcam capture with single-hand tracking via MediaPipe Hands.
- Live 21-landmark hand overlay in a small camera preview.
- Gesture-only drawing: thumb–index pinch starts a stroke; releasing the pinch ends it.
- Pinch distance is normalized by hand size, making the gesture less dependent on camera distance.
- Separate start and release thresholds (hysteresis) reduce accidental pen flicker.
- Three-frame confirmation for pen-down, so a near-pinch does not immediately start drawing.
- 2D pinch measurement to better accommodate different hand angles.
- Smoothed fingertip position and interpolated strokes to reduce camera jitter.
- Undo, clear, ink color selection, and local note-thumbnail previews.
- Responsive light-mode interface that maximizes the available whiteboard area on laptop and smaller displays.
- Live framing guidance when a hand is too close, too far, or near the camera-frame boundary.

## Run locally

The app is currently a dependency-free static frontend except for MediaPipe, which is loaded from a CDN.

1. Open a terminal in this project folder.
2. Start a local server:

   ```powershell
   python -m http.server 8000
   ```

3. Open [http://localhost:8000](http://localhost:8000).
4. Click **Start camera** and allow webcam access.
5. Keep one hand in view. Pinch thumb and index finger to draw; separate them to finish a stroke.

Use a current Chromium-based browser for the most reliable webcam and MediaPipe behavior. Internet access is required on first load for the MediaPipe CDN scripts and model assets.

## Gesture tuning notes

The current values are deliberately easy to tune from real camera testing:

| Setting | Current value | Purpose |
| --- | ---: | --- |
| Pen-down threshold | `0.22` | Requires a firm thumb–index pinch. |
| Pen-up threshold | `0.27` | Ends the stroke shortly after the fingertips separate. |
| Pen-down confirmation | 3 frames | Rejects a one-frame near-pinch. |
| Position smoothing | `0.20` | Reduces jitter before points are written. |

## Planned GPT-5.6 integration

GPT-5.6 is planned as a meaningful, separate “Clean up this note” workflow rather than part of the real-time drawing loop:

1. The user finishes a note and chooses **Clean up this note**.
2. The canvas image is sent to a minimal server endpoint, keeping the OpenAI API key off the client.
3. GPT-5.6 receives the image and returns a handwritten-text transcription, a short title, and a one-line summary.
4. The app displays that information alongside the unchanged original ink.

This separation protects the low-latency drawing experience and makes the AI contribution explicit in the demo. The UI currently includes a prototype clean-up panel; the API endpoint and live GPT-5.6 call remain upcoming work.

## How Codex collaboration shaped the project

Codex has been used as the primary implementation partner for the core frontend so far. The work has been deliberately split into testable milestones rather than attempting the entire product in one change.

- **Project framing:** Codex reviewed the hackathon brief and preserved the central product decision: raw air-drawn ink, not handwriting replacement.
- **Tracking milestone:** Codex converted a visual-only frontend prototype into a real webcam + MediaPipe Hands experience, then the collaborator tested live hand detection in-browser.
- **Gesture milestone:** Codex implemented normalized pinch detection, a distinct open/close hysteresis gap, and a short confirmation window. Thresholds were then tuned using real measured values reported during testing.
- **Interaction decisions:** Based on testing feedback, Codex changed the pinch metric from 3D to 2D landmark distance for better tolerance of hand angle, corrected camera-crop coordinate mapping so the cursor can reach board edges, and added framing/distance guidance.
- **Design iteration:** Codex changed the prototype to a light-mode responsive layout, made the canvas occupy the available viewport, and retained a small camera preview so the board stays the focus.
- **Quality checks:** After each implementation pass, Codex ran a JavaScript syntax check. Browser webcam behavior was verified collaboratively through live testing because it cannot be fully reproduced in a headless static check.

The human collaborator made the key product calls: keeping the UI minimal, prioritizing a natural physical pinch, testing each behavior on real laptop hardware, and deciding that the project name should remain undecided for now. Codex accelerated the implementation, debugging, responsive layout work, and documentation.

## Development log

| Milestone | Change and context |
| --- | --- |
| 1. Webcam tracking | Replaced simulated tracking and mouse-preview drawing with webcam permission, MediaPipe Hands, landmark connections, and tracking status. |
| 2. Real air ink | Mapped the mirrored index fingertip to a canvas and used a thumb–index pinch as pen down/up. |
| 3. Reliable pinch | Added hand-size normalization, hysteresis, and confirmation frames after early accidental drawing was observed in testing. |
| 4. Edge mapping | Corrected for `object-fit: cover` crop in the video preview so visible camera edges map to whiteboard edges. |
| 5. Visual refinement | Converted the interface to light mode, made the board viewport-responsive, and added a compact camera preview. |
| 6. Jitter and guidance | Added position/stroke smoothing and on-screen guidance for hand framing and camera distance. |
| 7. Current refinement | Enlarged the board inset and lowered the pen-up threshold to make release more immediate. Stroke thickness/appearance remains an intentional item for further tuning. |

## Next steps

1. Further tune pen-up responsiveness and stroke smoothing from live testing.
2. Refine ink rendering so smoothing does not make handwriting appear wider or heavier.
3. Add canvas export/cropping for the clean-up action.
4. Create the minimal secure backend endpoint and integrate GPT-5.6 vision.
5. Save notes persistently and show title/summary in a note gallery.
