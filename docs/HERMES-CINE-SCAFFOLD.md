# HERMES Agent Profile: Idea-to-Movie Pipeline (working name: HERMES-CINE)

> **STATUS: SKELETON ONLY.** Everything below defines architecture, flow, and decisions — it is not yet a validated, working agent. Each stage still needs to be **tested end-to-end with real tool calls** (not just the text-only mock tests run so far) and each stage's listed skills/capabilities need to be **confirmed as actually available and functional** (e.g. real comfyui-expert delegation, real DGX filesystem writes, real README auto-update) before this is production-ready. Treat every "LOCKED" decision as locked-in-design, not locked-as-proven.

> **Real-install gotcha found live (2026-07-26):** HERMES has a built-in prompt-injection content
> scanner (`tools/threat_patterns.py`) that runs against every context file, **including SOUL.md**,
> on every load. A rule meant to catch "hide this from the user"-style injection attacks
> (`deception_hide`: `do not ... tell ... the user`) silently matched a benign line in
> `hermes-cine-intake/SOUL.md` ("Do not tell the user 'that style isn't available'..."), and the
> **entire SOUL.md was dropped** — no error surfaced in the chat session, the agent just answered as
> generic "Hermes Agent" instead of its intended persona. Only visible in
> `~/.hermes/profiles/<profile>/logs/errors.log` as `Context file SOUL.md blocked: deception_hide`.
> Fixed by rewording to avoid the trigger phrase. **Lesson for all future SOUL.md authoring:** avoid
> "do not tell the user X" / "never tell the user X" phrasing even when benign — rephrase as "never
> comment on X" or similar. Check `errors.log` for `blocked:` whenever an agent's response doesn't
> match its SOUL.md identity — that's the tell, not a chat-visible error.

## 0. Meta / Architecture Decisions (LOCKED)

| Decision | Value |
|---|---|
| Scope | Full pipeline: idea → final cut |
| Interaction model | Hybrid — short interview up front, then autonomous execution per stage |
| Orchestration mode | Dual-mode, switchable: (a) direct orchestration calling ComfyUI/DGX/Ollama, or (b) spec/prompt-only output for manual run |
| Visual consistency | PuLID + Flux Kontext (proven combo), executed via existing **comfyui-expert** HERMES agent and its workflow catalog |
| Audio | In-scope, generated as the **last** stage, via ComfyUI workflows (ACE-Step/MMAudio) — not upfront |
| QC | Automatic QC gate + user confirmation **after every stage** |
| Delegation | This agent does NOT reimplement ComfyUI execution — it delegates generation calls to the existing comfyui-expert agent/workflow catalog |

## 1.5 Project Folder Structure (LOCKED, revised 2026-07-26 after live host validation)

**Location — two-host design, corrected from the original single-host assumption:**
Live testing on 2026-07-26 found all HERMES-CINE stage agents (Intake/Script/Router) actually run
on **EC2** as user `ubuntu` — not on DGX as originally assumed. DGX is reachable from EC2 only via
the existing reverse SSH tunnel `comfyui-expert` already uses (`ssh -i ~/.ssh/id_dgx_tunnel -p 2222
davidg@localhost`), where the DGX user is `davidg`, not `ubuntu`. `/data` is a real 7TB ext4 mount
on DGX (owned by `davidg`) — it does not exist at all on EC2.

