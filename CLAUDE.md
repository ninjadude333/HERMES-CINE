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
- **Stages 0, 1, 2, Router: all four now have real generated packages** (manifest.yaml + SOUL.md +
  eval.md + SETUP.md) in `profiles/hermes-cine-intake/`, `profiles/hermes-cine-script/`,
  `profiles/hermes-cine-chardesign/`, `profiles/hermes-cine-router/` — Router notably has NO
  worker/escalation model tier, orchestrator (Haiku) only, by design
- **Stages 0, 1, Router: validated live on the real EC2 host (2026-07-26).** Multiple real chat
  sessions run against hermano.dudelabz.com — Intake produces a correct, confirmed project;
  Script produces a correct, confirmed script; Router successfully composed and ran a real
  `hermes -p <profile> chat -q "..." -Q` dispatch subprocess and correctly handled both a
  not-yet-built target (exit 1, reported cleanly) and an unconfirmed-state block. See scaffold §1.6
  and CLAUDE.md's "one big open architectural risk" section below for the full list of real bugs
  found and fixed along the way.
- **Stage 2 (hermes-cine-chardesign): package generated, not yet run live.** Folds in every
  live-validated lesson from Stages 0/1 before its first real test.
- **Stage 3 (comfyui-expert delegation):** real generation runs completed and passed (2 test runs,
  resolution spec corrected between v1 and v2 handoff prompts — see tests/ folder). Not yet tested
  with real Stage 2 output as input (still using a hand-written test brief).
- **Stages 4, 8, 9:** not yet specced at all — currently just placeholder profile folders

## The one big open architectural risk

The whole multi-agent design rests on: hermes-cine-router dispatching other stage agents via
`hermes -p <profile> chat -q "<prompt>" -Q` as a subprocess (since one HERMES profile cannot spawn a
different profile as a child — see scaffold §1.6 for why). **CLI dispatch mechanism itself validated
live on 2026-07-26** against the real HERMES CLI on hermano.dudelabz.com — `-p` works and is
per-invocation scoped, but two corrections were made to the original design (see scaffold §1.6 for
detail): the `-- <args>` syntax is invalid (use `chat -q "<prompt>" -Q` instead), and exit code is
not a trustworthy signal on this build (always 0, even on real failures) — README.md state-check is
mandatory, not a nice-to-have. Naming convention was also revised to lowercase-hyphenated
repo-wide, since the real CLI silently lowercases profile directory names and requires exact-case
match for `-p`.

**Live-installed and dispatch-tested 2026-07-26.** All three original packages are installed as
real profiles on the EC2 host. Router successfully composed and ran a real dispatch subprocess
against a project with a genuinely confirmed Stage 1 — the target profile (`hermes-cine-chardesign`)
didn't exist yet at the time, so the dispatch returned exit 1 and Router correctly reported the gap
rather than inventing a workaround. **Real bugs found and fixed along the way** (all now baked into
every package's SOUL.md/eval.md so future stages don't repeat them): a content-injection scanner
silently drops SOUL.md if it contains phrasing like "do not tell the user X" even when benign;
`manifest.yaml`'s `models: {worker, escalation}` tiers and `guardrails.human_gate` field are design
intent only — HERMES has no config-level enforcement for either, so SOUL.md must explicitly instruct
the model every time (validated live: Router's first real dispatch ran with zero approval prompt
until this was added); stage agents will autonomously self-author skills, invent extra files, leave
README.md partially updated, hallucinate file writes with zero real tool calls, and leak
project-specific details across sessions via memory — all now explicitly guarded against in SOUL.md.
**Next up:** hermes-cine-chardesign has a real package (folds in all of the above) but hasn't been
installed/run live yet — that's the next test, followed by a real Router-driven dispatch to it.

## Conventions to follow without re-deriving

- Project folders: `~/hermes-cine-projects/{SeriesSlug}_Ep{NN}/` on EC2 (text stages 0-2 stage
  here) synced to `/data/hermes-cine-projects/{SeriesSlug}_Ep{NN}/` on DGX before Stage 3 generation
  (revised 2026-07-26 after live host validation — see scaffold §1.5). Flat per-episode, numbered
  subfolders per stage (`00_brief/` … `09_final_export/`)
- `README.md` per project is the pipeline-state source of truth; `script.md` / `bios.md` are
  overwritten in place, never versioned
- Reference image filenames are descriptive:
  `{SeriesSlug}_Ep{NN}_char-{slug}_scene-ref_{expression}_{WxH}.png` etc. — see scaffold §1.5
- Ref image resolution: 1536×864 scene refs + 1024×1024 square face-lock crops per character
- Model routing is decided **per-stage**, never assumed blanket — check what each profile's
  manifest.yaml actually declares rather than assuming the standard Haiku→worker→escalation chain
- Naming convention for all stage agents is **functional** (`hermes-cine-script`, not
  `hermes-cine-scriptWRITER`) — see scaffold §1.6 naming table

## What NOT to do

- Don't generate a new manifest.yaml/SOUL.md for a stage without checking the scaffold's Stage
  section first — the must-have answers need to match the locked spec exactly, not be reinvented
- Don't assume Stages 4/8/9 details — they're genuinely unspecced, ask before filling gaps
- Don't have Router (or any stage agent) attempt another stage's work inline "to save a step" —
  that defeats the entire point of the multi-agent split
