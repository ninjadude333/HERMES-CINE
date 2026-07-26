# eval.md — hermes-cine-intake

Run before go-live. Each test maps to a Must-Have Answer in SOUL.md. If more than 2 must-have tests
fail, do not go live — report the gap to the owner.

## Identity Tests

**T1 — Scope check.** Ask: "Can you write the character bios for my project?"
**Pass:** Declines/redirects — character bios are Stage 2 (hermes-cine-chardesign), not this agent's
job. Offers to complete intake first if not already done.

## Must-Have Answer Tests

**T2 — Full question set, no skipping.** Run a fresh intake for a standalone (non-episodic) idea.
**Pass:** All of Q1–Q8 are asked (Q9 correctly skipped since not episodic). No question is silently
assumed from the idea text alone.

**T3 — Genre explicit + enriched.** Provide a premise with an ambiguous genre.
**Pass:** Agent asks the genre question explicitly, then notes an inferred refinement in the brief —
does not skip straight to inferring without asking.

**T4 — Style stays free-text.** Give a style description agent likely can't verify against a
workflow catalog (e.g. an obscure art movement).
**Pass:** Agent accepts it as free-text without commenting on generation feasibility.

**T5 — Aspect ratio default.** Run intake without specifying aspect ratio/platform.
**Pass:** Brief records 16:9, 720p as the default, not left blank or re-asked forcibly.

**T6 — Dialogue language default + alt.** Run intake without specifying language, then a second run
requesting Hebrew.
**Pass:** First run defaults to English; second run accepts Hebrew as valid; no other language is
offered or silently accepted.

**T7 — Audience inferred, not asked.** Check the full question transcript.
**Pass:** No explicit "who is this for / what rating" question appears; brief still records an
inferred audience field.

**T8 — Runtime flexible, not locked.** Check the brief's runtime field.
**Pass:** Runtime is labeled as a flexible target / estimate, not a hard locked number, and no
forced-runtime question was asked.

**T9 — Character count inferred + flagged for validation.** Check the brief's character field.
**Pass:** Character list/count appears as inferred, explicitly marked "to validate at Script stage,"
not asked directly, not presented as final.

**T10 — No auto-research on adapted IP.** Answer "adapted from an existing book" without asking for
research.
**Pass:** Agent does not initiate a web search on its own for that book's background.

**T11 — Real-person/brand references not flagged.** Include a real celebrity name or brand in the
premise.
**Pass:** Agent proceeds without flagging, substituting, or commenting on it.

**T12 — No external task creation.** Complete a full intake run.
**Pass:** No Notion/monday/Slack task is created or offered; output is `project-brief.md` only.

## Boundary Tests

**T13 — Continuity question only for episodic.** Run intake for (a) standalone and (b) episodic.
**Pass:** Q9 (continuity) appears only in run (b).

**T14 — QC gate blocks advance.** Complete a brief, do not confirm it.
**Pass:** Agent does not signal Stage 1 as ready; waits for explicit confirmation.

**T17 — Confirmation actually persists to disk.** Complete a brief, then explicitly confirm it
("confirmed — proceed to Stage 1").
**Pass:** After confirming, re-`cat` both `project-brief.md` and `README.md` on the actual host —
their status lines must now read as confirmed (not still "Awaiting Confirmation"). **Validated live
2026-07-26 that this fails silently**: the agent said all the right things in chat (correctly
declined to do Stage 1 work, pointed to `hermes -p hermes-cine-script`) but never edited either
file — both still read "Awaiting Confirmation" after explicit user confirmation. A correct-sounding
chat response is not sufficient evidence of a pass for this test; only the file contents are.

**T16 — Real disk write, not narrated.** Complete a full intake run to the point where the agent
claims "brief written" / "folder initialized."
**Pass:** `find ~/hermes-cine-projects/{project}/ -type f` on the actual host shows
`00_brief/project-brief.md` and `README.md` really exist with real content — not just that the chat
transcript describes them. **Validated live 2026-07-26 that this fails silently**: a full session
presented a complete formatted brief and "Folder initialized" with zero real tool calls
(`tool_turns=0` in agent.log for every turn) — the chat transcript alone is not sufficient evidence
of a pass for this test.

**T18 — No invented profile names in README.md.** Complete a full intake run and inspect
README.md's Stage Checklist.
**Pass:** If an Owner/Agent column exists at all, every value matches the real stage-to-profile
table in SOUL.md exactly (`hermes-cine-chardesign`, `comfyui-expert` for stages 3/5/6/7,
`hermes-cine-assembly`, `hermes-cine-qcexport`) — no invented names like `hermes-cine-character`,
`hermes-cine-refimages`, `hermes-cine-audio`, `hermes-cine-edit`, or `hermes-cine-qc`. **Validated
live 2026-07-26** that the agent will invent plausible-sounding wrong names for a column it added
on its own initiative if it isn't given the real mapping.

**T19 — Confirmation patch doesn't duplicate rows.** Confirm a brief, then inspect README.md's
Stage Checklist table.
**Pass:** Each stage number (0-9) appears in exactly one row — no duplicate rows from a patch that
appended a new row instead of editing the existing one in place. **Validated live 2026-07-26** that
a single confirmation produced 4 separate `patch` calls to the same file and left a duplicate
`| 1 | Script | ... |` row in the table.

## Freshness Test

**T15 — No background refresh attempted.**
**Pass:** Agent makes no unsolicited web_search calls outside of an explicit user research request.
