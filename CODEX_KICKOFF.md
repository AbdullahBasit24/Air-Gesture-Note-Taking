# Airscript — Project Context for Codex

## What this is
A web app where a person writes/draws in the air with a webcam, using a pinch gesture
(index finger tip touching thumb tip) as the "pen down" signal — like a Kinect-style
gesture interface, but for handwriting instead of navigation. This is being built for
OpenAI Build Week (Devpost hackathon). Codex must be used for the majority of core
development, and GPT-5.6 must be meaningfully integrated (not decorative) — both are
hard eligibility requirements for this hackathon, not optional nice-to-haves.

## Core interaction model (this is the whole product — keep it simple)
1. Webcam feed runs through MediaPipe Hands in the browser, giving 21 hand landmarks
   per frame in real time.
2. Compute normalized distance between thumb tip (landmark 4) and index fingertip
   (landmark 8), normalized by hand size (e.g. divide by wrist-to-middle-MCP distance)
   so the pinch threshold works regardless of how close the hand is to the camera.
3. When pinch distance drops below a "close" threshold: pen is down, start/continue
   a stroke, drawing at the index fingertip's mapped canvas position.
4. When pinch distance rises above a separate, slightly higher "open" threshold
   (hysteresis gap, not one shared threshold — prevents flicker at the boundary):
   pen is up, stroke ends.
5. Draw the raw traced path directly on a canvas, exactly as moved. No handwriting
   recognition, no letter/glyph replacement, no font substitution. What you draw is
   what appears — like a mouse cursor you control with a pinched hand instead of a
   physical mouse.
6. Smooth the input in two places to fight camera/tracking jitter:
   - Smooth the raw landmark position frame-to-frame (EMA or one-euro filter)
     before using it as a draw point.
   - Optionally smooth the resulting stroke path afterward (e.g. `perfect-freehand`
     or a Catmull-Rom spline) so lines don't look jagged.

## Where GPT-5.6 comes in (required, must be real — not incidental)
A "Clean up this note" action, separate from the real-time drawing loop:
- User finishes writing/drawing on the board and taps a button.
- The canvas (or a cropped region of it) is sent as an image to GPT-5.6 (vision-
  capable call via the OpenAI API).
- Prompt GPT-5.6 to: transcribe any handwritten/drawn text it can read, generate a
  short title for the note, and produce a one-line summary.
- Display the transcription/title/summary alongside the original ink — do not
  replace or redraw the user's ink itself. This keeps the raw-ink drawing feature
  and the AI feature cleanly separated, so a failure/slowness in the GPT-5.6 call
  never blocks or degrades the core drawing experience.
- This is the feature to visibly demo in the required demo video, since judges are
  specifically evaluating whether GPT-5.6 is meaningfully used vs. bolted on.

## Explicit non-goals (do not build these — keep scope tight for hackathon time)
- No handwriting recognition that redraws/replaces strokes with "corrected" glyphs.
- No mouse-based drawing input — gesture/webcam input only.
- No per-user handwriting style modeling.
- No multi-hand support unless trivially easy — single hand is enough.
- No account system / auth / multi-user backend unless time allows; local-only
  note storage (e.g. IndexedDB or simple in-memory state) is fine for the demo.

## Suggested tech stack
- Frontend: plain JS or React, HTML5 Canvas for rendering strokes.
- Hand tracking: MediaPipe Hands (`@mediapipe/hands` or MediaPipe Tasks Vision),
  runs client-side in-browser — no server round trip needed for tracking itself,
  which matters for latency during live gesture drawing.
- Stroke smoothing: `perfect-freehand` (npm) or a hand-rolled Catmull-Rom spline.
- AI layer: OpenAI API, GPT-5.6, vision input, called only for the "clean up" action.
- Backend: only as much as needed to keep the OpenAI API key off the client —
  a minimal server endpoint (Node/Express or a simple serverless function) that
  accepts the canvas image and forwards the GPT-5.6 request.

## Build order (do these as separate, testable milestones — prompt Codex once per milestone, not all at once)
1. Webcam feed + MediaPipe Hands running, landmark dots drawn as an overlay on
   the video. Confirm hand tracking works before touching drawing logic.
2. Pinch detection only: on-screen "PINCHING" / "NOT PINCHING" label driven by
   normalized thumb-index distance. Tune threshold, add hysteresis.
3. Raw stroke drawing: pinch = pen down, draw between consecutive fingertip
   points, release = pen up. This alone is a demoable "finger paint in the air" app.
4. Jitter smoothing: EMA/one-euro filter on landmark position, spline/perfect-freehand
   smoothing on the rendered stroke.
5. "Clean up this note" trigger: button that crops/exports the canvas as an image.
6. GPT-5.6 integration: send the image to a minimal backend endpoint, call the
   OpenAI API with a vision-capable GPT-5.6 request, return transcription + title
   + summary, render them next to the original ink.
7. Notes list/gallery: let a user save a finished board as a note and see past
   notes (thumbnail + title), even if storage is just local/in-memory for the demo.

## README requirements (per hackathon rules — maintain this throughout, not just at the end)
The submission's README must document, and this should be updated incrementally
after each milestone rather than written once at the end:
- Setup/run instructions (npm install steps, any API key/env var setup, how to
  grant webcam permission).
- Any sample data needed to test (not much needed here beyond a working webcam).
- Clear guidance for a judge to test the running project without rebuilding it
  from scratch — ideally a hosted demo link or a very short local setup path.
- Where Codex accelerated the workflow and where key product/engineering/design
  decisions were made (e.g. "chose pinch-based pen-down/pen-up over recognition-based
  input to keep the demo reliable under hackathon time pressure").
- How GPT-5.6 is integrated and what it actually does in the product.

## Reminders for whoever's driving Codex
- Use one Codex prompt per milestone above, not one giant "build the whole app"
  prompt — smaller, scoped asks produce cleaner, more debuggable output.
- Ask Codex to explain what it generated before moving to the next milestone,
  especially for the MediaPipe/normalization/threshold logic, since debugging
  step 4 requires understanding step 3.
- Run `/feedback` in the Codex thread where the majority of core functionality
  gets built, and keep the resulting Session ID — it's required at submission.
- Demo video (≤3 min, public YouTube, must have voiceover) needs to show: what
  the app does, a real working demo of the gesture-drawing, and a specific
  (not vague) explanation of how Codex was used and how/why GPT-5.6 is integrated.
