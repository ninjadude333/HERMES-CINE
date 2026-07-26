# HERMES-CINE

Multi-agent HERMES pipeline for generating animated clips/short episodes — idea → final cut.

**Status: early live-validation phase.** See `docs/HERMES-CINE-SCAFFOLD.md` for the full architecture — pipeline stages, folder structure, multi-agent routing, model policy, and per-stage tool/skill requirements. Stages 0, 1, 2, and the Router have real generated packages; Stages 0/1/Router have been run for real against the live HERMES instance on EC2 and had multiple real bugs found and fixed (see CLAUDE.md's "one big open architectural risk" section for the log). Stage 2 has a package but hasn't been run live yet.

## Repo layout

```
hermes-cine/
├── README.md                          — this file
├── docs/
│   └── HERMES-CINE-SCAFFOLD.md        — living design doc: architecture, stages, folder structure, routing
├── profiles/                          — one folder per HERMES agent profile (see naming table in scaffold §1.6)
│   ├── hermes-cine-router/            — orchestrator: dispatches jobs via `hermes -p <profile> chat -q "..." -Q`, tracks pipeline state — REAL PACKAGE, LIVE-TESTED
│   ├── hermes-cine-intake/            — Stage 0: intake interview → project-brief.md — REAL PACKAGE, LIVE-TESTED
│   ├── hermes-cine-script/            — Stage 1: script generation → script.md + character list — REAL PACKAGE, LIVE-TESTED
│   ├── hermes-cine-chardesign/        — Stage 2: character/location bios → bios.md — REAL PACKAGE (v1), not yet live-tested
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

- **hermes-cine-intake, hermes-cine-script, hermes-cine-router** are installed as real live profiles on the EC2 host (hermano.dudelabz.com) and have each been run for real, multiple times, against real project state. Every bug found during those runs is documented and fixed in the relevant SOUL.md/eval.md — see the list in CLAUDE.md.
- **hermes-cine-router successfully ran a real dispatch subprocess** (`hermes -p hermes-cine-chardesign chat -q "..." -Q`) against a project with a genuinely confirmed Stage 1. The target profile didn't exist yet at the time, so the call returned exit 1 — Router correctly reported the gap instead of working around it. This validates the CLI dispatch mechanism itself; dispatching to a profile that exists and runs to completion is the next real test.
- **hermes-cine-chardesign** has a real package (folds in every lesson learned from the three profiles above) but has not yet been installed or run live.
- Stage 3 (via `comfyui-expert`) has real generation runs behind it and passed, using a hand-written test brief — not yet re-tested against real Stage 2 output.
- Stages 4, 8, 9 are design-only placeholders, not yet specced.

## Next steps

1. **Install `hermes-cine-chardesign` as a live profile** on the EC2 host (`hermes profile create hermes-cine-chardesign --no-alias --no-skills`, copy package files in, configure Bedrock provider + `agent.disabled_toolsets: [skills]` in config.yaml — see its SETUP.md) and run it directly against the confirmed script from the current test project.
2. **Have Router dispatch a real, complete run** — point `hermes-cine-router` at a project with Stage 1 confirmed and `hermes-cine-chardesign` actually installed, and confirm the full dispatch → run → README.md update → Router re-check loop works end to end, not just the dispatch-and-fail path already validated.
3. Generate a real package for `hermes-cine-storyboard` next (Stage 4 needs specing first — currently a placeholder).
4. Spec Stages 4, 8, 9 (currently placeholders).
5. Once Stage 2 output exists for real, re-run the Stage 3 (`comfyui-expert`) handoff test using that real `bios.md` instead of the hand-written test brief in `tests/ep01-alien-dog-twins/`.
