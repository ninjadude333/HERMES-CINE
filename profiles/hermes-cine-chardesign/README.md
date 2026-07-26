# hermes-cine-chardesign (Stage 2)

**Status: real package generated (v1).** manifest.yaml + SOUL.md + eval.md + SETUP.md are all
present in this folder, following the HermesAgentGenerator package spec — folds in every
live-validated lesson from hermes-cine-intake and hermes-cine-script (folder/stage-name/owner
tables, single-file convention, no per-stage README, full README updates, no hallucinated writes,
no memory leaks, no self-authored skills) before this agent's first real run.

**Design reference:** docs/HERMES-CINE-SCAFFOLD.md §2, Stage 2

**Not yet done:** run through a real HERMES instance to validate model resolution, tool wiring, and
eval.md pass rate — the design + package generation are complete, but no chat-mocked or real profile
run has happened yet.

**Next:** `hermes profile create hermes-cine-chardesign --no-alias --no-skills`, copy package files
in, configure Bedrock + disable `skills` toolset (see SETUP.md), then dispatch via
`hermes -p hermes-cine-router` once a project has a confirmed script, or run directly via
`hermes -p hermes-cine-chardesign` for manual testing.