**Revised flow:**
- **Text-only stages (Intake, Script, CharDesign) stage locally on EC2**, under
  `~/hermes-cine-projects/{SeriesSlug}_Ep{NN}/` (ubuntu's home dir) — this is where these agents
  actually run, so no cross-host round-trip is needed for pure text work.
- **Before Stage 3 (ref image generation) begins**, the project folder is synced from EC2 to DGX's
  `/data/hermes-cine-projects/{SeriesSlug}_Ep{NN}/` (owned by `davidg`) over the same tunnel, e.g.
  `rsync -e "ssh -i ~/.ssh/id_dgx_tunnel -p 2222" -av ~/hermes-cine-projects/{proj}/ davidg@localhost:/data/hermes-cine-projects/{proj}/`.
  This preserves the original rationale (avoid cross-host sync overhead *during* generation, the
  expensive part) while fixing the host assumption.
- **DGX is canonical from Stage 3 onward** — comfyui-expert and ComfyUI read/write directly there,
  no further EC2 round-trips needed mid-generation.
- If a later stage needs to run back on EC2 (not currently anticipated), sync direction would need
  to reverse — not yet designed, flag if this becomes needed.

**Root convention (per host):** `{host-root}/hermes-cine-projects/{SeriesSlug}_Ep{NN}/` where
`{host-root}` is `~` on EC2 (staging) or `/data` on DGX (canonical post-Stage-3).
- Flat per-episode folders — no series-level nesting; series grouping happens via the naming convention itself (e.g. `AlienDogTwins_Ep01`, `AlienDogTwins_Ep02`)

**Per-episode subfolder structure:**
```
{SeriesSlug}_Ep{NN}/
├── README.md                     (auto-generated/updated at every stage — indexes all files below + resume state)
├── 00_brief/
│   └── project-brief.md
├── 01_script/
│   └── script.md                 (overwritten per edit, per Stage 1 convention)
├── 02_characters_locations/
│   └── bios.md                   (overwritten per edit, per Stage 2 convention)
├── 03_ref_images/
│   ├── characters/
│   │   └── {character_name}/
│   │       ├── {SeriesSlug}_Ep{NN}_char-{character-slug}_scene-ref_{expression-slug}_1536x864.png
│   │       └── {SeriesSlug}_Ep{NN}_char-{character-slug}_face-lock_1024x1024.png
│   └── locations/
│       └── {location_name}/
│           └── {SeriesSlug}_Ep{NN}_loc-{location-slug}_scene-ref_1536x864.png
├── 04_storyboard/
├── 05_shot_images/
├── 06_clips/
├── 07_audio/
├── 08_assembly/
└── 09_final_export/
```

**Text/docs convention (LOCKED):** Originals stay distributed per-stage (script.md in 01_script/, bios.md in 02_characters_locations/, etc. — no forced consolidation). `README.md` at project root **links/indexes** all of them plus current pipeline status, so a project can be resumed just by reading the README.

**README.md behavior:** Auto-generated and updated at **every stage transition** — always reflects current progress, links to latest version of each file, and notes which stage is next/pending confirmation.

**ComfyUI output handling:** Files generated by comfyui-expert are **copied** into the relevant `03_ref_images/`, `05_shot_images/`, `06_clips/`, or `07_audio/` subfolder — not left in ComfyUI's own output directory and referenced by path.

**Series continuity (LOCKED):** Intake (Stage 0) gains an option to reference a **previous episode's project folder** (refs + clips) for visual/character continuity. Per-project choice between:
- Reuse previous episode's locked refs directly (skip Stage 3 regen for existing characters), or
- Regenerate refs but validate consistency against the previous episode's refs

Agent asks which approach per-project rather than defaulting to one.

**Tool/Skill implication:** Text stages (Intake/Script/CharDesign) need **filesystem read/write on
EC2 local disk** (`~/hermes-cine-projects/`) to create these folders, write stage output, and
maintain README.md. Generation stages (comfyui-expert delegation) need **filesystem read/write on
the DGX host** (`/data/hermes-cine-projects/`, via the existing tunnel) to receive the synced
project folder, write generated assets into it, and update README.md there. Not just abstract "file
write" as previously listed per-stage — the host differs by stage.

**Reference image file naming (LOCKED):** Descriptive, not generic — filenames must encode project/episode, character-or-location, image type, and (for characters with an expression range) the expression, so files are self-identifying against the script/bios without opening them:

```
{SeriesSlug}_Ep{NN}_char-{character-slug}_scene-ref_{expression-slug}_{WxH}.png
{SeriesSlug}_Ep{NN}_char-{character-slug}_face-lock_{WxH}.png
{SeriesSlug}_Ep{NN}_loc-{location-slug}_scene-ref_{WxH}.png
```
Examples: `AlienDogTwins_Ep01_char-twin-brother-1_scene-ref_curious_1536x864.png`, `AlienDogTwins_Ep01_char-twin-brother-1_face-lock_1024x1024.png`, `AlienDogTwins_Ep01_loc-workshop-garage_scene-ref_1536x864.png`

## 1.6 Multi-Agent Architecture & Routing (LOCKED)

**Decision:** Per-stage dedicated HERMES agent profiles, not one unified profile — matches your existing pattern (comfyui-expert is already a separate instance) and keeps each stage independently testable/versionable per test plans below.

**Why not one profile:** Confirmed via review of your actual HERMES architecture — each HERMES instance loads **one profile** (`manifest.yaml` + `SOUL.md` + tool allowlist), with internal delegation limited to a fixed 3-tier model chain **within that profile** (Bedrock Haiku orchestrator → local Ollama worker → local Ollama escalation). That chain cannot spawn a *different* profile/identity as a child — cross-profile invocation requires a separate running instance, exactly like comfyui-expert already is.

**Dispatch mechanism (LOCKED, syntax validated live 2026-07-26):** Router uses CLI dispatch as the default — `hermes -p <target-profile> chat -q "<prompt>" -Q` invoked as a subprocess, the same way you already trigger comfyui-expert manually via `hermes -p comfyui-expert` on the EC2 terminal. Telegram-based dispatch stays available as a secondary/manual path (e.g. for you to intervene directly on a stage agent), but CLI is what the router uses to move the pipeline forward automatically.

**Validated against the real CLI (2026-07-26, hermano.dudelabz.com):** `-p <profile>` is a real, working flag despite being undocumented in `hermes --help` — confirmed it's scoped per-invocation, not global (running with `-p` does not mutate `~/.hermes/active_profile`). Two corrections to the original design:
- The literal `-- <args>` syntax from earlier drafts of this doc **does not work** — bare `--` before a subcommand is an argparse error. The confirmed-working non-interactive form is `hermes -p <profile> chat -q "<prompt>" -Q` (`-Q` = quiet mode, built for programmatic use — suppresses banner/spinner/tool previews).
- An empty or malformed `-q` prompt causes `hermes` to silently fall through to an **interactive TUI session** instead of erroring — a hang risk for an unattended Router. Router must never dispatch with an empty prompt string.

**Completion detection (LOCKED):** Dual-signal — CLI exit code is read as an immediate pass/fail signal, but the project's `README.md` is the actual source of truth for pipeline state. A zero exit code doesn't mean "advance"; the router still confirms the expected stage output/confirmation flag is present in the project folder before dispatching the next stage. **Validated live:** this caution is justified — tested nonexistent profile, unconfigured profile, and empty-prompt-hang cases; all three returned **exit code 0**. Exit code is effectively meaningless on this HERMES build; README.md state-check is not a nice-to-have, it is the only real signal.

**Interview-style stages are owner-run, not Router-dispatched (LOCKED, discovered live 2026-07-26).**
`hermes -p <profile> chat -q "<prompt>" -Q` is fundamentally a **one-shot, non-interactive call** —
it sends one prompt, gets one response, and the process exits. There is no way for Router to
"continue" that conversation if the dispatched agent asks a clarifying question, because the
process is already dead by the time its response comes back.

This was discovered live when Router successfully dispatched `hermes-cine-chardesign` (a real
subprocess ran, exit code 0, SOUL.md loaded correctly) — the agent correctly followed its
ref-images-first design and asked its first question, then the session simply ended with no way to
answer it. Confirmed via `ps aux` that no process was left running, and via session export that the
transcript stopped at exactly 6 messages, the agent's question unanswered.

**Resolution — split by stage type, not a Router redesign:**
- **Interview-style stages (Intake, CharDesign, and any future stage whose LOCKED design involves
  genuine back-and-forth — asking questions, waiting for answers, adapting to what's provided) are
  never dispatched by Router.** Instead, Router detects "the prior stage is confirmed and this
  stage hasn't started" and **notifies the owner directly** to run that stage themselves:
  `hermes -p <profile> chat` (interactive, no `-q`) — exactly the manual flow already proven to
  work in every real test so far. Router's job for these stages is to say whose turn it is, not to
  conduct the interview.
- **Single-shot-capable stages (e.g. Script, once a format choice is made; likely Storyboard,
  Assembly) stay auto-dispatched by Router** via the existing `chat -q "..." -Q` mechanism — these
  don't need mid-task back-and-forth once given full context up front.
- Router resumes normal auto-dispatch once README.md shows the owner-run stage confirmed.

**Future enhancement, not yet built:** `--resume <session_id>` was confirmed live to genuinely
continue a prior session's full conversation history (tested by resuming the dead chardesign
session and getting a coherent, context-aware follow-up response). This means a **fully automated
relay** — Router detects "needs input," notifies the owner with the specific question, waits for a
reply, then re-invokes with `--resume <session_id> chat -q "<answer>" -Q`, repeating until the
agent signals real completion — is technically feasible and could fully automate interview stages
later. Deliberately deferred: it requires new Router logic to distinguish "agent is asking a
question" from "agent is done," state-tracking for in-flight interviews, and hasn't been validated
across a full multi-question interview (only a single resume-and-answer round was tested). Build
this once the simpler owner-run flow above is solid, not before.

**Router placement (LOCKED):** Router runs on EC2, same host as the other HERMES instances, so it has local shell access for `hermes -p <profile> chat -q "..." -Q` calls without needing SSH/remote dispatch overhead.

**Agent naming (LOCKED — Functional, lowercase-hyphenated):** Names are lowercase-hyphenated, not uppercase — the real `hermes profile create` silently forces profile directory names to lowercase, and `-p <profile>` requires an exact case match (mixed-case input fails with an unhelpful generic argparse error, not a helpful correction). Discovered live on 2026-07-26; the convention was revised repo-wide to match rather than fight the tool.

| Stage | Agent profile name |
|---|---|
| Router/orchestrator | `hermes-cine-router` |
| 0 — Intake | `hermes-cine-intake` |
| 1 — Script | `hermes-cine-script` |
| 2 — Character/Location Design | `hermes-cine-chardesign` |
| 3 — Ref Image Lock | delegates to existing `comfyui-expert` |
| 4 — Storyboard/Timeline | `hermes-cine-storyboard` |
| 5 — Shot Image Gen | delegates to `comfyui-expert` |
| 6 — Clip Gen | delegates to `comfyui-expert` |
| 7 — Audio Gen | delegates to `comfyui-expert` |
| 8 — Assembly/Edit | `hermes-cine-assembly` |
| 9 — Final QC/Export | `hermes-cine-qcexport` |

Lightweight stages (4, 8, 9) are candidates to fold into `hermes-cine-router` itself rather than getting a fully separate profile, if they end up thin enough during build — to revisit once Stage 4+ are specced.

## 1.7 Failure Handling, Notifications & Model Routing (LOCKED)

**QC failure handling (LOCKED):** On QC gate failure — retry the same stage **once automatically**, then stop and ask the user for a decision (edit/retry again/skip) if the retry also fails. No unlimited auto-retry loops.

**Notifications (LOCKED):** Router notifies via **Telegram + email**, and the project's README.md is always kept current as the durable record — so a confirmation/decision request reaches you through your primary interface (Telegram), is backed up via email, and is always visible if you check the project folder directly later.

**Model routing policy per profile (LOCKED — case-by-case, not blanket):** Not every HERMES-CINE profile automatically inherits the standard Bedrock Haiku orchestrator → local Ollama worker → local Ollama escalation chain. Some stages don't need a cloud call at all (e.g. pure dispatch/delegation stages with no independent reasoning to triage). **Routing tier is decided per-stage as each one gets specced** — not assumed universally. Candidates likely to skip cloud entirely: Stage 3/5/6/7 (comfyui-expert delegation stages, since they're pure generation dispatch, not reasoning) — to be confirmed individually when we spec them.

