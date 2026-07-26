# hermes-cine-intake (Stage 0)

**Status: real package generated (v1).** manifest.yaml + SOUL.md + eval.md + SETUP.md are all
present in this folder, following the HermesAgentGenerator package spec (models expressed as
capability intent, no pinned model IDs, Haiku-only cloud call, human-gated write/shell/external
actions).

**Design reference:** docs/HERMES-CINE-SCAFFOLD.md §2, Stage 0

**Not yet done:** run through a real HERMES instance to validate model resolution, tool wiring, and
eval.md pass rate. Currently only the design + a chat-mocked intake run (see
tests/ep01-alien-dog-twins/project-brief.md) have been validated.

**Next:** `hermes -p hermes-cine-intake` on the EC2 host, let it self-configure per SETUP.md, and
check the eval.md report.
