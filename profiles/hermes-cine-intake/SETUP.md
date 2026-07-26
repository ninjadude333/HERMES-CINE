# SETUP.md — hermes-cine-intake

Directives for HERMES to self-configure this agent from the package in this folder.

1. **Resolve models.** Query `/api/tags`. For `worker`/`escalation`, bind the best local model
   meeting the capability + constraints in `manifest.yaml`. If none qualifies, report the gap and
   suggest an `ollama pull` candidate; ask the owner before pulling. Bind `orchestrator` to the
   latest Bedrock Haiku. No `vision` tier — do not bind one.
2. **Wire tools** from the allowlist in `manifest.yaml` (`filesystem_rw`, `image_input`,
   `web_search`, `read_prior_project`) with their stated scopes. Attach the human gate to any tool
   call matching `guardrails.human_gate.fires_on` (state-changing, external call, shell, write).
3. **Install guardrails:** human gate, session token budget (100,000), secrets-by-reference
   resolution from env/SSM. Enforce `blocked_actions` from `manifest.yaml` as hard denials, not
   soft preferences.
4. **Load** `SOUL.md` as the system prompt. No `kb/` or `skills/` directories in this package —
   this agent's domain knowledge is small enough to live entirely in SOUL.md.
5. **Run `eval.md`.** Report pass/fail per test. If more than 2 must-have tests (T2–T12) fail, do
   not go live — recommend reviewing SOUL.md's Must-Have Answers section against the failing tests,
   or escalating to a larger worker model if failures look like reasoning gaps rather than
   instruction gaps.
6. **Go live** on the configured channels (`telegram`, `cli`). Print a one-line status: agent name,
   bound models (orchestrator/worker/escalation), eval pass rate.

## Notes specific to this agent

- This is a **Stage 0** agent in the HERMES-CINE multi-agent pipeline (see
  `docs/HERMES-CINE-SCAFFOLD.md` in the repo root for full pipeline context). It does not call
  other HERMES profiles — it is invoked by `hermes-cine-router` via
  `hermes -p hermes-cine-intake chat -q "<prompt>" -Q` and hands off by writing `project-brief.md` +
  `README.md` for the router to detect.
- This agent runs on EC2, not DGX. If the project folder root (`~/hermes-cine-projects/`) doesn't
  exist yet on this host, create it on first run rather than failing. Sync to DGX
  (`/data/hermes-cine-projects/`) happens later, before Stage 3 generation — not this agent's job.
