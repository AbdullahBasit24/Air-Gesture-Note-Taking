# Gesture Draw: Air Gesture Note-Taking

A webcam-powered whiteboard and gesture arcade. Draw in the air with your index finger, save notes as PNG snapshots, then take a break with small games controlled by the same hand tracking.

Built for the [Devpost OpenAI Build Week](https://openai.devpost.com/) hackathon. The app is fully client-side: it does not require an API key, backend, Vercel, or an AI service.

## What Gesture Draw aims to fix

Traditional note-taking needs a keyboard, a stylus, or a surface to write on. Gesture Draw explores a more natural alternative: writing in mid-air with a webcam. It focuses on reducing common air-writing frustrations such as shaky strokes, accidental pinch triggers, release hooks, cursor-to-hand misalignment, lost tracking near camera edges, and the lack of an easy way to keep or share a quick sketch.

## Drawing

| Mode | How it works |
| --- | --- |
| **Thumb-index (default)** | Pinch thumb and index to put the pen down; separate them to lift the pen. Pinch over controls to use them. |
| **Keyboard index** | Switch modes, hold **Space** or **Enter**, then move your index fingertip to draw. |

The board preserves raw air-drawn ink and includes:

- Undo, clear, eraser, ink colors, a custom color wheel, thickness control, and zoom.
- Live webcam preview with hand landmarks, camera status, framing guidance, and gesture hover feedback.
- Saved note thumbnails, plus one-click PNG downloads for every saved snapshot.
- Thumb-index pinch controls for drawing and selecting controls, plus keyboard-index mode for accessibility and testing.

### Writing refinements

- Hand-size-normalized pinch detection with separate down and up thresholds prevents accidental stroke flicker.
- Pinch settling, soft release, and tail trimming reduce hooks at the beginning and end of letters.
- Landmark smoothing, rolling averages, a One Euro filter, and a cursor deadband reduce jitter during slow, precise handwriting.
- Linear board mapping keeps diagonal motion predictable, while safe camera margins let users reach board edges before their hand reaches the webcam edge.
- Point spacing, curve rendering, and strict straight-line assist retain intended letter shapes without turning raw writing into a font.
- Live segments render as the hand moves; the full canvas redraw happens only when a stroke ends.

## Games Hub

Open **Games Hub** in the toolbar to play local, browser-only games. High scores are stored in `localStorage` on the current device and can be reset from the hub.

| Game | Gesture controls |
| --- | --- |
| **Tic-Tac-Toe** | Aim at an empty square and pinch to place X. |
| **Table Tennis** | Move the index fingertip left/right to guide the paddle. |
| **Endless Runner** | Move left/right to choose a lane; pinch to jump. |
| **Fruit Slice** | Sweep the index fingertip through fruit. |
| **Snake** | Swipe the index fingertip up, down, left, or right to steer. Collect red pixels and avoid walls and your own trail. |

The UI is tuned around a full-bleed board, compact top controls, a live webcam preview with landmark overlay, gesture hover feedback, framing guidance, and responsive desktop/mobile layouts.

## Run locally

The project is a single static HTML app. MediaPipe and Tesseract load from CDNs on the first visit.

1. Open a terminal in this project folder.
2. Start a local server:

   ```powershell
   python -m http.server 8000
   ```

3. Open [http://localhost:8000](http://localhost:8000).
4. Click **Start Camera** and grant webcam permission.

Requirements: a recent Chromium-based browser, a webcam, and internet access for the CDN scripts and MediaPipe hand-tracking assets.

## Quick test guide

1. Start the camera and confirm the 21-point hand overlay follows one hand.
2. Pinch, draw a few letters, and release. Try both drawing modes.
3. Test undo, eraser, color, thickness, zoom, saved notes, PNG downloads, and pinch-to-click controls.
4. Open Games Hub and verify the gesture controls for each game; Snake responds to clear fingertip swipes.

## Codex collaboration

Codex helped implement the gesture drawing pipeline, smoothing and stroke-fidelity improvements, responsive UI refinements, and the local gesture-controlled games. The product intentionally keeps raw ink as the source of truth and keeps the experience self-contained in the browser.
