# livemoji

Live emojis for video calls: cartoon-style effects on your live camera feed, live in the call. The founding examples are your head exploding, your eyes popping out like a cartoon character, and a dazed look with birds circling your head. Discord chats are the starting context. Those three are requested examples, not the full list.

Status: docs-only scaffold (plan, setup path, sourced research). No code exists yet.

## The real-time rule

Your constraint: anything we build toward has to work in real time. What "real time" should mean in numbers is not validated yet. Proposed engineering targets to measure against on your hardware (targets, not requirements or measured results):

- 30 fps floor, 60 fps goal, end to end
- Under ~100 ms added camera-to-call latency
- An effect that can't hold the measured budget ships behind a flag or not at all

Settling these numbers on your actual machine is part of the first desktop session (see SETUP.md and docs/architecture.md).

## Proposed route: virtual camera

In the sources we checked (Nov 2026) we found no official Discord API for third-party in-call video effects. Existing third-party effects tools (SplitCam, OBS-based apps) instead ship a virtual camera:

```
webcam -> livemoji effect pipeline -> virtual camera device -> Discord
```

You'd pick the virtual camera in Discord's Voice & Video settings. In principle the same output should also work in other call apps that accept a camera source (Meet, Zoom, Teams), but that's untested and per-app quirks are possible.

## Two lanes, no settled product decision

Your ask is video generation. The lanes below are about how to get there, not a decision to swap generation for something simpler.

**Lane A: CV-based baseline (proposed prototype).** Google's MediaPipe Face Landmarker advertises 478 3D face landmarks plus 52 expression blendshapes in real time, on-device and GPU-accelerated. That's the vendor's claim, not yet measured on your machine. The proposal: use it to build the effect framework, triggers and virtual-camera plumbing quickly, and to set a measured real-time baseline that any generative approach has to beat on quality. The three founding effects have plausible landmark-anchored implementations here; whether they'd look good enough is an open question, not a settled one.

**Lane B: generative video (the stated direction; feasibility open).** Real-time video diffusion exists in published results, but every measured number we found was produced on datacenter-class GPUs with specific resolutions and settings (details and links in docs/architecture.md). We found no published measurements on consumer laptops, and one system's own ~300 ms glass-to-glass figure may already read as lag in a call. Whether generation is viable for you, locally or via a hosted endpoint, is an open validation task with a concrete test plan in docs/architecture.md.

## Repo layout

- `SETUP.md` - docs-only desktop setup path, from fresh clone to one measured effect in a Discord call
- `docs/architecture.md` - proposed pipeline, unvalidated latency budget, Lane B research with sources, and the feasibility test plan
- `docs/effect-ideas.md` - the effect backlog: the three requested examples plus candidates

## Naming

"livemoji" is a working title. Renaming a repo is cheap; pick the real name once the product shape settles.
