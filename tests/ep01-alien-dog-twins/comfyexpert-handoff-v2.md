# comfyui-expert Handoff — Stage 3: Character & Location Ref Image Lock (v2 — resolution-corrected)
**Project:** Ep.1 Test — Twins/Alien/Dog | **Style:** Disney Pixar (3D animated) | **Final export target:** 16:9, 720p

## Task
Generate locked, consistent reference images for the characters and locations below, using PuLID + Flux Kontext (or your preferred consistency-locking workflow) so these refs can be reused across all subsequent shot/clip generation for this episode.

## Output Resolution (LOCKED — apply to all refs below)
- **Scene/full reference image:** 1536×864 (16:9) — 2x final 720p render res, clean integer downscale to 1280×720
- **Character face-lock crop:** additional **square 1024×1024** per character, alongside the 16:9 scene ref (for PuLID-style face-embedding consistency)
- **Locations:** 1536×864 only, no square crop needed

## Characters
*(generate: one 1536×864 scene ref + one 1024×1024 square face-lock crop, each)*

**Twin Brother 1**
- ~9 years old, Pixar-style 3D stylization, average kid build
- Differentiator: wears a **red cap/shirt accent**
- Expression range needed: curious/playful (Scene 1), shocked (Scene 3-4), panicked (Scene 5)

**Twin Brother 2**
- ~9 years old, near-identical to Twin 1 (same base facial structure — twins)
- Differentiator: wears a **blue cap/shirt accent**
- Same expression range as Twin 1

**Father**
- Adult male, tech-savvy/inventor archetype, workshop setting
- Suggested look: casual work clothes, rolled sleeves, gadget/tool in hand or nearby
- Expression range: focused (mid-project), alarmed, intrigued/determined (final beat)

**Dog**
- Puppy, **brown & white Border Collie**
- Expression/pose range: playful/trotting (Scene 1), startled/scared (Scene 2-3)
- (No face-lock crop needed — animal, skip square crop)

## Locations
*(generate: one 1536×864 scene ref each, no square crop)*

- **Suburban street:** tree-lined, quiet residential, golden-hour late-afternoon lighting
- **Home workshop/garage:** cluttered workbench, gadgets/tools, warm interior lighting contrasted with the alien craft's cool blue glow from Scene 2-3

## Output Requirements
- Per character: 1× 1536×864 scene ref + 1× 1024×1024 square face-lock crop
- Per location: 1× 1536×864 scene ref
- Consistency priority: character faces/proportions must hold up across all listed expressions, since these refs get reused for every shot in Stage 5/6

## Notes
- v2 of this handoff — same characters/locations as the first test run, corrected to the locked resolution spec (1536×864 + 1024×1024 face-lock). Compare against the first pass to confirm the face-lock crop improves consistency downstream.
