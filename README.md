# livemoji

Live emojis for video calls. Cartoon effects rendered on your real camera feed, in real time: make your head explode, pop your eyes out like a cartoon character, or sit there dazed with birds circling your head, live in the call. Discord chats are the starting context. Those three are the founding effects, not the full list.

Status: fresh scaffold. This repo holds the plan, the setup path, and the grounded research. There is no working app yet.

## The one hard rule

Everything built toward this has to work in real time. Targets:

- 30 fps minimum end to end, 60 fps as the goal
- Under ~100 ms added latency from camera sensor to the call
- An effect that misses the budget ships behind a flag or not at all

## How it works

Discord (like Zoom, Meet and Teams) has no official API for plugging custom effects into a call. The standard route is a virtual camera:

```
webcam -> livemoji effect pipeline (per frame) -> virtual camera device -> Discord
```

You pick the virtual camera in Discord's Voice & Video settings, and everyone in the call sees the processed feed. No Discord plugin, no per-app integration, and the same output works in Meet, Zoom and Teams for free.

## Two build lanes

**Lane A: classic CV effects (build first).** Google's MediaPipe Face Landmarker detects 478 3D face landmarks plus 52 expression blendshape scores in real time, on-device and GPU-accelerated. Anchor sprites, shaders and particle effects to those landmarks. All three founding effects are fully achievable in this lane on an ordinary laptop: eyes pop (scale and displace eye sprites off the eye landmarks), dazed birds (particles orbiting the head pose matrix), head explode (segment the head region and blow it into particles while the body stays).

**Lane B: generative video effects (research track).** Real-time video diffusion exists but is datacenter-class hardware today. Measured numbers and links are in `docs/architecture.md`. The pipeline exposes a pluggable frame-processor interface so a generative backend (local GPU, or hosted on something like fal) can slot in behind the same virtual camera output later without a redesign.

## Repo layout

- `SETUP.md` - dev environment and the path from clone to first effect live in Discord
- `docs/architecture.md` - pipeline design, latency budget, Lane B research with sources
- `docs/effect-ideas.md` - the effect backlog: founding three plus candidates

## Naming

"livemoji" is a working title. Renaming a repo is cheap; pick the real name once the product shape settles.