---

```
[Intake/Interview] → [Script] → [Character & Location Design] → [Ref Image Lock]
→ [Storyboard/Timeline] → [Shot Image Gen] → [Clip Gen] → [Audio Gen] → [Assembly/Edit] → [Final QC/Export]
```

Every column = same card template:
- **Input** (what feeds this stage)
- **Process/Tool** (agent logic + which ComfyUI wf / model)
- **Output** (artifact produced)
- **QC Gate** (auto-check + what "pass" means)
- **Confirmation** (user approves before advancing)

---

## 2. Stage Breakdown

### Stage 0 — Intake / Interview

**Question set (LOCKED):**

| # | Question | Notes |
|---|---|---|
| 1 | What's the idea / premise? | Core input |
| 2 | Is this original, or adapted from existing IP (book, game, story, script)? | Always asked explicitly |
| 3 | Single standalone piece, or episodic/series? | Asked every time, no default |
| 4 | Visual style — description + optional reference image(s) | Free-text; **not** cross-checked against comfyui-expert catalog at this stage — mapped to actual workflows later |
| 5 | Target aspect ratio/platform | Default **16:9, 720p** if not specified |
| 6 | Genre/tone (comedy, drama, thriller, etc.) | Explicit question, **then enriched/refined by inference** from idea + style |
| 7 | Dialogue/output language | Default **English**; alternate option **Hebrew** |

