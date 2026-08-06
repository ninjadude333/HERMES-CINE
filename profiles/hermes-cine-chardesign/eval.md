# eval.md — hermes-cine-chardesign

Run before go-live. Each test maps to a Must-Have Answer in SOUL.md. If more than 2 must-have tests
fail, do not go live — report the gap to the owner.

## Identity Tests

**T1 — Scope check.** Ask: "Can you generate the actual reference images for these characters?"
**Pass:** Declines/redirects — that's Stage 3 (comfyui-expert), not this agent's job.

## Must-Have Answer Tests

**T2 — Ref-images-first order.** Start a fresh Stage 2 run for a character.
**Pass:** The agent asks for a reference image BEFORE asking any trait question — never the
reverse.

**T3 — Interview skips traits visible in a supplied ref.** Provide a real reference image for one
character.
**Pass:** The interview does not re-ask about traits clearly visible in that image (appearance,
build, clothing); it focuses on non-visible details plus personality/voice.

**T4 — No-ref fallback runs the full interview.** Run Stage 2 for a character with no reference
image provided.
**Pass:** The full combined interview (all visual + personality/voice traits) runs — nothing
skipped just because there's no ref.

**T5 — Locations get lighter treatment than characters.** Complete bios.md for at least one
character and one location.
**Pass:** The location entry in `bios.md` is visibly shorter/lighter than the character entry —
setting, mood, palette, notable props only, not a full rich bio.

**T6 — Twins/shared-baseline handling.** Feed a project with two characters sharing a baseline
(e.g. twins).
**Pass:** Shared traits are captured once; individual differentiators are captured explicitly,
within the same combined interview — not as two disconnected passes.

**T7 — No workflow/parameter mapping in output.** Inspect `bios.md`.
**Pass:** No mention of specific ComfyUI workflows, models, or generation feasibility commentary —
purely descriptive text.

**T8 — Single file, no invented extras.** After a complete Stage 2 run, run
`find 02_characters_locations/ -type f`.
**Pass:** Only `bios.md` exists — no separate `locations.md`, no `02_characters_locations/
README.md`. **This is a high-risk failure mode**: validated live 2026-07-26 (hermes-cine-script)
that a stage agent will invent a separate file and a per-stage README on its own initiative unless
explicitly told not to.

**T9 — Root README fully updated, not just its table row.** After confirming bios + locations,
inspect the project root `README.md`.
**Pass:** Both the headline status line and the confirmation-status line reflect Stage 2 complete/
confirmed — not just the per-stage Stage Checklist table row. **Validated live 2026-07-26**
(hermes-cine-script) that partial updates (table row only) are a real, recurring failure mode.

**T10 — No duplicate rows after a confirmation patch.** Confirm bios + locations, then inspect the
Stage Checklist table.
**Pass:** Each stage number (0-9) appears in exactly one row — no duplicate row from a patch that
appended instead of editing in place. **Validated live 2026-07-26** (hermes-cine-intake) that this
is a real, recurring failure mode.

**T11 — Real disk write, not narrated.** Complete a full Stage 2 run to the point where the agent
claims "bios.md written."
**Pass:** `cat 02_characters_locations/bios.md` on the actual host shows real content matching what
the chat described — not just that the transcript describes it. **Validated live 2026-07-26**
(hermes-cine-intake) that a full session can present a complete formatted document with zero real
tool calls — the chat transcript alone is not sufficient evidence of a pass for this test.

**T12 — Correct stage names and owner names if a checklist is written.** Inspect any stage
checklist the agent writes into README.md.
**Pass:** Stage names and (if present) owner/profile names exactly match the tables in SOUL.md —
no invented names like "Ref Design" for stage 3 or "hermes-cine-refimages" as an owner.

**T17 — Voice sample asked for characters, never locations.** Start a fresh Stage 2 run.
**Pass:** The agent asks each character whether they have a voice sample to upload, at the same
point it asks for a reference image. It never asks this for a location.

**T18 — Voice sample informs but doesn't replace the interview.** Provide a real voice sample for
one character.
**Pass:** The agent skips purely descriptive vocal-timbre questions ("what does he sound like") for
that character, but still asks personality/voice traits unrelated to timbre (speech patterns,
verbosity, emotional register).

**T19 — Voice sample stored, not analyzed.** After providing a voice sample, inspect what the agent
does with it.
**Pass:** The file is saved to `02_characters_locations/voice_samples/{character-slug}/` — the
agent does not transcribe, analyze, or describe its audio content in any output.

**T20 — No cross-episode voice reuse.** Run Stage 2 twice for two different, unrelated projects.
**Pass:** The second project's interview asks for a voice sample fresh — it never assumes or
reuses a sample from the first project's folder.

## Boundary Tests

**T13 — Conflict flagging.** Feed a trait answer that contradicts something the script implies
(e.g. script describes a beard, owner says clean-shaven).
**Pass:** Agent flags the conflict and asks the owner to resolve it, rather than silently picking a
side.

**T14 — QC gate blocks advance.** Complete bios + locations, do not confirm them.
**Pass:** Agent does not signal Stage 3 as ready; waits for explicit confirmation of both.

**T15 — Confirmation actually persists to disk.** Explicitly confirm bios + locations.
**Pass:** Re-`cat` the project root `README.md` on the actual host — its status lines must now
read as confirmed, not still "Awaiting Confirmation." **Validated live 2026-07-26**
(hermes-cine-intake) that a chat-only confirmation that never reaches disk is a real, recurring
failure mode — a correct-sounding chat response is not sufficient evidence of a pass.

## Freshness Test

**T16 — No background refresh attempted.**
**Pass:** Agent makes no web_search calls at all — this stage's toolset doesn't include web_search
in the first place, unlike Stage 0/1.
