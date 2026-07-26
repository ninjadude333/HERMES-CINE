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
│   ├── hermes-cine-router/            — orchestrator: dispatches jobs via `hermes -p <profile> chat -q "..." -Q`, tracks pipeline state — REAL PACKAGE (v1)
│   ├── hermes-cine-intake/            — Stage 0: intake interview → project-brief.md — REAL PACKAGE (v1)
│   ├── hermes-cine-script/            — Stage 1: script generation → script.md + character list — REAL PACKAGE (v1)
│   ├── hermes-cine-chardesign/        — Stage 2: character/location bios → bios.md (design-only, no package yet)
│   ├── hermes-cine-storyboard/        — Stage 4: storyboard/shot list (not yet specced)
│   ├── hermes-cine-assembly/          — Stage 8: edit/assembly (not yet specced)
│   └── hermes-cine-qcexport/          — Stage 9: final QC/export (not yet specced)
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
`hermes-cine-router`, `hermes-cine-intake`, and `hermes-cine-script` now have **real generated
packages** (manifest.yaml + SOUL.md + eval.md + SETUP.md) — but none have been run through an
actual HERMES instance yet, so tool wiring, model resolution, and eval.md pass rates are all still
unverified. Stage 3 (via `comfyui-expert`) has real generation runs behind it and passed. Stages 0-1
are otherwise chat-mocked only. Stage 2 and beyond are design-only, no package yet.

## Next steps

1. **Dispatch mechanism validated live (2026-07-26)** against the real HERMES CLI on
   hermano.dudelabz.com: `hermes -p <profile>` is real and per-invocation scoped, but the working
   non-interactive form is `hermes -p <profile> chat -q "<prompt>" -Q` — the `-- <args>` syntax from
   earlier drafts errors out, and exit code is unreliable (always 0, even on real failures) so
   README.md state-check is mandatory. Naming convention revised to lowercase-hyphenated repo-wide
   to match what the real CLI forces on profile directory names. See scaffold §1.6 for full detail.
2. **Install the three real packages as live profiles** on the EC2 host (`hermes profile create
   hermes-cine-intake/-script/-router --no-alias --no-skills`, copy package files in, run each
   SETUP.md) and check each passes its eval.md.
3. **Validate the core architecture bet:** confirm `hermes-cine-router` can actually dispatch
   `hermes-cine-intake` and `hermes-cine-script` via `hermes -p <profile> chat -q "..." -Q`
   subprocess calls, and correctly reads README.md to decide what's next — this is the main open
   risk in the whole design, now that the CLI syntax itself is confirmed.
4. Generate real packages for `hermes-cine-chardesign` next (Stage 2 is already fully specced).
5. Spec Stages 4, 8, 9 (currently placeholders).
6. Run each stage's test plan (docs §4) against the real running profile, not just chat-mocked.