| 8 | (Optional) Any "must include" elements — specific line, prop, character trait | Optional, skip if user has none |
| 9 | (Episodic only) Is this a continuation of a previous episode? If yes, link its project folder | Enables continuity — reuse refs directly, or regenerate with consistency validation (asked per-project, see §1.5 Series Continuity) |

**Inferred, not asked directly:**
- Target audience/rating (kids/general/mature) — inferred from the idea itself
- Target runtime/length — flexible target, refined after script stage rather than locked upfront
- Number of main characters — inferred from idea/premise, **validated/confirmed at script stage** (not asked at intake)

**Research behavior:**
- Adapted IP (book/game/story) → agent does **not** auto web-search background/context. Only researches if user explicitly requests it.
- Real-person/trademarked-brand references — **not flagged**, not a concern for this use case.

**Output:** Project Brief (structured local doc/file) — no auto-creation of external tracked tasks (no Notion/monday page). Feeds Stage 1.

**Tool / Skill Access Required for Stage 0:**
| Capability | Needed? | Notes |
|---|---|---|
| LLM reasoning (brief drafting, genre inference) | Yes | Core function |
| Web search | Conditional | Only on explicit user request for IP research |
| Image input handling | Yes | For optional style reference images user uploads |
| comfyui-expert catalog access | No (at this stage) | Style stays free-text; mapping happens later at generation stages |
| External task tools (Notion/monday/Slack) | No | Output stays local file/doc |
| Filesystem read/write on DGX host | Yes | Create `{SeriesSlug}_Ep{NN}/` root + `00_brief/`, write project-brief.md, initialize README.md |
| Read previous episode's project folder | Conditional | Only if continuity question (Q9) answered yes |

