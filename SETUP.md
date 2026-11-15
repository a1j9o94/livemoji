# Setup

Goal of this pass: from a fresh clone to one real-time effect visible in a Discord call.

## 0. Fill in your targets

Edit these once known; the virtual camera step and packaging choices depend on them.

- Desktop OS: TODO (Windows / macOS / Linux)
- GPU: TODO (Lane A runs fine on CPU or integrated graphics; Lane B wants serious NVIDIA hardware)
- Discord desktop installed

## 1. Virtual camera (one time, per OS)

- Windows: install OBS Studio from https://obsproject.com and start its Virtual Camera once to register the driver.
- macOS: same. OBS ships a virtual camera on macOS.
- Linux: load the `v4l2loopback` kernel module instead.

Check it: Discord -> Settings -> Voice & Video -> camera dropdown. The virtual camera should appear in the list.

## 2. Effect development environment

Lane A prototype target: a local web app. Browsers give you `getUserMedia`, GPU-accelerated MediaPipe, and WebGL with zero native build pain, which makes effect iteration fast.

- Node.js LTS (22+)
- Scaffold: `npm create vite@latest app -- --template vanilla-ts`
- `npm i @mediapipe/tasks-vision`
- Face landmarker model: https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/1/face_landmarker.task

Per-frame loop:

```
getUserMedia -> FaceLandmarker.detectForVideo -> draw effect layers on a canvas -> canvas.captureStream()
```

Run the landmarker with `delegate: 'GPU'`, `runningMode: 'VIDEO'`, `outputFaceBlendshapes: true`. On the web these calls block the main thread, so put detection in a Web Worker for a stable 60 fps.

## 3. Bridge the canvas to the virtual camera

TODO: pick one, depends on the OS answer above.

- Simplest: OBS browser/window source pointed at the local effect page, OBS Virtual Camera as output.
- Cleaner later: a small native bridge (`pyvirtualcam` on Windows with the OBS driver, `v4l2loopback` on Linux) fed by a local WebSocket or shared-memory frame source.

## 4. First milestone checklist

- [ ] Virtual camera shows up in Discord's camera dropdown
- [ ] Face landmarks tracking at 30+ fps in the prototype
- [ ] One effect pinned to the head (dazed birds is the friendliest first build)
- [ ] Full loop live in a real Discord call (second account or a friend)
