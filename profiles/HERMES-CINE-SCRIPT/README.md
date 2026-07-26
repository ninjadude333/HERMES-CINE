# HERMES-CINE-SCRIPT (Stage 1)

**Status: real package generated (v1).** manifest.yaml + SOUL.md + eval.md + SETUP.md are all
present in this folder, following the HermesAgentGenerator package spec.

**Design reference:** docs/HERMES-CINE-SCAFFOLD.md §2, Stage 1

**Not yet done:** run through a real HERMES instance. Currently only the design + a chat-mocked
script generation run (see tests/ep01-alien-dog-twins/script.md) have been validated.

**Next:** `hermes -p HERMES-CINE-SCRIPT` on the EC2 host, feeding it the real
HERMES-CINE-INTAKE output once that's also live, and check the eval.md report.
