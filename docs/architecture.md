# Architecture

## Proposed pipeline

```
[webcam] --frames--> [frame processor] --frames--> [virtual camera] --> Discord
                          |
              +-----------+-----------+
              |                       |
        Lane A: CV effects      Lane B: generative
        (MediaPipe + WebGL)     (video diffusion)
```

The frame processor is the only interface that matters: one frame in, one frame out, with a per-effect config. Keeping that seam is what lets a generative backend slot in later without rewriting capture, output or triggers. This is a proposed design, not validated code.

## Proposed latency budget (unvalidated)

At 30 fps a frame lasts 33 ms. Rough proposed per-frame budget:

| Stage | Proposed budget |
| --- | --- |
| Camera capture | ~5 ms |
| Face landmark detection | 5-10 ms (GPU delegate, per vendor docs) |
| Effect render | ~8 ms |
| Virtual camera handoff | 5-10 ms |
| **Total** | **< 33 ms** |

These are planning guesses. The first desktop session should replace them with measurements (SETUP.md step 3) and record the results here.

## Lane A: classic CV (proposed baseline)

MediaPipe Face Landmarker (`@mediapipe/tasks-vision` on web), per Google's docs:

- 478 3D face landmarks per face
- 52 blendshape scores for expressions (candidate effect triggers: wide eyes, open mouth)
- Facial transformation matrices for anchoring 3D effects to head pose
- Advertised as real time with the GPU delegate; not yet measured on target hardware

Sources:
- https://developers.google.com/edge/mediapipe/solutions/vision/face_landmarker
- Web worker pattern: https://developers.google.com/edge/mediapipe/solutions/vision/face_landmarker/web_js

## Lane B: generative video (stated direction; feasibility open)

Published real-time video-diffusion numbers, each under the benchmark conditions its source states:

| System | Claimed performance | Hardware/conditions | Source |
| --- | --- | --- | --- |
| StreamDiffusionV2 | 58 FPS (14B model), 64 FPS (1.3B), first frame < 0.5 s | 4x H100 | https://streamdiffusionv2.github.io/ |
| fal.ai StreamDiffusionV2 deploy | 512x512 at 16 fps | 1x H100, hosted | https://fal.ai/docs/examples/video-generation/deploy-realtime-video-restyle |
| LiveGen 1.0 | 720p at 25-30 fps, ~300 ms camera-to-screen | 1x RTX PRO 6000 | https://occipitallabs.com/blog/livegen/ |
| Krea Realtime 14B | ~11 fps, 40GB+ VRAM | 1x B200 | https://deepwiki.com/krea-ai/realtime-video |

What this does and doesn't say:

- Every measured real-time result we found is on datacenter-class GPUs. We found no published measurements on consumer laptops, so local generative feasibility on your hardware is unknown, not ruled out.
- The one glass-to-glass figure published (LiveGen, ~300 ms) may read as lag in a live call; acceptable added latency for a call is itself an untested assumption.
- A hosted endpoint (fal-style) adds network round-trip latency on top of generation; untested for calls.
- A hybrid (generative keyframes plus CV interpolation) is a speculative idea, unexplored and unsourced.

## Feasibility test plan (desktop continuation)

1. Build the Lane A prototype and measure the full loop (SETUP.md step 3). This yields the baseline fps/latency any generative approach must beat on quality to be worth it.
2. If a hosted real-time endpoint looks promising, wire it behind the same frame-processor interface and measure end-to-end latency including the network. Record results here.
3. Only then decide what Lane B is: local GPU (if hardware allows), hosted, hybrid, or deferred. No decision is baked into this scaffold.

## Virtual camera route

In sources checked Nov 2026 we found no official Discord API for third-party in-call video effects. Existing third-party effects tools ship a virtual camera that call apps select as their input:

- pyvirtualcam + OBS Virtual Camera pattern: https://github.com/Mahdi-hasan-shuvo/virtualcam-studio
- Linux v4l2loopback daemon approach: https://github.com/mexicantexan/studiocast
- SplitCam (virtual camera driver, advertises Discord among 60+ apps): https://splitcam.com/virtual-camera

Untested on your machine and OS. Compatibility with apps other than Discord (Meet, Zoom, Teams) is plausible from how virtual cameras work but not verified here.
