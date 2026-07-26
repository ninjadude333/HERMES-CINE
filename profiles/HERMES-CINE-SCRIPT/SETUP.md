# SETUP.md — HERMES-CINE-SCRIPT

Directives for HERMES to self-configure this agent from the package in this folder.

1. **Resolve models.** Query `/api/tags`. For `worker`/`escalation`, bind the best local model
   meeting the capability + constraints in `manifest.yaml`. If none qualifies, report the gap and
   suggest an `ollama pull` candidate; ask the owner before pulling. Bind `orchestrator` to the
   latest Bedrock Haiku. No `vision` tier — do not bind one.
2. **Wire tools** from the allowlist (`filesystem_rw`, `web_search`) with their stated scopes.
   Attach the human gate to any tool call matching `guardrails.human_gate.fires_on`.
3. **Install guardrails:** human gate, session token budget (150,000), secrets-by-reference
   resolution from env/SSM. Enforce `blocked_actions` as hard denials — especially the edit-round
   cap and the single-version-script rule, since these are behavioral, not just tool-permission,
   constraints and must be reinforced by SOUL.md at runtime, not only by this config.
4. **Load** `SOUL.md` as the system prompt. No `kb/` or `skills/` in this package.
5. **Run `eval.md`.** Report pass/fail per test. If more than 2 must-have tests (T2–T8) fail, do not
   go live — review SOUL.md's Must-Have Answers against the failures, or consider escalating to a
   larger worker model if failures look like reasoning gaps.
6. **Go live** on configured channels (`telegram`, `cli`). Print a one-line status: agent name,
   bound models, eval pass rate.

## Notes specific to this agent

- This is a **Stage 1** agent in the HERMES-CINE multi-agent pipeline (see
  `docs/HERMES-CINE-SCAFFOLD.md` in the repo root). It is invoked by `HERMES-CINE-ROUTER` via
  `hermes -p HERMES-CINE-SCRIPT -- <args>`, expects `project-brief.md` to already exist in
  `00_brief/`, and hands off by writing `script.md` + character list into `01_script/` for the
  router to detect via `README.md`.
- If `project-brief.md` is missing or unconfirmed (per README.md state), this agent should refuse to
  proceed and report the gap rather than guessing at brief content.
