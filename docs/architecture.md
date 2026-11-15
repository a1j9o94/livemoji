# Architecture

## Pipeline

```
[webcam] --frames--> [frame processor] --frames--> [virtual camera] --> Discord
                          |
              +-----------+-----------+
              |                       |
        Lane A: CV effects      Lane B: generative
        (MediaPipe + WebGL)     (video diffusion)
```

The frame processor is the only interface that matters: one frame in, one frame out, with a per-effect config. Everything else (capture, virtual camera, UI) is plumbing that stays fixed when backends change. That interface is what keeps Lane B from forcing a rewrite later.

## Latency budget

At 30 fps a frame lasts 33 ms. Rough budget per frame:

| Stage | Budget |
| --- | --- |
| Camera capture | ~5 ms |
| Face landmark detection | 5-10 ms (GPU delegate) |
| Effect render | ~8 ms |
| Virtual camera handoff | 5-10 ms |
| **Total** | **< 33 ms** |

Measure all four from day one. A slow effect that drops frames feels worse than a simple one that holds 60.

## Lane A: classic CV (ship first)

MediaPipe Face Landmarker (`@mediapipe/tasks-vision` on web, native delegates elsewhere):

- 478 3D face landmarks per face
- 52 blendshape scores for expressions (useful as effect triggers: wide eyes trigger the pop, open mouth triggers the explosion)
- Facial transformation matrices for anchoring 3D effects to head pose
- Real time with the GPU delegate

Source: https://developers.google.com/edge/mediapipe/solutions/vision/face_landmarker

Web worker pattern for the render loop: https://developers.google.com/edge/mediapipe/solutions/vision/face_landmarker/web_js

## Lane B: generative video (research track)

Honest state of the art, measured claims from the owning sources:

| System | Claim | Hardware | Source |
| --- | --- | --- | --- |
| StreamDiffusionV2 | 58 FPS (14B), 64 FPS (1.3B), first frame < 0.5 s | 4x H100 | https://streamdiffusionv2.github.io/ |
| fal.ai StreamDiffusionV2 deploy | 512x512 at 16 fps | 1x H100 | https://fal.ai/docs/examples/video-generation/deploy-realtime-video-restyle |
| LiveGen 1.0 | 720p at 25-30 fps, ~300 ms glass-to-glass | 1x RTX PRO 6000 | https://occipitallabs.com/blog/livegen/ |
| Krea Realtime 14B | ~11 fps, 40GB+ VRAM | 1x B200 | https://deepwiki.com/krea-ai/realtime-video |

Takeaway: real-time generative video is real, but nothing here runs on a consumer laptop yet, and 300 ms of added latency is already noticeable in a call. Lane B's near-term realistic form is a hosted backend (fal-style) behind the same frame-processor interface, tried as an experiment, not the foundation. Lane A is the product for now.

## Virtual camera route

Discord has no official custom-effects API for calls. Every working third-party effects product (Snap Camera back when, SplitCam, OBS-based apps) ships a virtual camera the call app selects as its input. References:

- pyvirtualcam + OBS Virtual Camera pattern: https://github.com/Mahdi-hasan-shuvo/virtualcam-studio
- Linux v4l2loopback daemon approach: https://github.com/mexicantexan/studiocast
- SplitCam (kernel virtual camera, 60+ apps incl. Discord): https://splitcam.com/virtual-camera

This also means "works in Discord" gets you Meet, Zoom and Teams at no extra cost.
