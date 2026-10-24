# How the current workflow runs — 3.1.0

What happens, in order, after you send the first prompt. Every step names the file that governs it and the artefact it produces. Verified by running the helpers in this package; the render/audio steps need `node`, `ffmpeg`, `numpy` and `scipy`.

**Shape of it:** seven stages, **two hard gates** (`check_assets.py`, `finalize.py`) that a command enforces, and **one human gate** (treatment/script approval) that the agent may skip only when your prompt already authorized the work.

---

## The 60-second version

| # | Stage | Agent does | Command | Produces | Blocks if… |
|---|---|---|---|---|---|
| 0 | Load | opens `video-director`, then `references/current-workflow.md`; loads upstream skills as needed | — | — | — |
| 1 | Understand | purpose, audience, prior knowledge, evidence, scope, constraints; records *your* requirements with a source | `init_video.py` | `video/plan.json`, `assets/brief.json`, `assets/manifest.json`, `assets/{source,review}/` | — |
| 2 | Treat | writes `narration[]` and `shots[]` **independently**, plus `brief`, `treatment`, `voice` | `plan_docs.py` | `video/PLAN.md`, `video/narration.txt`, `video/tts-prompt.txt` | plan is structurally incomplete |
| 3 | Look | derives `art_direction` (setting, materials, lighting, context, tone, production method, palette roles, typography roles, reference roles) | — | plan fields + style-frame tests | palette/typography/references missing reasons |
| 4 | Prepare | creates project artwork, proof stills, motion proofs and review notes | `check_assets.py --seal`, then `check_assets.py` | `assets/source/…`, `assets/review/…`, `video/asset-check.json` | **GATE 1** — any missing/unsigned asset |
| 5 | Assemble | Studio + composition; registers your voice take; aligns actual words; authors shot frames, captions, sound events | `after_vo.py`, `auto_sfx.py` | `src/*`, `public/*`, `video/cues.json`, `src/sfxCues.ts`, `video/cue-sheet.md`, `video/preview-mix.wav`, `src/data/timing.json` | take changed, event outside its shot, no `total_frames` |
| 6 | Review | looks at actual pictures and normal-speed sequences; runs engineering checks separately | `verify_export.py`, `audit_audio.py` | `video/export-check.json`, filled QA record | decode/metadata/voice-hash failure |
| 7 | Deliver | masters audio, verifies the encoded file, hands over MP4 + SRT | `finalize.py` | `out/final.mp4` | **GATE 2** — delivery contract or asset re-check fails |

---

## Stage 0 — How the agent starts

Your prompt rarely names a skill. What happens:

1. The installed skill list shows twelve entries. The one whose description matches "direct a film from the current brief…" is **`video-director`**, and it is the entry point for anything that ends in a finished video.
2. It opens `skills/video-director/references/current-workflow.md` first. That file, not the prompt, defines authority: **your requirements** (with a source), **technical requirements**, and **project decisions** are kept separate, and your instructions outrank both.
3. Implementation guidance is pulled *on demand* from the upstream skills — `remotion-create` to scaffold, `remotion-studio` to preview, `remotion-markup` for JSX/animation/effects, `remotion-captions` for the caption track, `remotion-render` for export, `remotion-docs` for current APIs, `remotion-interactivity` for Studio-editable structure, plus `maps`/`multimedia` only if the film needs them.

Nothing is loaded as a checklist. The workflow is seven stages, not sixteen documents.

---

## Stage 1 — Understand, and open the project

The agent establishes purpose, audience and what they already know, factual scope, delivery constraints, and any unresolved ambiguity. Anything it *infers* rather than being told is recorded as an assumption, not as your preference. That distinction is enforced later: `requirements[].source` must be `{kind, reference}` and a CLI value becomes an `assumption` entry, so an agent cannot quietly attribute its own aesthetic choice to you.

```bash
python3 <install>/video-director/scripts/init_video.py /path/to/project \
  --profile channel --size 1080x1920 --fps 30
```

Output (real):

```
Created schema-3 brief/treatment records with independent empty narration and shots.
Next: establish sourced requirements and project decisions, then prepare artwork before final assembly.
```

Files created — note how little arrives:

```
project/
  video/plan.json          # schema 3, everything creative is empty
  assets/brief.json        # schema 2, empty review fields
  assets/manifest.json     # stage: "draft", assets: []
  assets/source/           # empty
  assets/review/           # empty
```

The starter is deliberately hollow: `title ""`, `theme null`, `width/height/fps null` unless you passed them, `narration []`, `shots []`, `palette.roles {}`, `typography.roles {}`, `captions {enabled: null, style: null}`. **No palette, font, narrator, duration, aspect ratio, scenery or sound is injected.** If `--profile channel` is selected explicitly, it copies four *sourced user preferences* (paper medium, lower smooth captions, purposeful display text, an engaging opening) with the request text that justifies each. `theme` becomes `cut-paper` as a recorded medium preference — no theme or font files ship, so the paper look is built by the project.

`init_video.py` never overwrites an existing plan. Re-running prints `Existing plan preserved; edit its sourced requirements/decisions explicitly.`

---

## Stage 2 — Script and shots, written as two independent things

This is the core architectural rule of 3.0/3.1, and it is what the file schema enforces:

- `narration: [{id, text, proof: []}]` — exact spoken paragraphs.
- `shots: [{id, purpose, visual, action, framing, camera, connection_to_next, spoken_cues: [], events: []}]` — illustrated shots.

Neither derives from the other. A paragraph may span several shots; a shot may span several paragraphs; `spoken_cues` may point at any narration ids. Shot boundaries are an **editorial** decision, taken later, from the actual take — never inferred from sentence ends or pauses.

```bash
python3 <install>/video-director/scripts/plan_docs.py video/plan.json
```

This validates structure and writes a reviewable handoff:

```
video/PLAN.md          # canvas, brief, sourced requirements, assumptions, treatment,
                       # art direction, full script, shot table, voice prompt
video/narration.txt    # exact text to be recorded
video/tts-prompt.txt   # voice direction + "Read only the transcript, exactly as written."
```

On an untouched starter it refuses and lists every hole (real output, trimmed):

```
title is unresolved
brief.purpose is unresolved
brief.audience is unresolved
shots must be a nonempty array
treatment.approach is unresolved
art_direction.setting needs a value and reason
art_direction.palette needs explicit project roles and reason
enabled captions require explicit project style
```

**Human gate — the only one in the workflow.** If no plan/script approval exists yet, the agent presents the treatment, script, shot list and voice prompt and waits. If your prompt already authorized the film, authorization carries forward and it continues without asking again — that is the documented behaviour ("Existing authorization avoids renewed gates"). Asset review, caption review and audio handoff afterwards add **no** new gates.

---

## Stage 3 — Art direction (a decision record, not a template)

The agent fills `treatment.art_direction`, each with a **reason**, not just a value:

- `setting`, `materials`, `lighting`, `context`, `tone`, `production_method` — each `{value, reason}`;
- `palette: {roles: {role: colour}, reason, evidence}` — colours derived from the scene, not from the topic label;
- `typography: {roles: {role: font}, reason}` — a font chosen for this project, with its licence;
- `references: [{uri, roles: […], adopt, limits}]` — a reference may only contribute the roles you named. A material reference does not hand over a palette; a named medium does not dictate cast or environment.

Style frames are made and looked at here, at delivery size, before full artwork is committed. The agent is told to compare options only when a real uncertainty exists — no mandatory variation quota.

---

## Stage 4 — Project artwork, and GATE 1

Order is enforced by the tooling, not just prose: *plan → approval → all artwork → inspect proofs → seal → assemble*.

1. Fill `assets/brief.json`: `reference_review` and `visual_direction` (each ≥ 20 chars), plus `requirements[]` — `{key, purpose, moving, shot_ids}`.
2. Create layered sources under `assets/source/`, proofs in `assets/review/`, and for `moving: true` an actual motion clip landing in `motion_files`.
3. Write `review_notes` (≥ 40 chars) describing what was actually looked at — colour/shape/scale/coverage and motion/contact limits.
4. `assets/manifest.json`: `stage: "prepared-before-assembly"`, and one entry per requirement with `origin` ∈ `created-for-project | user-supplied | licensed-for-project` and `status: "ready"`.