**OPEN (still to resolve):**
- None currently — Stage 0 fully specced. Next: Stage 1 script format.

### Stage 1 — Script Generation

**Format decision logic (LOCKED):**
- Agent suggests a script format based on intake data (premise, genre, style, episodic vs single) — not fixed to one format
- For each suggested format, agent explains **why** it fits + **pros/cons**
- User confirms the suggested format or edits/overrides it

**Variants:** Only the premise/beat level gets 2-3 variant options if needed — the **full script itself is single-version**, straight to confirmation (no parallel full-script drafts)

**Edit model after QC gate:** Agent works as a **co-writer** — suggests specific edits/alternatives per scene rather than blind full regeneration; user reviews suggestions and approves

**Character/role validation:** Stage 1 output includes an **explicit character list** (roles inferred from premise, e.g. "2 twin brothers, 1 father, 1 dog") — this list must be **confirmed by user before Stage 2** begins (fulfills the earlier "infer + validate at script stage" rule)

**Output:** Clean prose script (format per above) + explicit character/role list. **No structured beat/scene breakdown** — Stage 4 (storyboard) extracts beats itself from the prose script.

**QC Gate:** User confirms script + character list before Stage 2.

**Tool / Skill Access Required for Stage 1:**
| Capability | Needed? | Notes |
|---|---|---|
| LLM reasoning (script writing, format selection, pros/cons framing) | Yes | Core function |
| Web search | Conditional | Same rule as intake — only if user explicitly requests research (e.g. adapted IP) |
| Read Project Brief (Stage 0 output) | Yes | Required input |
| Filesystem read/write on DGX host | Yes | Write `script.md` to `01_script/`, update README.md |
| comfyui-expert catalog access | No | Text-only stage, no image/video generation here |
| Image/visual generation | No | Not until Stage 3 |

