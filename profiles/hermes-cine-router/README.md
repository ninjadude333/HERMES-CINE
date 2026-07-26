# hermes-cine-router

**Status: real package generated (v1).** manifest.yaml + SOUL.md + eval.md + SETUP.md are all
present in this folder. Notably has NO worker/escalation tier — orchestrator (Haiku) only, by
design, since this agent does pure dispatch/state-machine logic, not content generation.

**Design reference:** docs/HERMES-CINE-SCAFFOLD.md §1.6, §1.7

**Not yet done:** run through a real HERMES instance. This is the first real test of the "per-stage
agents can't spawn each other, must dispatch via CLI" architecture decision — validating this is
the main point of building this package now.

**Next:** `hermes -p hermes-cine-router` on the EC2 host, point it at a real project folder with
hermes-cine-intake and hermes-cine-script also live, and confirm it correctly dispatches
`hermes -p hermes-cine-intake` / `hermes -p hermes-cine-script` in sequence.