```bash
python3 <install>/video-director/scripts/check_assets.py --project . --seal   # records hashes
python3 <install>/video-director/scripts/check_assets.py --project .          # verifies them
```

**What the gate actually rejects** (all real messages):

| Failure | Message |
|---|---|
| no artwork yet | `asset preparation/review is incomplete` |
| incomplete brief | `complete the asset brief: visual_direction` |
| a brief requirement with no matching asset | `unprepared requirements: hero, mechanism` |
| flat image where animation was claimed | `hero: moving artwork needs a motion proof` |
| proof missing after an earlier pass | invalidates the previous `PASS`; report is reset to `NOT_CHECKED` first |
| edited source without re-sealing | `hero: changed/unsealed files; review changes, then deliberately run --seal` |
| art direction edited after sealing | `art-direction context changed/unsealed; inspect affected artwork, then deliberately run --seal` |
| absolute path / symlink / `..` | `asset path outside allowed project directory`, `asset symlinks/escaping paths are not portable` |
| runtime copy edited by hand | `hero: runtime copy differs from source` |
| an imported helper file not inventoried | `unrecorded source dependency: assets/source/rig.ts` |

Result is written to `video/asset-check.json` with the scope spelled out: *"Does not certify visual quality, originality, playback or production order."* The agent is explicitly told not to call it creative approval.

---

## Stage 5 — Assemble

Two things happen in parallel here, which is why the schema keeps them separate.

**A. The voice take you supplied is registered, never modified.**

```bash
python3 <install>/video-director/scripts/after_vo.py --project . --vo public/vo.wav
```

- refuses anything outside `public/`: `Copy the original take unchanged under public/ first.`
- refuses pre-schema-3 plans: `schema 3 required; older scene-based plans keep their existing exports.`
- writes `vo{file, duration_s, original_sha256, timing_verified: false}` and `src/data/timing.json`
- prints: *"no shot boundaries, effects or visual tail were inferred"*

Then the agent compares the real words against the approved script, saves source-relative timestamps, sets `vo.timing_method`, and only after review sets `vo.timing_verified: true`. The audio file itself is never re-encoded — the SHA-256 is re-checked at the end and the export verifier compares it again.

**B. The picture is built in Studio, from the prepared assets.**

`npx remotion studio` is opened early and kept open so you can watch and steer. Shot `start_frame`/`duration_frames`, `total_frames`, camera moves, transitions and captions are authored by hand against the aligned words — deliberately *not* auto-derived. Captions are an exact-speech JSON track with an explicit project style; `plan_docs.py` rejects captions without one.

**C. Sound, only where the agent asked for it.**

Each event is authored on the shot as `{t, file, volume, note}` with the file already in `public/sfx/`. Then:

```bash
python3 <install>/video-director/scripts/auto_sfx.py video/plan.json --project .
```

It emits exactly those events — `video/cues.json`, `src/sfxCues.ts`, a `video/cue-sheet.md`, and `video/preview-mix.wav` — and refuses anything it cannot see: `missing project sound: …`, `sound event lies outside its shot`, `author total_frames before emitting a mix`, `selected music needs an explicit project volume`. **No events means no effects**; there is no theme palette, variant rotation or "one sound per motion" rule left anywhere.

---

## Stage 6 — Review, split in two

The package is emphatic that these are different kinds of evidence, and the QA template has one section for each (`Creative status` and `Engineering status` both start at `NOT REVIEWED` / `NOT CHECKED`):

- **Creative** — actual frames opened at viewing size; the whole cut watched at normal speed; joins, opening and ending judged; listening if audio exists. Timecodes and revisions recorded. If playback or listening was not possible, that is recorded as incomplete rather than implied.
- **Engineering** — `verify_export.py` (dimensions, fps, frame count, duration, full decode, audio stream, 48 kHz, original-voice SHA-256) and `audit_audio.py` (LUFS, true peak, clipping). A clean exit here does **not** clear creative review, and a beautiful frame does not clear engineering.

---

## Stage 7 — Deliver, and GATE 2