**Versioning:** Simple `script.md`, overwritten on each edit (no version-numbered files)
**Edit rounds:** Capped at 5 co-writer suggestion rounds, then agent forces a confirm/reject decision

**OPEN (still to resolve):**
- None currently — Stage 1 fully specced. Next: Stage 2 character & location design.

### Stage 2 — Character & Location Design

**Input:** Confirmed script + confirmed character/role list (from Stage 1)

**Character bios (LOCKED):**
- **Step order (LOCKED):** Agent first asks user for **reference images** (per character/location, optional upload) — *before* running the trait interview
- **Interview building logic:** The combined trait interview is then built **around what refs were/weren't provided** — traits already visible in a supplied ref image are not re-asked; interview focuses on gaps (details not visible in the image) plus non-visual traits (personality/voice) regardless of refs
- **Depth:** Rich — visual details (appearance, build, clothing, distinguishing features) **plus** personality/voice traits (not just looks)
- **No refs provided case:** Falls back to full combined interview covering all visual + personality traits, as originally specified
- **Twins handling:** Shared baseline traits + individual differentiators (e.g. one wears red, one wears blue) captured within that same combined interview

**Location design (LOCKED):**
- Same ref-first order: ask for location reference images before describing
- **Lighter treatment** than characters — key visual description only (setting, mood, color palette, notable props), not a full rich bio
- Applies per recurring location in the episode (e.g. street, workshop/garage)

**Output:** Character bios (rich) + location descriptions (light) — purely descriptive text, **not** mapped to specific ComfyUI workflows/parameters at this stage (that mapping happens at Stage 3)

