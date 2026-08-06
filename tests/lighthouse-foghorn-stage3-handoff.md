# comfyui-expert Handoff — Stage 3: Character & Location Ref Image Lock
**Project:** TheLighthouseFoghorn_Ep01 | **Style:** Hand-drawn animation, stylized/graphic, muted/desaturated palette | **Tone:** Melancholic/bittersweet | **Final export target:** 16:9, 720p

## Task
Generate locked, consistent reference images for the characters and locations below, using PuLID + Flux Kontext (or your preferred consistency-locking workflow) so these refs can be reused across all subsequent shot/clip generation for this episode.

## Output Resolution (LOCKED — apply to all refs below)
- **Scene/full reference image:** 1536×864 (16:9) — 2x final 720p render res, clean integer downscale to 1280×720
- **Character face-lock crop:** additional square **1024×1024** per character, alongside the 16:9 scene ref (for PuLID-style face-embedding consistency)
- **Locations:** 1536×864 only, no square crop needed

## Characters
*(generate: one 1536×864 scene ref + one 1024×1024 square face-lock crop, each — except the Creature, see note)*

**The Keeper**
- 70 years old, lean and wiry build, weathered face with age spots, visible scars, broken nose
- Cloudy brown eyes, clean-shaven but perpetually grey-stubbled
- Faded earth-tone wool flannel shirt (muted rust/grey-brown), thick wool trousers, worn cracked leather boots
- Carries an old worn pocket watch
- Expression range needed: quiet/withdrawn (routine), wonder/emotional (discovering the creature — eyes wet, jaw tight), gentle resolve (final farewell)

**The Foghorn Creature**
- NOT a drawn character — this is a design/mood-reference request, not a character portrait.
- Generate as an **abstract visual-metaphor reference** for the resonance chamber interior: faint inner glow (light seeming to emanate from within brass rather than reflecting off it), dust motes caught in swirling spiral patterns, water droplets rippling on brass in concentric circles, subtle warm/cool color shift in the metal's patina
- No face-lock crop — non-humanoid, not applicable
- Generate as a **location-style scene ref** (1536×864) of "inside the resonance chamber" rather than a character portrait

**The Coast Guard Officer**
- 40s, tall, fit build
- Sharp eyes, one with a thin old scar running from eyebrow to cheekbone; clean-shaven, strong jawline
- Formal Coast Guard uniform, visibly rain-soaked (clinging fabric, softened professional edges)
- Expression range needed: professional/composed (arrival), genuine warmth breaking through (acknowledging the sound was "alive")

## Locations
*(generate: one 1536×864 scene ref each, no square crop)*

- **Lighthouse Lamp Room:** small intimate circular space, bronze foghorn bell dominating center covered in barnacle-like salt crusts and verdigris, antique brass lever/dial control panel positioned lower off to the side, rain-streaked gallery windows, peeling paint, salt stains, dust, oil stains, pale grey dawn/overcast light. Palette: dull gold, verdigris green, rust orange/red, iron grey, warm brass accents.
- **Lighthouse Keeper's Quarters:** small austere room, narrow bed with folded muted blue-grey quilt, weathered wooden desk piled with leather-bound logbooks facing a single window over the sea, simple blackened cast-iron stove with kettle, whitewashed stone/concrete walls, worn grey wood flooring, soft diffuse grey natural light, unlit oil lamp on desk. Palette: greys, creams, weathered browns, faded blue-grey accent.
- **Lighthouse Machinery Room:** vast yet claustrophobic vertical space, foghorn mechanism floor-mounted rising near the ceiling with a resonance chamber at mid-height, dense organic tangle of pulleys/gears/pipes/rope coils/chains, brass/copper/iron with heavy verdigris and rust bloom patina, dim shadowy lighting from small high windows, dust motes in light shafts, pooled water reflecting light, salt-crusted lower pipes. Palette: dull gold, verdigris green, rust red/orange, deep shadow black.

## Output Requirements
- Per character (Keeper, Officer): 1× 1536×864 scene ref + 1× 1024×1024 square face-lock crop
- Per location (Lamp Room, Quarters, Machinery Room): 1× 1536×864 scene ref
- Creature: 1× 1536×864 abstract mood/metaphor reference (resonance chamber interior), no face-lock crop
- Consistency priority: Keeper and Officer faces/proportions must hold up across all listed expressions, since these refs get reused for every shot in Stage 5/6

## Source
Generated from the confirmed `bios.md` at
`~/hermes-cine-projects/TheLighthouseFoghorn_Ep01/02_characters_locations/bios.md` (Stage 2,
confirmed 2026-07-26). This is the first real Stage 2→3 handoff using actual live-generated bios
output, not a hand-written test brief (see `tests/ep01-alien-dog-twins/` for the earlier
hand-written-brief version of this test).
