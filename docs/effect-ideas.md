# Effect ideas

Founding three (from the original idea):

1. **Head explode** - head region bursts into particles/debris, body keeps talking. Trigger candidate: mouth-open blendshape held, or a hotkey.
2. **Eyes pop out** - cartoon-style eye sprites launch off the eye landmarks on springs, then snap back. Trigger candidate: eye-wide blendshape spike.
3. **Dazed birds** - tiny birds orbit the head using the head pose matrix, with a wobble. Trigger candidate: head shake, or manual.

Candidates (brainstorm, not commitments):

- Spiral/hypno eyes
- Anime sweat drop and waterfall tears
- Steam from ears
- Heart eyes
- Face melt
- Pixelate/glitch head
- Screen shake "earthquake"
- Question marks or exclamation marks over the head
- Speech-bubble captions from live transcription (a11y angle too)
- Background swap to a generated scene (Lane B crossover)

Design rules:

- Every effect must hold the real-time budget in `docs/architecture.md`.
- Effects should read in a small video tile: big shapes, high contrast, motion over detail.
- Trigger by expression where possible; hotkey fallback always.