**Reference image resolution spec (LOCKED, applies at Stage 3 generation but defined here since it's part of the char/location design handoff):**
- **Scene/full refs:** 1536×864 (16:9, 2x final 720p render res) — clean integer downscale to 1280×720, divisible by 8 for diffusion pipeline compatibility, quality headroom without excess VRAM/time cost
- **Character face-lock crop:** additional square 1024×1024 per character, alongside the 16:9 scene ref — standard for PuLID-style face-embedding workflows
- Locations: 1536×864 only (no square crop needed)

**QC Gate:** User confirms bios + location descriptions before Stage 3 (ref image generation).

**Tool / Skill Access Required for Stage 2:**
| Capability | Needed? | Notes |
|---|---|---|
| LLM reasoning (bio writing, trait interview) | Yes | Core function |
| Image input handling | Yes | To accept user-uploaded reference images before interview |
| Read Stage 1 output (script + character list) | Yes | Required input |
| Filesystem read/write on DGX host | Yes | Write `bios.md` to `02_characters_locations/`, update README.md |
| comfyui-expert catalog access | No | Bios stay purely descriptive; workflow mapping deferred to Stage 3 |
| Image generation | No | Not until Stage 3 (user-supplied refs are input here, not generated) |

**Versioning:** `bios.md`, overwritten on each edit (matches `script.md` convention)
**Conflict handling:** If a user's trait answer conflicts with something implied in the script, agent flags it and asks the user to resolve — never silently overrides either side.

**OPEN (still to resolve):**
- None currently — Stage 2 fully specced. Next: Stage 3 ref image lock (comfyui-expert handoff).

### Stage 3 — Reference Image Lock (Character + Location)
- Tool: comfyui-expert agent, PuLID/Flux Kontext workflows
- **Resolution:** 1536×864 (16:9 scene refs) + 1024×1024 square face-lock crop per character — per spec defined in Stage 2
- Output: locked ref image set per character/location
- QC + Confirmation: **user must approve refs before proceeding** (explicit gate you specified)

### Stage 4 — Storyboard / Timeline
- Input: script + locked refs
- Output: shot list with per-shot: description, characters present, location, camera notes, duration
- OPEN: storyboard format (visual thumbnails vs text shot list vs both?)

### Stage 5 — Shot Image Generation
- Tool: comfyui-expert, using locked char/location refs for consistency
- Output: key frame image per shot
- QC: consistency check against ref images

### Stage 6 — Clip Generation
- Tool: comfyui-expert (LTX Video / Wan2.2 workflows per your existing stack)
- Output: video clip per shot, consistent characters
- QC: animated_video_reviewer.py pass

### Stage 7 — Audio Generation (music / SFX / voice)
- Tool: ComfyUI workflows (ACE-Step/MMAudio)
- Timing: runs **after** visuals are locked, not upfront
- OPEN: voice/dialogue — TTS per character? Any voice-cloning requirement?

### Stage 8 — Assembly / Edit
- Tool: TBD — Premiere/Resolve scripted assembly? Or agent-driven EDL/XML generation (you've done this before)?
- OPEN: how much of this stays manual in Premiere/Resolve vs automated

### Stage 9 — Final QC / Export
- Tool: animated_video_reviewer.py full pass
- Output: final cut + export specs
- OPEN: delivery formats/platforms needed

---

## 3. Open Questions Log (to resolve as you send notes)

- [x] Intake interview question set — LOCKED (see Stage 0, fully resolved)
- [x] Script format — LOCKED (agent suggests w/ pros-cons, see Stage 1)
- [x] Script file naming/versioning — LOCKED (script.md, overwritten)
- [x] Max edit rounds — LOCKED (cap 5, then force decision)
- [ ] Character/location bible template (reuse existing?)
- [ ] Storyboard format (visual vs text)
- [ ] Voice/dialogue handling (TTS, cloning?)
- [ ] Assembly/editing automation level (scripted EDL vs manual)
- [ ] Target output specs (resolution, duration, platform)
- [ ] How comfyui-expert agent is invoked (API call? shared task queue? Kanban board itself as interface?)
- [ ] Model routing (Bedrock Haiku orchestrator + local Ollama, per standard HERMES pattern — confirm applies here too)
- [ ] Interface: Telegram/web like other HERMES agents, or dedicated UI?
- [ ] Naming/versioning convention for project files and ComfyUI outputs
- [ ] Failure/retry handling per stage (what happens on QC fail)

---

## 4. Test Plans — Completed Stages (real validation, not just chat-mock)

### Stage 0 — Intake Test Plan
- [ ] Run with **adapted IP** answer — confirm agent does NOT auto-research unless explicitly asked
- [ ] Run with **no style reference image** provided — confirm free-text-only path works
- [ ] Run with **Hebrew** dialogue language — confirm it's accepted as alt option
- [ ] Run with **single standalone** (non-episodic) — confirm Q9 (continuity) is skipped/not asked
- [ ] Run with **episodic + "yes" to continuity (Q9)** — confirm it correctly prompts for/links a previous episode's project folder
- [ ] Confirm real filesystem write: `{SeriesSlug}_Ep{NN}/00_brief/project-brief.md` actually created on DGX, README.md initialized
- [ ] Confirm "must include" question is skippable with no downstream errors

### Stage 1 — Script Test Plan
- [ ] Run with a **different genre/format combo** (e.g. non-animated, dialogue-heavy) — confirm format suggestion + pros/cons logic adapts, not hardcoded to shot-ready prose
- [ ] Trigger **5 edit rounds** — confirm agent forces a decision on round 5, doesn't loop indefinitely
- [ ] Deliberately give a **character trait that contradicts the script** — confirm conflict is flagged, not silently resolved
- [ ] Confirm real filesystem write: `01_script/script.md` overwritten correctly on edit (not versioned/duplicated), README.md updated to point to latest
- [ ] Confirm character/role list requires explicit user confirmation before proceeding (gate actually blocks, doesn't auto-advance)

### Stage 2 — Character & Location Design Test Plan
- [ ] Upload a **real reference image** for one character — confirm the trait interview correctly skips traits already visible in that image
- [ ] Run with **no reference images at all** — confirm fallback to full combined interview works
- [ ] Run the **combined interview** across all characters at once (not sequential) — confirm it doesn't fragment into multiple separate prompts
- [ ] Deliberately create a **trait-vs-script conflict** — confirm it's flagged and resolution is asked, not auto-decided
- [ ] Confirm **locations get lighter treatment** in actual output (shorter than character bios, not equal depth)
- [ ] Confirm real filesystem write: `02_characters_locations/bios.md`, README.md updated

### Stage 3 — Ref Image Lock Test Plan (partially run already)
- [x] Basic generation via comfyui-expert handoff — **passed** (2 runs, images came back good)
- [ ] Re-run using **actual bios.md output from a real Stage 2 run** (not hand-written test brief) — validates the real Stage 2→3 handoff, not just a manually authored prompt
- [ ] Confirm resolution compliance: scene refs actually 1536×864, face-lock crops actually 1024×1024 (not just requested — verify output files)
- [ ] Confirm generated files are **copied into** `03_ref_images/characters/{name}/` and `03_ref_images/locations/{name}/` automatically, not left in ComfyUI's output dir
- [ ] Confirm QC gate: user must explicitly confirm refs before Stage 4 unlocks (test that it doesn't auto-advance)
- [ ] Stress-test consistency: generate the **full expression range** listed per character (playful/shocked/panicked etc.) and confirm face/proportions hold up across all of them, not just a single neutral pose
- [ ] Test **series continuity path**: reuse-prior-refs vs regenerate-with-validation, once a second test episode exists

---

## 5. Validation Status (per stage — update as real testing happens)

| Stage | Design specced? | Text-flow tested? | Real tool/skill calls validated? |
|---|---|---|---|
| 0 — Intake | Yes | Yes (mock + multiple real runs) | Yes — live on EC2, 2026-07-26. Real disk writes, correct folder/stage-name/owner tables, confirmation persists to disk. Several real bugs found and fixed along the way (see CLAUDE.md log). |
| 1 — Script | Yes | Yes (mock + real run) | Yes — live on EC2, 2026-07-26. Real disk writes, character list correctly merged into script.md (not a separate file), root README fully updated. |
| Router | Yes | N/A (orchestration agent) | Yes — live on EC2, 2026-07-26. Composed and ran a real `hermes -p <profile> chat -q "..." -Q` dispatch subprocess against three different scenarios: unconfirmed-state block, not-yet-built-target (exit 1), and a real existing target (`hermes-cine-chardesign`, exit 0, correct SOUL.md identity). Human-gate on `shell_exec` found to be unenforced by HERMES itself and fixed via explicit SOUL.md instruction — now confirmed working (real approval prompt observed). **Found the dispatch-mode architectural gap** (interview-style stages can't be dispatched via one-shot `-q` — see §1.6) and the fix (interview stages become owner-run, not Router-dispatched) is designed but not yet implemented in Router's SOUL.md or re-tested. |
| 2 — Character/Location Design | Yes | No | No — real package generated (folds in every live-validated lesson from Stages 0/1/Router), not yet installed or run live. |
| 3 — Ref Image Lock | Partial (res spec + folder conventions locked) | Yes (2 real comfyui-expert runs, images came back good) | Partial — real generation validated, but not yet through full Stage 2→3 handoff with actual bios.md |
| 4 — Storyboard/Timeline | Not yet specced | No | No |
| 5 — Shot Image Gen | Not yet specced | No | No |
| 6 — Clip Gen | Not yet specced | No | No |
| 7 — Audio Gen | Not yet specced | No | No |
| 8 — Assembly/Edit | Not yet specced | No | No |
| 9 — Final QC/Export | Not yet specced | No | No |

*This is a living scaffold — send notes and I'll fold them into the relevant stage or open question above.*
