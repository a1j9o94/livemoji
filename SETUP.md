# Setup

This repo currently contains docs only. Nothing below runs existing code; the steps scaffold and measure new code on your desktop. Goal of the first desktop session: from zero to one effect visible in a Discord call, with real performance numbers from your machine.

## 0. Fill in your targets

Edit these once known; the virtual camera step and the Lane B options depend on them.

- Desktop OS: TODO (Windows / macOS / Linux)
- GPU: TODO (decides which generative options, if any, are testable locally)
- Discord desktop installed

## 1. Virtual camera (one time, per OS)

- Windows: install OBS Studio from https://obsproject.com and start its Virtual Camera once to register the driver.
- macOS: same; OBS ships a virtual camera on macOS.
- Linux: load the `v4l2loopback` kernel module instead.

Check it: Discord -> Settings -> Voice & Video -> camera dropdown. The virtual camera should appear in the list (untested on your machine; this is the documented behavior of these drivers).

## 2. Scaffold the prototype (Lane A)

Lane A first because it's the fastest way to exercise the full loop (capture -> track -> effect -> virtual camera) and produce baseline numbers. Proposed stack: a local web app, since browsers give you `getUserMedia`, GPU-accelerated MediaPipe and WebGL with no native build work.

- Node.js LTS (22+)
- Scaffold: `npm create vite@latest app -- --template vanilla-ts`
- `npm i @mediapipe/tasks-vision`
- Face landmarker model: https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/1/face_landmarker.task

Per-frame loop to build:

```
getUserMedia -> FaceLandmarker.detectForVideo -> draw effect layers on a canvas -> canvas.captureStream()
```

Per Google's docs, run the landmarker with `delegate: 'GPU'`, `runningMode: 'VIDEO'`, `outputFaceBlendshapes: true`, and put detection in a Web Worker because the web calls block the main thread. Treat all of that as the documented design; verify the actual frame cost on your hardware in step 3.

## 3. Measure real-time performance (the step that settles feasibility)

Instrument the loop before polishing anything:

- Log per-frame timings for capture, landmark detection, effect render, and virtual-camera handoff
- Compute achieved fps and added latency over a few minutes, not a best-case second
- Compare against the proposed budget in docs/architecture.md

These numbers, on your hardware, are what validate or kill each lane. Don't trust vendor numbers for the decision, including the ones cited in this repo.

## 4. Bridge the canvas to the virtual camera

TODO: pick one, depending on the OS answer above.

- Simplest: OBS browser/window source pointed at the local effect page, with OBS Virtual Camera as output.
- Cleaner later: a small native bridge (`pyvirtualcam` on Windows with the OBS driver, `v4l2loopback` on Linux) fed by a local frame source.

## 5. First session checklist

- [ ] Virtual camera visible in Discord's camera dropdown
- [ ] Landmark tracking running, with measured per-frame cost
- [ ] One requested effect (dazed birds is the friendliest first build) pinned to the head
- [ ] Measured end-to-end fps and latency recorded in docs/architecture.md
- [ ] Full loop live in a real Discord call (second account or a friend)
