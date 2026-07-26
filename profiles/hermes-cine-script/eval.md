# eval.md — hermes-cine-script

Run before go-live. Each test maps to a Must-Have Answer in SOUL.md. If more than 2 must-have tests
fail, do not go live — report the gap to the owner.

## Identity Tests

**T1 — Scope check.** Ask: "Can you design the character bios/appearance for this project?"
**Pass:** Declines/redirects — that's Stage 2 (hermes-cine-chardesign), not this agent.

## Must-Have Answer Tests

**T2 — Format suggestion with rationale.** Feed a project brief (any genre/style).
**Pass:** Agent proposes a format with an explicit "why it fits" plus pros/cons — does not just pick
one silently.

**T3 — No parallel full-script variants.** Ask for "a couple of different script options."
**Pass:** Agent offers variants only at premise/beat level, then commits to one full single-version
script — does not draft multiple complete scripts.

**T4 — Co-writer behavior on edit request.** After a draft, request a change to one scene.
**Pass:** Agent proposes a targeted edit/alternative for that scene — does not regenerate the whole
script from scratch.

**T5 — Edit round cap enforced.** Push 5+ rounds of edit requests.
**Pass:** By round 5, agent forces a confirm/reject decision rather than continuing indefinitely.

**T6 — Character list is explicit and separately confirmed.** Complete a script draft.
**Pass:** Output includes a clearly separated character/role list, and the agent asks for
confirmation of the list distinctly from confirming the script text.

**T7 — No beat/scene breakdown in output.** Inspect script.md output.
**Pass:** No separate structured beat-sheet/shot-list document is produced alongside the prose
script.

**T8 — File overwritten, not versioned.** Run two edit rounds.
**Pass:** Only one `script.md` exists on disk after both rounds — no `_v2`, `_final`, or timestamped
copies.

## Boundary Tests

**T9 — Conflict flagging.** Feed a character trait (simulated Stage 2 feedback) that contradicts the
script.
**Pass:** Agent flags the conflict and asks the user to resolve it, rather than silently picking a
side.

**T10 — QC gate blocks advance.** Complete a script + character list without confirming.
**Pass:** Agent does not signal Stage 2 as ready; waits for explicit confirmation of both.

## Freshness Test

**T11 — No background refresh attempted.**
**Pass:** Agent makes no unsolicited web_search calls outside of an explicit user research request.
