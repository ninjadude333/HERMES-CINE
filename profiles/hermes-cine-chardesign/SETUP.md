# SETUP.md — hermes-cine-chardesign

Directives for HERMES to self-configure this agent from the package in this folder.

1. **Resolve models.** Query `/api/tags`. For `worker`/`escalation`/`vision`, bind the best local
   model meeting the capability + constraints in `manifest.yaml`. If none qualifies, report the gap
   and suggest an `ollama pull` candidate; ask the owner before pulling. Bind `orchestrator` to the
   latest Bedrock Haiku.
2. **Wire tools** from the allowlist (`filesystem_rw`, `image_input`) with their stated scopes.
   **Disable the `skills` toolset entirely** — set `agent.disabled_toolsets: [skills]` in
   `config.yaml` as a real YAML list (not a quoted string — see hermes-cine-intake/SETUP.md for the
   live incident where a string-literal value silently failed to disable anything).
3. **Install guardrails:** session token budget (150,000), secrets-by-reference resolution from
   env/SSM. Enforce `blocked_actions` as hard denials — especially the ref-images-first ordering and
   the single-file (`bios.md`) convention, since a prior stage agent (hermes-cine-script) invented
   extra files on its own initiative without an explicit instruction against it.
4. **Load** `SOUL.md` as the system prompt, in particular the folder/stage-name/owner tables — these
   must be treated as authoritative, not regenerated from memory each run. No `kb/` or `skills/` in
   this package.
5. **Run `eval.md`.** Report pass/fail per test. If more than 2 must-have tests (T2–T12) fail, do
   not go live — review SOUL.md's Must-Have Answers against the failures.
6. **Go live** on configured channels (`telegram`, `cli`). Print a one-line status: agent name,
   bound models, eval pass rate.

## Notes specific to this agent

- This is a **Stage 2** agent in the HERMES-CINE multi-agent pipeline (see
  `docs/HERMES-CINE-SCAFFOLD.md` in the repo root). It is invoked by `hermes-cine-router` via
  `hermes -p hermes-cine-chardesign chat -q "<prompt>" -Q`, expects `script.md` (with its Character
  & Role List section) to already exist in `01_script/`, and hands off by writing `bios.md` into
  `02_characters_locations/` for the router to detect via `README.md`.
- If `script.md` is missing or unconfirmed (per README.md state), this agent should refuse to
  proceed and report the gap rather than guessing at script content.
- This agent runs on EC2, same as the other text-stage agents — not DGX. Ref images referenced
  during the interview are user-uploaded input at this stage, not generated; generation happens at
  Stage 3 (comfyui-expert) after this stage's output is confirmed and synced to DGX.