```bash
python3 <install>/video-director/scripts/finalize.py --project . --comp Main \
  --audio-master video/preview-mix.wav --out out/final.mp4
```

In order, it: re-validates the plan in **render mode** → re-checks the sealed assets → `npx remotion render` to `out/raw.mp4` → two-pass `loudnorm` to your target (−14 LUFS / −1.5 dBTP by default) → `audit_audio.py --mix` → `verify_export.py` → `out/final.mp4` + `video/export-check.json`.

Render mode adds exactly the two things that cannot exist earlier: every shot needs real `start_frame`/`duration_frames` within `total_frames`, and a narrated film needs `vo.file` **plus** `timing_verified: true` **plus** `timing_method`:

```
Plan delivery contract failed: narrated delivery needs supplied voice and verified actual-speech alignment
Asset readiness failed before render/master: art-direction context changed/unsealed; …
audio master duration differs from picture by over 60ms; fix the timeline before mastering
--silent requires a deliberate non-narrated source with no audio; no supplied sound is removed
```

Then the MP4 and a useful SRT are handed over. Publishing or pushing happens only if you authorized it.

---

## What a finished project looks like

```
project/
  video/plan.json          # the whole decision record: requirements + sources, brief,
                           # treatment, art_direction, narration, shots, timing, vo, sound
  video/PLAN.md            # reviewable treatment/script/shot handoff
  video/narration.txt      # exact recorded text
  video/tts-prompt.txt     # voice direction
  video/cues.json          # authored sound events
  video/cue-sheet.md       # seconds / file / level / reason
  video/preview-mix.wav    # verified mix used as the audio master
  video/asset-check.json   # asset gate report
  video/export-check.json  # encoded-file checks
  qa-report.md             # creative + engineering evidence, filled from the template
  assets/brief.json        # what artwork was required, and why
  assets/manifest.json     # sealed sources, proofs, notes, hashes, runtime copies
  assets/source/…          # layered, editable artwork
  assets/review/…          # proof stills, motion clips, observations
  public/fonts|sfx|vo…     # project media, unchanged originals
  src/…                    # composition, components, sfxCues.ts, data/timing.json
  out/final.mp4            # deliverable
```

---

## Where you are actually asked something

1. **Plan approval** — once, and only if your prompt did not already authorize the film. You get treatment + script + shots + voice prompt in `video/PLAN.md`.
2. **A genuine ambiguity** — only when context and evidence cannot resolve it (e.g. duration or language is genuinely unclear).
3. **A conflict you created** — e.g. the supplied take is 40 s longer than the requested duration; the workflow forbids silently cutting or retiming it.

Everything else — art direction, style frames, caption styling, sound design, mix, QA — is the agent's job, recorded in the project, with no extra permission prompts.

---

## Rules the tooling will not let it break

| Rule | Enforced by |
|---|---|
| Your requirements carry a source; the agent's taste does not | `plan_docs.py` (`source.kind` + `reference`, `scope`) |
| Script and shots stay independent | `plan_docs.py` (schema 3; `scenes` rejected) |
| Artwork exists and was actually inspected before assembly | `check_assets.py` (sources, picture proofs, motion proofs, notes ≥ 40 chars) |
| Changing the art direction invalidates the asset review | `check_assets.py` (context hash) |
| Your voice is never altered | `after_vo.py` + `verify_export.py` (SHA-256 before/after) |
| No shot timing invented from speech pauses | schema 3 + the removed legacy estimator |
| No sound invented | `auto_sfx.py` (only authored events) |
| Captions carry explicit project styling | `plan_docs.py` + `SmoothPaperCaptions.tsx` (throws without a style) |
| The delivered file is decodable and matches the plan | `finalize.py` → `verify_export.py` |
| No reusable visual library, font pack or sound palette ships | `validate_bundle.py` (release checks) |

---

## Environment requirements

Python 3.9+; `node` 20.19+/22.13+/24+ with `npx` for Remotion render and Studio; `ffmpeg` and `ffprobe` for the take, loudness and verification steps; `numpy` + `scipy` for `mix_preview.py`, `audit_audio.py` and `finalize.py`'s audio path. A run without audio (`--silent`) skips the mix and loudness work; the picture path still needs node and ffmpeg.
