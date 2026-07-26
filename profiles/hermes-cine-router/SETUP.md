# SETUP.md — hermes-cine-router

Directives for HERMES to self-configure this agent from the package in this folder.

1. **Resolve models.** Query `/api/tags` is not required for this agent — it has no `worker` or
   `escalation` tier by design (see `manifest.yaml` note). Bind `orchestrator` to the latest Bedrock
   Haiku only.
2. **Wire tools** from the allowlist (`filesystem_rw`, `shell_exec`, `telegram_send`, `email_send`)
   with their stated scopes. `shell_exec` is human-gated — every dispatch call must pause for owner
   approval per `guardrails.human_gate`, since it's the mechanism that actually kicks off other
   agents' work and real generation cost. **Disable the `skills` toolset entirely** — set
   `agent.disabled_toolsets: [skills]` in `config.yaml` as a real YAML list (not a quoted string —
   see hermes-cine-intake/SETUP.md for the live incident where a string-literal value silently
   failed to disable anything).
3. **Install guardrails:** human gate, session token budget (50,000 — this agent's job is small,
   frequent, cheap calls, not long generation), secrets-by-reference resolution from env/SSM.
   Enforce `blocked_actions` as hard denials — particularly the "advance on exit code alone" and
   "retry more than once" rules, since those are the two most likely failure modes for an
   orchestrator to drift into under time pressure.
4. **Load** `SOUL.md` as the system prompt, in particular the stage-to-profile mapping table — this
   must be treated as authoritative, not regenerated from memory each run.
5. **Run `eval.md`.** Report pass/fail per test. If more than 2 must-have tests (T2–T7) fail, do not
   go live.
6. **Go live** on configured channels (`telegram`, `cli`). Print a one-line status: agent name,
   bound model (orchestrator only), eval pass rate.

## Notes specific to this agent

- **This agent must run on EC2**, same host as the other HERMES instances, so `shell_exec` can
  invoke `hermes -p <profile> chat -q "<prompt>" -Q` locally without SSH/remote-dispatch overhead
  (per `docs/HERMES-CINE-SCAFFOLD.md` §1.6). The `-Q` (quiet) flag and `chat -q` form are required —
  the literal `-- <args>` syntax from earlier drafts does not work on this HERMES build.
- This agent is the **entry point** for the whole HERMES-CINE pipeline — the owner talks to this
  agent ("continue the AlienDogTwins project"), not directly to individual stage agents, except
  when manually intervening.
- As Stages 4, 8, 9 get built, add their profile folders alongside this one and they'll be picked up
  automatically via the stage-to-profile table in SOUL.md — no change needed to this SETUP.md.
