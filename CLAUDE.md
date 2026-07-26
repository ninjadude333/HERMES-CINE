# CLAUDE.md — HERMES-CINE

This file is auto-loaded by Claude Code at the start of every session in this repo. Read it first,
then read `docs/HERMES-CINE-SCAFFOLD.md` in full before doing any work — that document is the
living source of truth for architecture and every locked decision. Do not re-derive or
second-guess decisions marked LOCKED in it without flagging that explicitly to the user first.

## What this project is

HERMES-CINE is a multi-agent HERMES pipeline that takes an idea and produces a finished animated
clip/short episode — idea → final cut. It's built as several dedicated HERMES agent profiles (not
one unified agent) that dispatch to each other via CLI, plus delegation to the user's existing
`comfyui-expert` HERMES agent for all ComfyUI generation work.

## Read order for a new session

1. `README.md` (repo root) — current file layout and status at a glance
2. `docs/HERMES-CINE-SCAFFOLD.md` — full architecture: pipeline stages, folder structure,
   multi-agent routing, model policy, failure handling, per-stage tool/skill requirements, test
   plans, validation status (§5 has the master status table)
3. `profiles/*/README.md` — per-profile status (which ones have real generated packages vs.
   placeholders)
4. `tests/ep01-alien-dog-twins/` — the one mock end-to-end test run so far (idea: twins, a dog,
   an alien abduction, Pixar style), used to validate Stage 0/1 design and real Stage 3
   comfyui-expert generation

## Current state (keep this section updated as work progresses)

- **Architecture:** fully locked — see scaffold §0, §1.5, §1.6, §1.7
- **Stages 0, 1:** fully specced AND have real generated packages (manifest.yaml + SOUL.md +
  eval.md + SETUP.md) in `profiles/HERMES-CINE-INTAKE/` and `profiles/HERMES-CINE-SCRIPT/`
- **Router:** fully specced AND has a real generated package in `profiles/HERMES-CINE-ROUTER/` —
  notably has NO worker/escalation model tier, orchestrator (Haiku) only, by design
- **Stage 2 (HERMES-CINE-CHARDESIGN):** fully specced in the scaffold, but no real package
  generated yet
- **Stage 3 (comfyui-expert delegation):** real generation runs completed and passed (2 test runs,
  resolution spec corrected between v1 and v2 handoff prompts — see tests/ folder)
- **Stages 4, 8, 9:** not yet specced at all — currently just placeholder profile folders
- **Nothing has been run through a real HERMES instance yet** except the comfyui-expert Stage 3
  test — Stages 0/1/Router are packages-on-paper only, not yet validated live

## The one big open architectural risk

The whole multi-agent design rests on: HERMES-CINE-ROUTER dispatching other stage agents via
`hermes -p <profile> -- <args>` as a subprocess (since one HERMES profile cannot spawn a different
profile as a child — see scaffold §1.6 for why). **This has never actually been run.** The highest-
value next test is running the three real packages on the real HERMES/EC2 instance and confirming
Router can dispatch Intake then Script in sequence, reading README.md correctly at each step.

## Conventions to follow without re-deriving

- Project folders: `/data/hermes-cine-projects/{SeriesSlug}_Ep{NN}/`, flat per-episode, numbered
  subfolders per stage (`00_brief/` … `09_final_export/`)
- `README.md` per project is the pipeline-state source of truth; `script.md` / `bios.md` are
  overwritten in place, never versioned
- Reference image filenames are descriptive:
  `{SeriesSlug}_Ep{NN}_char-{slug}_scene-ref_{expression}_{WxH}.png` etc. — see scaffold §1.5
- Ref image resolution: 1536×864 scene refs + 1024×1024 square face-lock crops per character
- Model routing is decided **per-stage**, never assumed blanket — check what each profile's
  manifest.yaml actually declares rather than assuming the standard Haiku→worker→escalation chain
- Naming convention for all stage agents is **functional** (`HERMES-CINE-SCRIPT`, not
  `HERMES-CINE-SCRIPTWRITER`) — see scaffold §1.6 naming table

## What NOT to do

- Don't generate a new manifest.yaml/SOUL.md for a stage without checking the scaffold's Stage
  section first — the must-have answers need to match the locked spec exactly, not be reinvented
- Don't assume Stages 4/8/9 details — they're genuinely unspecced, ask before filling gaps
- Don't have Router (or any stage agent) attempt another stage's work inline "to save a step" —
  that defeats the entire point of the multi-agent split
