# HERMES-CINE

Multi-agent HERMES pipeline for generating animated clips/short episodes — idea → final cut.

**Status: design/skeleton phase.** See `docs/HERMES-CINE-SCAFFOLD.md` for the full architecture — pipeline stages, folder structure, multi-agent routing, model policy, and per-stage tool/skill requirements. Nothing here is a working agent yet; profiles below are placeholders until each stage is built out into a real `manifest.yaml` + `SOUL.md` package per your existing HERMES package spec.

## Repo layout

```
hermes-cine/
├── README.md                          — this file
├── docs/
│   └── HERMES-CINE-SCAFFOLD.md        — living design doc: architecture, stages, folder structure, routing
├── profiles/                          — one folder per HERMES agent profile (see naming table in scaffold §1.6)
│   ├── HERMES-CINE-ROUTER/            — orchestrator: dispatches jobs via `hermes -p <profile>`, tracks pipeline state — REAL PACKAGE (v1)
│   ├── HERMES-CINE-INTAKE/            — Stage 0: intake interview → project-brief.md — REAL PACKAGE (v1)
│   ├── HERMES-CINE-SCRIPT/            — Stage 1: script generation → script.md + character list — REAL PACKAGE (v1)
│   ├── HERMES-CINE-CHARDESIGN/        — Stage 2: character/location bios → bios.md (design-only, no package yet)
│   ├── HERMES-CINE-STORYBOARD/        — Stage 4: storyboard/shot list (not yet specced)
│   ├── HERMES-CINE-ASSEMBLY/          — Stage 8: edit/assembly (not yet specced)
│   └── HERMES-CINE-QCEXPORT/          — Stage 9: final QC/export (not yet specced)
│   (Stages 3/5/6/7 delegate to your existing `comfyui-expert` agent — no dedicated profile folder here)
└── tests/
    └── ep01-alien-dog-twins/          — mock end-to-end test run (Stages 0-1 text-flow tested; Stage 3 real comfyui-expert runs passed)
        ├── project-brief.md
        ├── script.md
        ├── comfyexpert-handoff-v1.md  — first test handoff (pre resolution-spec)
        └── comfyexpert-handoff-v2.md  — corrected handoff (1536x864 + 1024x1024 face-lock)
```

## Current validation status

See `docs/HERMES-CINE-SCAFFOLD.md` §5 for the full per-stage validation table. Short version:
`HERMES-CINE-ROUTER`, `HERMES-CINE-INTAKE`, and `HERMES-CINE-SCRIPT` now have **real generated
packages** (manifest.yaml + SOUL.md + eval.md + SETUP.md) — but none have been run through an
actual HERMES instance yet, so tool wiring, model resolution, and eval.md pass rates are all still
unverified. Stage 3 (via `comfyui-expert`) has real generation runs behind it and passed. Stages 0-1
are otherwise chat-mocked only. Stage 2 and beyond are design-only, no package yet.

## Next steps

1. **Run the three real packages** on the actual HERMES/EC2 instance: `hermes -p HERMES-CINE-INTAKE`,
   `hermes -p HERMES-CINE-SCRIPT`, `hermes -p HERMES-CINE-ROUTER` — check each self-configures per
   its SETUP.md and passes its eval.md.
2. **Validate the core architecture bet:** confirm `HERMES-CINE-ROUTER` can actually dispatch
   `HERMES-CINE-INTAKE` and `HERMES-CINE-SCRIPT` via `hermes -p <profile>` subprocess calls, and
   correctly reads README.md to decide what's next — this is the main open risk in the whole design.
3. Generate real packages for `HERMES-CINE-CHARDESIGN` next (Stage 2 is already fully specced).
4. Spec Stages 4, 8, 9 (currently placeholders).
5. Run each stage's test plan (docs §4) against the real running profile, not just chat-mocked.
