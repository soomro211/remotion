# Remotion Video Studio 3.0.0 — Complete Skill & Workflow Audit

**Audited artifact:** `remotion-video-studio-final.zip`
SHA-256 `bdfc3ff6e0364c816c015c8d772b5f45129ad59103dd5f4f9d8b27e51b173d92` (matches the shipped `.sha256`)
**Contents:** 184 files · 2,001,900 bytes uncompressed · 934,442 bytes compressed · 16 skills · 99 skill Markdown files · 4 Python tools · 34 test functions
**Date of audit:** 2026-10-24
**Nothing was modified or deleted.** Every recommendation below is a proposal with the exact paths and the exact bookkeeping a change would require.

---

## 0. Method

I extracted the ZIP and treated it as the single source of truth (`MANIFEST.json` verifies it is internally consistent, so nothing was lost in packaging).

1. **Full inventory** — listed every one of the 184 files with size, then read every Markdown file (all 99 under `skills/`, plus all 10 root documents).
2. **Dependency graph** — parsed every Markdown link `](…)` and resolved it on disk, then walked the graph from all 16 `SKILL.md` entry points to find files that no entry point can reach.
3. **Name-level reachability** — because this package also uses backtick references (`` `references/render-stability.md` ``) that are not links, I re-ran reachability on *plain filename mentions* anywhere in the bundle. A file that fails **both** tests is unreachable in practice.
4. **Code-level reachability** — for all 12 skill Python scripts, the 4 maintainer tools, the 5 test modules and every `.ts/.tsx/.mjs` helper, I checked whether any other script imports or shells out to them, and whether any document mentions them.
5. **Duplication analysis** — hashed every file (byte-identical detection) and computed token-set Jaccard similarity across all 99 Markdown files to find paraphrased duplicates.
6. **Version archaeology** — grepped for identifiers that belong to earlier releases (`assets/library`, `catalog`, `person rig`, `CameraRig`, `y=1346`, `scene_ids`, `remotion-saas`, `--no-sfx`, `asset_catalog_entries`, `story_asset_packages`) to separate 3.0.0 material from carried-over material.
7. **Ran the bundled verification** — `python3 tools/validate_bundle.py` → PASS (16 skills, 184 files); `python3 tools/audit_references.py` → PASS (99 files); `python3 -m unittest discover -s tests -v` → 24 tests collected, 3 errors, 1 class skipped.

**Environment caveat:** this sandbox has no `ffmpeg`, `ffprobe`, `node` or `numpy`. The 3 test errors and the `PipelineRegression` class skip are caused by that, not by the package. All conclusions about *file usage* are independent of the environment.

---

## 1. Executive summary

| Metric | Value |
|---|---|
| Files in the bundle | 184 |
| Skill Markdown files | 99 |
| Markdown files with **no inbound link and no textual mention anywhere** | **13** |
| Skills with **zero** inbound links (`remotion-themes`, `remotion-sound-design`) | **2** |
| Skills missing from the `remotion-best-practices` router | **4** of 16 |
| Bytes that are two font files nothing ever names (`Bricolage.ttf`, `Caveat.ttf`) | **812 KB raw / 470 KB zipped = 50.4 % of the download** |
| Bytes that are the optional maps skill | **491 KB raw / 165 KB zipped = 17.6 %** |
| Bytes that are the actual directed-film workflow (director, script, markup, captions, interactivity, create, studio, render, router) | **12.3 % of the download** |
| Byte-identical duplicate files | 2 groups (11 × `remotion-icon.svg`, 3 × `render-stability.md`) |
| Files still carrying previous-version material | **9** distinct items (§2) |
| Verification claims that do not hold as stated | **3** (§6) |

**The headline:** the package's own documentation says "All 16 resources are available" and "99 skill Markdown references pass routing". Both are literally true and both are misleading. Four of the sixteen skills cannot be reached from the router, two of them cannot be reached from *any* document, thirteen reference files are unreachable by any route, and the audit that certifies routing passes because all 99 files contain the same copy-pasted `current-workflow.md` link line — it is a tautology, not a reachability test.

**The biggest single finding:** `remotion-themes` — the skill that owns art direction and the Cut Paper theme — is one of the two skills with zero inbound links, and its 1.05 MB of bundled fonts are named in no instruction anywhere. Over half the download is a resource the agent is never told exists.

---

## 2. Stale material carried over from previous versions

These are not style opinions. Each item is text or code that describes behaviour that 3.0.0 explicitly removed, and in two cases it **contradicts** the current contract.

### 2.1 `PLAN.md` — a v1.5.0 plan shipped at the root  *(delete)*

The file is titled `# Universal library and workflow update — 1.5.0` and instructs:

> "Add approximately 70 distinct assets/motion tools … Paginate the visual catalog, render every asset, review contact sheets … The permanent skill archive and original-voice/lower-caption/no-persistent-heading defaults remain mandatory for future films."

Every one of those things is now forbidden by 3.0.0:

* `video-director/references/asset-first-workflow.md`: "**No shared visual library, catalog search, stock rigs or archive/sync step.**"
* `tools/validate_bundle.py` fails the build if `remotion-themes/scripts` exists or if any reusable visual artwork enters the release.
* `RELEASE.json` declares `asset_catalog_entries: 0`.
* `README.md` states the starter "injects no palette, font, narrator persona, duration, aspect ratio, scenes, artwork or sound" — the opposite of "lower-caption … defaults remain mandatory".

This is the clearest example in the bundle of an *earlier* version's document outliving its feature set. It is 1.7 KB of instructions that an agent could read first and follow.

### 2.2 `CHANGELOG.md` stops at 1.2.1 *(update)*

The release is 3.0.0. The changelog's newest entry is `# 1.2.1 — 2026-10-23`, so **2.0.0 and 3.0.0 are undocumented**, including the schema-3 re-architecture that every other file in the package is built around. Worse, the surviving final entry describes a *removed* default:

> "Lowered the default subtitle plate from y=1346 to y=1500 at 1080×1920, keeping its 148px height …"

3.0.0 has no default subtitle plate at all: `plan.template.json` ships `"captions": {"enabled": null, "style": null}`, `SmoothPaperCaptions.tsx` throws without an explicit project style, and `validate_bundle.py` rejects a starter with caption geometry. The changelog's last word describes a behaviour the validator now blocks.

### 2.3 `legacy_pause_estimator.py` and its call chain *(delete)*

`video-director/scripts/legacy_pause_estimator.py` (6.5 KB, imports `numpy`) is the previous version's pause-based cut estimator. It is still wired in:

* `after_vo.py` line 24: `if p.get('schema_version') == 3: … else: subprocess.run([... 'legacy_pause_estimator.py' ...])`
* `after_vo.py` still exposes `--tail` (`help='legacy estimator only'`) and `--music`, which are only used inside that `else` branch.
* After estimating, that branch also invokes `auto_sfx.py` automatically — which 3.0.0 otherwise forbids: `auto_sfx.py` is documented as emitting "only explicitly authored events" and "no theme/scene-based sound selection".

So a schema-2 plan silently gets *cut timing inferred from speech pauses and sound events auto-emitted*, i.e. exactly the two behaviours `current-workflow.md` forbids. The file is dead for schema 3, and live only as a trap for old projects. The docs already say older plans "remain unverified" — they do not need an executor.

### 2.4 `init_video.py --no-sfx` — hidden argument from 2.x *(remove the flag)*

```python
ap.add_argument('--no-sfx', action='store_true', help=argparse.SUPPRESS)
```

Suppressing the help text is the tell: the option is undocumented on purpose. It is used by exactly one place — `tests/test_pipeline.py:68` — and no longer does anything meaningful (the initializer injects no sound either way). It survives only because a test calls it.

### 2.5 `remotion-preproduction` — the skill 3.0.0 replaced *(merge or delete)*

`video-director` (version 3.0.0) and `remotion-preproduction` (also relabelled 3.0.0) describe the same phase in near-identical language:

| video-director `SKILL.md` | remotion-preproduction `SKILL.md` |
|---|---|
| "Establish the specific question/purpose, audience, evidence, scope and delivery constraints." | "Establish the current film's purpose, audience, evidence, scope and treatment." |
| "Write `narration` and `shots` independently using schema 3." | "Write speech with scriptwriting. Author shots separately…" |
| "Art direction comes from setting, materials, lighting…" | "Art direction uses setting/material/light/context/tone…" |
| "…following [project asset preparation]." | "…using [asset preparation](../video-director/references/asset-first-workflow.md)." |

The preproduction skill is a thin layer over video-director's own references, and **five of its six reference files are duplicated content that video-director already owns**:

| Preproduction file | Superseded by | Inbound refs |
|---|---|---|
| `references/motion-principles.md` | `video-director/references/camera-and-continuity.md` | **0** |
| `references/qa.md` | `video-director/references/production-quality.md` + `assets/qa-report.template.md` | **0** |
| `references/creative-diagnosis.md` | `production-quality.md` §"Learning from feedback" | **0** |
| `references/script-and-copy.md` | `video-scriptwriting/SKILL.md` + `references/script-format-and-checks.md` | **0** |
| `references/storyboard-timing-specs.md` | `video-director/references/plan-schema.md` + `camera-and-continuity.md` | **0** |
| `references/design-rules.md` | *(unique — this one is real)* | 2 |

This is the classic v2→v3 split that was consolidated into `video-director` but whose leftovers were never removed. Only `design-rules.md` is load-bearing, and it is already reachable through `remotion-themes/SKILL.md`.

### 2.6 `remotion-upgrade/SKILL.md` still upgrades a skill that no longer exists

Line 34:

```bash
npx skills update remotion-best-practices remotion-captions remotion-create remotion-docs \
  remotion-interactivity remotion-maps remotion-markup remotion-multimedia remotion-render \
  remotion-saas remotion-studio remotion-upgrade --yes
```

`remotion-saas` is not in this bundle and `remotion-best-practices/SKILL.md` explicitly says "This bundle focuses on finished videos and **does not include the separate SaaS skill**." Meanwhile `remotion-preproduction`, `remotion-themes`, `video-director`, `video-scriptwriting` and `remotion-sound-design` are *not* in the update list. The one maintenance command in the package is wrong in both directions.

### 2.7 `RELEASE.json` + `validate_bundle.py` enforce the absence of deleted features

`RELEASE.json` carries `asset_catalog_entries: 0` and `story_asset_packages: 0`, and `validate_bundle.py` hard-fails if either is non-zero, or if `asset_workflow != 'plan-create-review-project-assets-before-assembly'`. These are assertions about a v2 concept (a shared visual catalogue). They cost nothing but they are pure legacy: they would only fire if a future author reintroduced a feature that 3.0.0 defines as out of scope.

### 2.8 `tools/audit_references.py` still scans for v2-era phrases

Its `obsolete` list is entirely composed of removed-feature detectors: `assets/library/`, `assets/catalog.json`, `person rig`, `x80/y1500`, `strong blue, warm yellow`, `sound-palettes.json`, `default Cut Paper treatment`, `minimum 4 on each`, `Hook scored 4+`. All of them are now vacuous (the features are gone). Meanwhile the same script's positive "routing" check is the tautology described in §6.1.

### 2.9 `display-captions.md` sample geometry vs. the neutral starter

`remotion-captions/display-captions.md` still centres a 900 px caption block in a 1920 px composition with `bottom: 120`, and `y: 1450` appears in `tools/check_fresh_project.py`. These are harmless *examples*, but they are the only numeric caption geometry left in the package, and the changelog's `y=1500` default reinforces the impression that a default exists. Anyone reading the three files together will conclude the package recommends a specific plate position, which §2.2 says it must not.

---

## 3. Unreachable content — the routing graph

### 3.1 Files no entry point can reach

Reachability was tested twice: (a) following real Markdown links from all 16 `SKILL.md` files, (b) searching the whole bundle for the filename in any form. **13 Markdown files fail both.**

| File | Bytes | Unique content? | Disposition |
|---|---:|---|---|
| `skills/remotion-preproduction/references/creative-diagnosis.md` | 723 | No — restated in `production-quality.md` | delete |
| `skills/remotion-preproduction/references/motion-principles.md` | 1,154 | No — restated in `camera-and-continuity.md` | delete |
| `skills/remotion-preproduction/references/qa.md` | 670 | No — restated in `production-quality.md` + QA template | delete |
| `skills/remotion-preproduction/references/script-and-copy.md` | 661 | No — restated in `video-scriptwriting` | delete |
| `skills/remotion-preproduction/references/storyboard-timing-specs.md` | 1,118 | No — restated in `plan-schema.md` | delete |
| `skills/video-scriptwriting/references/rewrite-examples.md` | 846 | Yes (revision method) | link it, or fold into `voice-and-craft.md` |
| `skills/video-scriptwriting/references/structures-and-formats.md` | 1,007 | Yes (structure choice) | link it, or fold into `hooks.md` |
| `skills/remotion-markup/light-leaks.md` | 3,279 | No — `lightLeak()` already catalogued in `effects.md` | delete |
| `skills/remotion-sound-design/references/mixing-and-loudness.md` | 3,111 | **Yes** — LUFS, true-peak, EQ targets | link it (this is a routing bug, not a stale file) |
| `skills/remotion-sound-design/references/remotion-audio.md` | 3,110 | **Yes** — ducking curve, cue-list integration | link it |
| `skills/remotion-sound-design/references/sound-design-principles.md` | 1,044 | Partly — overlaps `SKILL.md` | link it or merge into `SKILL.md` |
| `skills/remotion-sound-design/references/sources-and-licensing.md` | 1,908 | **Yes** — the only licence-checking guidance in the package | link it |
| `skills/remotion-sound-design/references/why-it-sounds-bad.md` | 2,642 | Partly — overlaps `mixing-and-loudness.md` | link it or merge |

Two different diseases are visible here:

* **Preproduction + light-leaks (7 files, ~7.6 KB): genuinely stale.** Their content already lives in a file that *is* routed. Deleting them loses nothing.
* **Sound design + two scriptwriting files (7 files, ~13.7 KB): orphans by omission.** The content is unique and operationally important, but `remotion-sound-design/SKILL.md` links only to `../../video-director/references/delivery-workflow.md`. An agent that loads the sound skill is told *that* sound matters and never told *how* to measure LUFS, how to duck music, how to license a sample, or that `SfxTrack.tsx` exists. **This is a functional defect in the skill, not dead weight.** Note also that `mixing-and-loudness.md` is the only file that documents `audit_audio.py` usage, and `sources-and-licensing.md` is the only file that documents `synth_sfx.py`.

### 3.2 Skills that nothing points at

Counted as inbound Markdown links to each `SKILL.md`:

| Skill | Inbound links | Notes |
|---|---:|---|
| `remotion-sound-design` | **0** | Only its Python scripts are referenced (by `finalize.py`/`auto_sfx.py`). |
| `remotion-themes` | **0** | The art-direction and Cut Paper skill. Its `THEME.md` is therefore unreachable too. |
| `remotion-docs` | 1 | router only |
| `remotion-multimedia` | 1 | router only |
| `remotion-preproduction` | 1 | `video-director` only |
| `remotion-upgrade` | 1 | router only |
| `remotion-create` | 2 | |
| `remotion-captions` | 2 | |
| `remotion-maps` | 3 | |
| `remotion-render` | 3 | |
| `remotion-studio` | 3 | |
| `video-director` | 3 | entry point |
| `video-scriptwriting` | 3 | |
| `remotion-markup` | 4 | |
| `remotion-best-practices` | 5 | router |
| `remotion-interactivity` | 7 | |

The router at `remotion-best-practices/SKILL.md` links to eleven skills and **omits four**: `video-scriptwriting`, `remotion-preproduction`, `remotion-themes`, `remotion-sound-design`. Combined with the fact that `video-director/SKILL.md` links only to preproduction and scriptwriting, the package graph has two disconnected clusters:

```
best-practices ──► create / markup / studio / render / captions / maps /
                   multimedia / interactivity / docs / upgrade / video-director
video-director ──► preproduction / video-scriptwriting
                   (dead end: nothing links to themes, sound-design, or the
                    sound-design references)
```

An agent that starts at `best-practices` (the skill named "Router for all Remotion skills") and works on a narrated paper film will be routed to `video-director` and then to *nothing* about art direction or sound. It must discover `remotion-themes` and `remotion-sound-design` by reading the skill registry, not by following the documented workflow.

### 3.3 Code and asset files with no reference

| File | Bytes | Checked against | Finding |
|---|---:|---|---|
| `skills/video-director/scripts/doctor.py` | 2,711 | all MD, all Py, all YAML | **0 references.** Nothing ever tells the agent to run the environment check, and no script calls it. It is a useful tool that is unreachable by documentation. |
| `skills/remotion-sound-design/assets/example-cues.json` | 741 | all MD, all Py | **0 references.** Also stale in content: it ends with a `chime-success-01.wav` cue noted `"CTA"`, while `video-director` and `channel.json` both say no call to action is assumed. |
| `skills/remotion-markup/assets/useProjectFonts.ts` | 623 | all MD, all Py | Referenced **only** by `tools/check_fresh_project.py`. The documentation teaches `loadFont()` from `@remotion/fonts` (`local-fonts.md`), so two competing font loaders ship with no statement of which is canonical. |
| `tools/audit_references.py` | 1,807 | `README.md` verification block | Not listed among the documented commands (README lists `validate_bundle.py`, `unittest`, `check_fresh_project.py`, `package_release.py`). It regenerates `REFERENCE-AUDIT.json` and is therefore necessary for a reproducible release. |
| `skills/remotion-themes/themes/_shared/fonts/Bricolage.ttf` | 408,496 | all MD, all Py, all YAML | **0 references.** |
| `skills/remotion-themes/themes/_shared/fonts/Caveat.ttf` | 403,648 | all MD, all Py, all YAML | **0 references.** |
| `skills/remotion-themes/themes/_shared/fonts/Bricolage-OFL.txt`, `Caveat-OFL.txt` | 8,788 | — | Only meaningful while the two fonts above ship. |

`DMSans.ttf` is referenced — but only by `tools/check_fresh_project.py`, which copies it to a project as `Selected.ttf` for a compile test. **No instruction in the package tells a film-maker that three licensed fonts are available, what they are called, or where they live.** `remotion-markup/SKILL.md` says only "For Cut Paper, use the bundled local fonts and notices" — with no path, no family names, and no licence pointer; `remotion-themes/SKILL.md` says "optional licensed font resources" without naming them; `THIRD_PARTY.md` is the only file that names Bricolage, Caveat and DM Sans, and it is a legal notice, not guidance.

### 3.4 Soft links that are not links

`remotion-maps` avoids the hard-link problem by naming files in backticks, so its seven reference files are discoverable (``Read `references/render-stability.md` ``). That is worth noting because `validate_bundle.py` only checks real links — the map docs would pass even if they pointed nowhere. Two other cases are worse:

* `remotion-maps/techniques/maptiler/references/map-explainer-architecture.md` refers to `data-sources.md` and `geo-prep.md` — abbreviated names that do not resolve.
* `remotion-maps/techniques/cesium/references/3d-troubleshooting.md` states: "The standalone `flythrough.html` / `render.mjs` / `probe.mjs` from the original spike **are kept** only as the record of this dead-end." None of those three files are in the package. The sentence describes files that were removed.

---

## 4. Redundancy and duplication

### 4.1 Byte-identical files

| Hash group | Files | Size each |
|---|---|---|
| `5007669b…` | `remotion-{best-practices,captions,create,docs,interactivity,maps,markup,multimedia,render,studio,upgrade}/assets/remotion-icon.svg` | 1,303 B × 11 |
| `41beac50…` | `remotion-maps/techniques/{mapbox,maplibre,maptiler}/references/render-stability.md` | 3,779 B × 3 |

The icon duplication is unavoidable (each skill folder must be self-contained for installation). The `render-stability.md` triplication is avoidable: it is one document about headless WebGL capture, filed three times because each technique folder copies the others. It should be one shared reference (e.g. `remotion-maps/references/render-stability.md`) with three links.

### 4.2 Near-duplicate documents (token-set Jaccard)

| Similarity | A | B | Assessment |
|---:|---|---|---|
| **1.00** | `mapbox/references/render-stability.md` | `maptiler/references/render-stability.md` | identical (§4.1) |
| **0.85** | `remotion-multimedia/get-audio-duration.md` | `get-video-duration.md` | same Mediabunny snippet, different accessor |
| **0.75** | `remotion-maps/techniques/mapbox/TECHNIQUE.md` | `maplibre/TECHNIQUE.md` | same document, `mapbox-gl`→`maplibre-gl`, token paragraph removed. ~27 KB combined for one technique. |
| **0.70** | `remotion-markup/audio.md` | `embedding-videos.md` | <Audio>/<Video> props tables duplicated |
| **0.65** | `remotion-multimedia/get-audio-duration.md` | `get-video-dimensions.md` | same snippet again |
| **0.35** | `remotion-interactivity/timeline-structure.md` | `remotion-markup/connected-compositions.md` | deliberate split, acceptable |
| **0.34** | `remotion-markup/google-fonts.md` | `local-fonts.md` | two halves of one "fonts" topic |

Paraphrase-level duplication that the token metric understates (because the studio docs are heavily rewritten in a different voice), verified by reading:

* `remotion-sound-design/references/remotion-audio.md` retreads `remotion-markup/audio.md` and `remotion-markup/sfx.md`, then adds the genuinely unique ducking curve and cue-list section. The right shape is one audio doc plus an "integration into a mixed timeline" section.
* `remotion-sound-design/references/sound-design-principles.md` repeats the first paragraph of `remotion-sound-design/SKILL.md` almost line for line.
* `remotion-sound-design/references/why-it-sounds-bad.md` is a 16-row symptom table that restates the targets in `mixing-and-loudness.md`; useful as a diagnostic appendix, not as a separate routed document.
* `remotion-preproduction` + its five references vs. `video-director`'s five references (§2.5).

### 4.3 Repeated boilerplate

72 Markdown files carry the identical line

```
Process and authority: [decision architecture](../video-director/references/current-workflow.md).
```

and 72 carry `<!-- studio-reference:3.0.0 -->`. That is intentional branding/routing, but it is also what makes the "routing" audit meaningless (§6.1): every file including an orphan has the line.

---

## 5. Optional skills and dead weight, by size

Compressed contribution to the 934,442-byte download:

| Area | Compressed | Share | Verdict |
|---|---:|---:|---|
| `remotion-themes` (2 KB docs + 1.05 MB fonts) | 587,196 | **62.8 %** | 3 docs KB + 3 fonts; 2 of the fonts unreferenced |
| `remotion-maps` | 164,799 | **17.6 %** | optional specialist skill; one 333 KB JSON |
| `remotion-markup` | 50,931 | 5.5 % | core |
| `video-director` | 32,183 | 3.4 % | core |
| `remotion-sound-design` | 16,984 | 1.8 % | partly unreachable |
| `tests` + `tools` | 15,889 | 1.7 % | maintainer surface |
| `remotion-captions` | 8,763 | 0.9 % | core |
| `remotion-interactivity` | 8,753 | 0.9 % | core |
| everything else | ~35 KB | ~4 % | |

**5.1 Fonts — 1.05 MB raw.** Bricolage (408 KB) and Caveat (404 KB) account for **50.4 % of the entire ZIP** and are named in zero instruction files. This is the single largest removable object in the release. Two honest options:

* **Keep and document** — add the family names, paths, weights and licence pointer to `remotion-themes/SKILL.md` and `remotion-markup/local-fonts.md`, so the agent can actually find them; and note them in `README.md` ("three optional OFL fonts are included").
* **Remove** — delete both `.ttf` and their two `-OFL.txt` notices; keep `DMSans.ttf` because the fresh-project test requires it, or replace the test fixture with a system font.

Note the contradiction with 3.0.0's own history: CHANGELOG 1.2.0 says "Removed … unused fonts", yet two clearly-unused fonts remain.

**5.2 Maps — 491 KB raw, 82 KB of Markdown.** This is an upstream specialist skill with five techniques, two of which ship a 27 KB near-duplicate of each other, one of which ships a 333 KB generated dataset. Nothing in the directed-film workflow needs it; `channel.json` selects `cut-paper`, and `remotion-markup/SKILL.md` `light-leaks`/maps links are the only paths in. It is a legitimate *optional* skill for briefs about geography — but it should be:

* de-duplicated (`mapbox` vs `maplibre`; the three `render-stability.md` copies);
* assessed for the 333 KB `sample-data/country-meta.json`, which is generated boundary geometry whose provenance is not stated in `THIRD_PARTY.md` (see §6.4);
* moved out of the "16 core resources" count so the release does not imply it is part of the film workflow.

**5.3 `remotion-multimedia` — 4 files, 6 KB.** Mediabunny metadata helpers (`get-audio-duration`, `get-video-duration`, `get-video-dimensions`). The directed workflow never uses them: `after_vo.py` measures the take with `ffmpeg`, `finalize.py` and `verify_export.py` use `ffprobe`. It is only relevant if a *browser* needs metadata. Either keep it explicitly as a browser-media skill (and say so in the router) or fold the three snippets into `remotion-markup/calculate-metadata.md`, which already links to it.

**5.4 `synth_sfx.py` — 13.8 KB + `numpy`/`scipy`.** No script imports it; no document invokes it except the unrouted `sources-and-licensing.md` ("`scripts/synth_sfx.py` can create project sound sources when its synthesis approach suits"). It is a legitimate optional generator, but as shipped it is an unreachable dependency on two heavy libraries. Document it from the sound skill or move it out.

**5.5 `doctor.py` — 2.7 KB.** Genuinely useful, entirely unreferenced (§3.3).

**5.6 `remotion-docs` — 1.5 KB.** Embeds an Algolia **search-only** key and app ID. That is the same public key the Remotion docs site uses for browser search and is not a secret, but shipping a hard-coded third-party credential inside a distributed skill bundle deserves an explicit note (rotate-if-abused, scope, terms). Nothing else in the package hard-codes a credential, and `THIRD_PARTY.md` says "No provider credentials are included" — technically true (it is a public search key), but the statement should be qualified.

---

## 6. Verification-integrity problems

These matter more than any individual orphan, because they are the reasons the orphans were not caught.

### 6.1 The "99 Markdown references pass routing" check is a tautology

`tools/audit_references.py`:

```python
routed = 'current-workflow.md' in text or p.name == 'current-workflow.md'
if not routed: errors.append('Missing current-default routing: ' + relative)
```

98 of the 99 skill Markdown files contain that string; the 99th *is* `current-workflow.md`. So the pass condition is satisfied by construction for every file in the bundle, whether or not anything links to it. **Result: 99/99 PASS while 13 files have no inbound reference at all.** The audit certifies a copy-pasted line, not reachability.

`REFERENCE-AUDIT.json` therefore records `"current_routing": true` for `light-leaks.md`, for all five preproduction leftovers, and for all five unreferenced sound-design files. The `VALIDATION.md` sentence "99 skill Markdown references audited for architecture routing" overstates what the script does.

**Fix:** make the check a real graph walk — parse `](…)` links from all 16 `SKILL.md` entry points, resolve them, and fail on any Markdown file outside the reachable set. That single change would have surfaced 20 files (13 unreachable + 7 map references that are only named in backticks).

### 6.2 `validate_bundle.py` checks syntax and links but not reachability

It validates: folder set, front-matter `name`, agent registration, Python/JSON/SVG syntax, local link targets, manifest hashes, theme folder contents, and neutral starter fields. It never asks whether a document is reachable. It *does* have a rule requiring exactly these 16 skill folders, so any consolidation (§7) must update `EXPECTED_SKILLS` and `RELEASE.json.skill_count` in the same change.

### 6.3 "34 regressions passed" is a defined-count, not a run-count

`VALIDATION.md` and `README.md` both cite 34. There are exactly 34 `def test_` functions (13 + 10 + 11), so the number is honest *when ffmpeg and numpy are present*. In this environment discovery collected 24 (the 10 `PipelineRegression` tests collapse into one `setUpClass` skip without ffmpeg) and 3 of the remaining 24 errored on the missing binary. Recommendation: state the prerequisite next to the number, e.g. "34 tests (requires ffmpeg, ffprobe, numpy, scipy; 14 are skipped or error without them)".

### 6.4 `THIRD_PARTY.md` and the shipped map dataset

> "Optional geography implementation references describe external datasets and provider services with their own access, attribution and licensing requirements. **The former map theme and its Natural Earth datasets are excluded from this release.**"

But `remotion-maps/techniques/maptiler/assets/sample-data/` ships 346 KB of generated geography (`country-meta.json` contains baked China/India/Bangladesh border polylines; `yarlung-flow.json` a river centreline), and `prep-geo.mjs` says the skill "ships the OUTPUTS in `assets/sample-data/` instead" of its source data. The source of those outputs, and therefore their licence/attribution terms, is not stated anywhere. Either name the source and terms, or drop the dataset and let `prep-geo.mjs` generate it.

### 6.5 Counts that must move together

Adding or removing anything touches: `MANIFEST.json` (regenerated by `package_release.py`), `REFERENCE-AUDIT.json` (`markdown_files`), `RELEASE.json` (`skill_count`), `validate_bundle.EXPECTED_SKILLS`, `README.md` ("All 16 resources"), `VALIDATION.md`, and the ZIP + `.sha256`. The package is internally consistent today; any edit must be re-run through `package_release.py` rather than hand-patched.

---

## 7. Guidance conflicts (not stale, but contradictory)

| # | Where | Conflict |
|---|---|---|
| 1 | `remotion-markup/sfx.md` vs `remotion-sound-design/references/why-it-sounds-bad.md` | The former offers `vine-boom.wav`, `bruh.wav`, `anime-wow.wav`, `spongebob-fail.wav` as the standard set; the latter lists "Meme/cheesy feel → Vine-boom, bruh, stock 'swoosh' packs" as a defect to fix. An agent can follow one file and fail the other. |
| 2 | `remotion-captions/SKILL.md` → `display-captions.md` vs `references/smooth-subtitles.md` | Two renderers, no statement of default: the Basic Captions element (fetched from remotion.dev, caption editor writes back to the prop) and the local `SmoothPaperCaptions.tsx` (explicit style object, required by `plan_docs.py`). The channel profile asks for "lower, smooth phrase changes", which only the local renderer guarantees. |
| 3 | `remotion-captions/SKILL.md` (typo "alreay") + `display-captions.md` sample geometry | Minor, but it is the only numeric caption layout left and it reads like a recommendation. |
| 4 | `remotion-markup/voiceover.md` (ElevenLabs TTS) vs `video-director` ("keep the supplied voice intact") | Both are explicitly scoped ("Only when the user explicitly authorizes new TTS"), so this is acceptable — just make sure the router's chain reaches the qualifier. |
| 5 | `remotion-maps/techniques/maptiler/assets/tokens.ts` vs `validate_bundle.py` ("reusable visual artwork/catalog must not enter the release") | `tokens.ts` hard-codes a topic palette (`china: #D4A853`, `india: #5B8A8A`, `bangladesh: #C07B57`) inside a skill. It is labelled "Example tokens only", but it is the only colour inventory in the package and it is exactly the kind of default 3.0.0 says it removed. |

---

## 8. Verdict by skill

| Skill | Role in the 3.0.0 workflow | Reachable? | Verdict |
|---|---|---|---|
| `video-director` | entry point; whole pipeline | yes (root) | **keep** — core |
| `video-scriptwriting` | speech, step 2 | yes (from director) | **keep**; link 2 orphan refs |
| `remotion-markup` | implementation guidance | yes | **keep**; delete `light-leaks.md`, resolve font-loader ambiguity, fix meme-sound conflict |
| `remotion-captions` | caption contract + renderer | yes | **keep**; state which renderer the workflow expects |
| `remotion-interactivity` | Studio editability | yes | **keep** |
| `remotion-best-practices` | router | yes | **keep**; add the 4 missing skills |
| `remotion-create` | scaffolding | yes | **keep** |
| `remotion-studio` | preview | yes | **keep** |
| `remotion-render` | export | yes | **keep** |
| `remotion-themes` | art direction + Cut Paper | **no** | **route it**; decide the fonts (§5.1) |
| `remotion-sound-design` | sound treatment + mix | **no** | **route it**; wire its 5 references |
| `remotion-preproduction` | superseded planning layer | yes (director only) | **merge into video-director**, keep `design-rules.md` somewhere routed, delete the 5 leftovers |
| `remotion-multimedia` | browser media metadata | router only | **keep as optional**; describe it as browser-only, or fold into `calculate-metadata.md` |
| `remotion-maps` | geography specialist | router + markup | **keep as optional**; de-duplicate, resolve dataset provenance, stop counting it as core |
| `remotion-docs` | API lookup | router only | **keep**; qualify the embedded search key |
| `remotion-upgrade` | maintenance | router only | **keep**; fix the skill list |

---

## 9. Recommended cleanup, in three tiers

### Tier 1 — safe deletions (no content lost, ~18 KB, zero routing impact)

1. `PLAN.md`
2. `skills/remotion-preproduction/references/{creative-diagnosis,motion-principles,qa,script-and-copy,storyboard-timing-specs}.md`
3. `skills/video-scriptwriting/references/{rewrite-examples,structures-and-formats}.md` — *or* Tier 2 instead
4. `skills/remotion-markup/light-leaks.md`
5. `skills/remotion-sound-design/assets/example-cues.json`
6. `skills/video-director/scripts/legacy_pause_estimator.py` (+ drop the `else` branch, `--tail`, `--music` in `after_vo.py`)

Bookkeeping: regenerate `MANIFEST.json` via `tools/package_release.py`; `REFERENCE-AUDIT.json.markdown_files` 99 → 91.

### Tier 2 — routing and consolidation fixes (no deletions, but the biggest quality gain)

7. `remotion-best-practices/SKILL.md`: add links for `video-scriptwriting`, `remotion-themes`, `remotion-sound-design` (and `remotion-preproduction` if it survives).
8. `remotion-sound-design/SKILL.md`: link all five references (and `assets/SfxTrack.tsx`).
9. `video-scriptwriting/SKILL.md`: link `hooks.md` + `voice-and-craft.md` + `script-format-and-checks.md` **and** `rewrite-examples.md` / `structures-and-formats.md`.
10. `remotion-themes/SKILL.md`: link `themes/cut-paper/THEME.md` explicitly, name the three bundled fonts, and point at `remotion-preproduction/references/design-rules.md` (or move that file here).
11. Document `doctor.py` (Video Director "Understand and plan" step or README verification block).
12. Document `tools/audit_references.py` in the README verification block.
13. Merge `remotion-preproduction` into `video-director` (skill count 16 → 15), or keep it as a one-page pointer.
14. Replace the "routing" check in `tools/audit_references.py` with a real reachability walk (§6.1).
15. Fix `remotion-upgrade`'s `npx skills update` list (`remotion-saas` out; `remotion-preproduction`, `remotion-themes`, `remotion-sound-design`, `video-director`, `video-scriptwriting` in, per what ships).
16. Add CHANGELOG entries for 2.0.0 and 3.0.0; stop the 1.2.1 entry being the changelog's headline.
17. Resolve the sfx/meme-sound conflict, the two-caption-renderer ambiguity, and the `useProjectFonts.ts` vs `@remotion/fonts` ambiguity by naming one default.
18. Fix `3d-troubleshooting.md`'s reference to `render.mjs`/`probe.mjs`/`flythrough.html`, and `map-explainer-architecture.md`'s `data-sources.md`/`geo-prep.md` abbreviations.

### Tier 3 — size decisions (needs your call)

19. **Fonts:** document Bricolage/Caveat (family, path, weights, OFL) **or** remove them → 470 KB off the download, 50 % of the ZIP. `DMSans.ttf` stays only while the fresh-project test uses it.
20. **Maps:** de-duplicate `mapbox` vs `maplibre` (−13 KB), collapse the three `render-stability.md` copies into one (−7.5 KB), decide on the 333 KB `country-meta.json` sample data (+ licensing note either way).
21. **`synth_sfx.py`:** keep and document, or move out of the skill bundle.
22. **`remotion-multimedia`:** keep as an explicitly-optional browser skill, or fold into `remotion-markup/calculate-metadata.md`.
23. **`remotion-preproduction`:** delete the skill outright (scenario 13) or keep the name as a compatibility pointer.

---

## 10. What should NOT be touched

Worth stating explicitly, because a size-driven cleanup could easily destroy the release's value:

* `video-director/references/current-workflow.md`, `plan-schema.md`, `asset-first-workflow.md`, `delivery-workflow.md`, `production-quality.md`, `camera-and-continuity.md` — the actual 3.0.0 architecture. Every other file should be judged against these.
* `video-director/scripts/{init_video,plan_docs,check_assets,after_vo,auto_sfx,finalize,verify_export}.py` — the pipeline is genuinely wired: `init_video` → `plan_docs` → `check_assets --seal` → `after_vo` → `auto_sfx` → `finalize` → `verify_export`. Every call in that chain is exercised by both docs and tests.
* `remotion-captions/assets/SmoothPaperCaptions.tsx` + `references/smooth-subtitles.md` — the only implementation of the project-caption contract.
* `tests/` and `tools/` — including `check_fresh_project.py`, which is the only end-to-end evidence in the package.
* `FRESH-PROJECT-CHECK.json`, `TREATMENT-EVALUATION.json` — release evidence, correctly separated from the skills.
* The eleven upstream `remotion-*` skills, even the optional ones: they are licensed third-party material with their own attribution (`THIRD_PARTY.md`), and removal is a product decision, not a hygiene decision.

---

## Appendix A — complete file inventory (184 files)

`Name refs` counts how many other files mention the filename anywhere (documents or code). 0 means literally nothing in the package refers to it.

| File | Bytes | Name refs | Verdict | Note |
|---|---:|---:|---|---|
| **root** | | | | |
| `CHANGELOG.md` | 1,669 | 0 | **UPDATE** | Stops at 1.2.1; no 2.0.0/3.0.0 entry. Its final entry describes a default subtitle position that 3.0.0 removed. |
| `FRESH-PROJECT-CHECK.json` | 588 | 3 | **KEEP** | Release evidence. |
| `MANIFEST.json` | 22,297 | 2 | **KEEP** | Integrity record. |
| `PLAN.md` | 1,714 | 2 | **DELETE** | v1.5.0 plan for the removed asset library/catalog/CameraRig. Contradicts the 3.0.0 asset policy. |
| `README.md` | 3,829 | 0 | **KEEP+** | Correct except counts (16 resources / 34 tests) and no pointer to themes/sound-design. |
| `REFERENCE-AUDIT.json` | 19,286 | 3 | **KEEP** | Regenerate; its "routing" column is a boilerplate-string test, not a reachability test. |
| `RELEASE.json` | 487 | 2 | **KEEP+** | Vestigial fields `asset_catalog_entries` and `story_asset_packages` only restate removed features. |
| `THIRD_PARTY.md` | 1,473 | 1 | **UPDATE** | Says map datasets are excluded, but 333 KB of generated country boundaries ships in remotion-maps/maptiler/assets/sample-data. |
| `TREATMENT-EVALUATION.json` | 5,150 | 1 | **KEEP** | Release evidence. |
| `VALIDATION.md` | 3,976 | 2 | **KEEP** | Accurate when ffmpeg is available (24 of 34 tests cannot run without it). |
| **skills/remotion-best-practices** | | | | |
| `skills/remotion-best-practices/SKILL.md` | 4,747 | 14 | **KEEP+** | Router omits video-scriptwriting, remotion-preproduction, remotion-themes and remotion-sound-design. |
| `skills/remotion-best-practices/agents/openai.yaml` | 311 | 1 | **KEEP** |  |
| `skills/remotion-best-practices/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| **skills/remotion-captions** | | | | |
| `skills/remotion-captions/SKILL.md` | 1,579 | 14 | **KEEP+** | Typo "alreay"; offers both BasicCaptions and SmoothPaperCaptions without stating which the workflow expects. |
| `skills/remotion-captions/agents/openai.yaml` | 305 | 1 | **KEEP** |  |
| `skills/remotion-captions/assets/SmoothPaperCaptions.tsx` | 2,135 | 3 | **KEEP** | Technical renderer required by the caption contract. |
| `skills/remotion-captions/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| `skills/remotion-captions/display-captions.md` | 6,460 | 4 | **KEEP?** | Alternative renderer (Basic Captions element). |
| `skills/remotion-captions/import-srt-captions.md` | 2,362 | 1 | **KEEP?** | SRT path. |
| `skills/remotion-captions/references/smooth-subtitles.md` | 1,419 | 2 | **KEEP** | Project caption contract. |
| `skills/remotion-captions/transcribe-captions.md` | 5,664 | 3 | **KEEP?** | Whisper path; needs a GPU. |
| **skills/remotion-create** | | | | |
| `skills/remotion-create/SKILL.md` | 3,625 | 14 | **KEEP** |  |
| `skills/remotion-create/agents/openai.yaml` | 310 | 1 | **KEEP** |  |
| `skills/remotion-create/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| `skills/remotion-create/tailwind.md` | 517 | 1 | **KEEP?** | Two rules; only relevant if Tailwind is requested. |
| `skills/remotion-create/video-layout.md` | 636 | 1 | **KEEP** |  |
| **skills/remotion-docs** | | | | |
| `skills/remotion-docs/SKILL.md` | 1,512 | 14 | **KEEP?** | Algolia search recipe; public search-only key embedded. |
| `skills/remotion-docs/agents/openai.yaml` | 307 | 1 | **KEEP** |  |
| `skills/remotion-docs/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| **skills/remotion-interactivity** | | | | |
| `skills/remotion-interactivity/SKILL.md` | 13,962 | 15 | **KEEP** |  |
| `skills/remotion-interactivity/agents/openai.yaml` | 320 | 1 | **KEEP** |  |
| `skills/remotion-interactivity/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| `skills/remotion-interactivity/inline-effects.md` | 1,124 | 1 | **KEEP** |  |
| `skills/remotion-interactivity/timeline-structure.md` | 7,147 | 7 | **KEEP** |  |
| **skills/remotion-maps** | | | | |
| `skills/remotion-maps/SKILL.md` | 1,163 | 15 | **KEEP?** | Optional specialist skill; 17.6% of the bundle; not needed by the Cut Paper channel workflow. |
| `skills/remotion-maps/agents/openai.yaml` | 293 | 1 | **KEEP?** |  |
| `skills/remotion-maps/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** | Registration icon. |
| `skills/remotion-maps/techniques/cesium/TECHNIQUE.md` | 4,046 | 3 | **KEEP?** | Unique 3D branch. |
| `skills/remotion-maps/techniques/cesium/assets/CesiumFlythrough.tsx` | 8,912 | 3 | **KEEP?** |  |
| `skills/remotion-maps/techniques/cesium/assets/cesium-path.json` | 12,329 | 1 | **KEEP?** |  |
| `skills/remotion-maps/techniques/cesium/assets/city-path.json` | 223 | 1 | **KEEP?** |  |
| `skills/remotion-maps/techniques/cesium/assets/example-Root.tsx` | 923 | 2 | **KEEP?** |  |
| `skills/remotion-maps/techniques/cesium/assets/flight-path.ts` | 1,240 | 1 | **KEEP?** |  |
| `skills/remotion-maps/techniques/cesium/assets/sample-river.geojson` | 6,409 | 1 | **KEEP?** | Input for prep-cesium-path.mjs. |
| `skills/remotion-maps/techniques/cesium/references/3d-data-sources.md` | 1,591 | 1 | **KEEP?** |  |
| `skills/remotion-maps/techniques/cesium/references/3d-flyover-architecture.md` | 8,027 | 1 | **KEEP?** |  |
| `skills/remotion-maps/techniques/cesium/references/3d-troubleshooting.md` | 6,881 | 1 | **KEEP?** | Retains a "dead-end" record referring to render.mjs/probe.mjs that are not shipped. |
| `skills/remotion-maps/techniques/cesium/scripts/prep-cesium-path.mjs` | 5,561 | 2 | **KEEP?** |  |
| `skills/remotion-maps/techniques/mapbox/TECHNIQUE.md` | 13,265 | 3 | **KEEP?** | 75% token overlap with maplibre; the two differ mainly by package name and token env var. |
| `skills/remotion-maps/techniques/mapbox/references/render-stability.md` | 3,779 | 4 | **DEDUPE** | Byte-identical to the maplibre and maptiler copies. |
| `skills/remotion-maps/techniques/maplibre/TECHNIQUE.md` | 14,070 | 3 | **MERGE?** | Near-duplicate of mapbox/TECHNIQUE.md. |
| `skills/remotion-maps/techniques/maplibre/references/render-stability.md` | 3,779 | 4 | **DEDUPE** | Byte-identical copy. |
| `skills/remotion-maps/techniques/maptiler/TECHNIQUE.md` | 4,655 | 3 | **KEEP?** | Unique MapTiler pipeline. |
| `skills/remotion-maps/techniques/maptiler/assets/CountryLabel.tsx` | 1,849 | 2 | **KEEP?** |  |
| `skills/remotion-maps/techniques/maptiler/assets/MapTilerVectorElement.ts` | 2,183 | 2 | **KEEP?** |  |
| `skills/remotion-maps/techniques/maptiler/assets/RiverReveal.tsx` | 10,830 | 3 | **KEEP?** |  |
| `skills/remotion-maps/techniques/maptiler/assets/example-Root.tsx` | 1,039 | 2 | **KEEP?** |  |
| `skills/remotion-maps/techniques/maptiler/assets/sample-data/country-meta.json` | 333,134 | 2 | **KEEP?** | 333 KB generated borders; provenance/licence not stated in THIRD_PARTY.md. |
| `skills/remotion-maps/techniques/maptiler/assets/sample-data/yarlung-flow.json` | 13,082 | 0 | **KEEP?** | Sample route for RiverReveal. |
| `skills/remotion-maps/techniques/maptiler/assets/tokens.ts` | 1,344 | 2 | **KEEP?** | Bakes a topic palette (China/India/Bangladesh) into the skill; labelled example, but it is the only colour inventory left in the package. |
| `skills/remotion-maps/techniques/maptiler/references/map-data-sources.md` | 4,938 | 1 | **KEEP?** | Textual reference only (no link); discoverable. |
| `skills/remotion-maps/techniques/maptiler/references/map-explainer-architecture.md` | 7,011 | 1 | **KEEP?** |  |
| `skills/remotion-maps/techniques/maptiler/references/map-geo-prep.md` | 4,238 | 1 | **KEEP?** |  |
| `skills/remotion-maps/techniques/maptiler/references/render-stability.md` | 3,779 | 4 | **DEDUPE** | Byte-identical copy. |
| `skills/remotion-maps/techniques/maptiler/scripts/prep-geo.mjs` | 7,553 | 3 | **KEEP?** |  |
| `skills/remotion-maps/techniques/static-map/TECHNIQUE.md` | 1,280 | 3 | **KEEP?** | Smallest technique; fine. |
| **skills/remotion-markup** | | | | |
| `skills/remotion-markup/3d.md` | 2,423 | 1 | **KEEP** |  |
| `skills/remotion-markup/SKILL.md` | 13,629 | 14 | **KEEP** | Main implementation router; long but every section routes onward. |
| `skills/remotion-markup/agents/openai.yaml` | 310 | 1 | **KEEP** |  |
| `skills/remotion-markup/arabic-text.md` | 4,671 | 1 | **KEEP** |  |
| `skills/remotion-markup/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| `skills/remotion-markup/assets/useProjectFonts.ts` | 623 | 1 | **ROUTE?** | Documented only in the fresh-project test harness; local-fonts.md teaches @remotion/fonts instead, so two competing loaders exist. |
| `skills/remotion-markup/audio-visualization.md` | 5,023 | 1 | **KEEP** |  |
| `skills/remotion-markup/audio.md` | 4,252 | 3 | **KEEP** |  |
| `skills/remotion-markup/calculate-metadata.md` | 3,629 | 3 | **KEEP** |  |
| `skills/remotion-markup/color-correction.md` | 4,527 | 2 | **KEEP** |  |
| `skills/remotion-markup/compositions.md` | 5,265 | 5 | **KEEP** |  |
| `skills/remotion-markup/connected-compositions.md` | 3,642 | 5 | **KEEP** |  |
| `skills/remotion-markup/cropping.md` | 1,451 | 1 | **KEEP** |  |
| `skills/remotion-markup/effects.md` | 11,005 | 3 | **KEEP+** | Refers to `packages/example/...` and `src/effects/palette-map.ts` paths from the upstream Remotion repo that do not exist here. |
| `skills/remotion-markup/embedding-videos.md` | 3,856 | 1 | **KEEP** |  |
| `skills/remotion-markup/ffmpeg.md` | 1,243 | 2 | **KEEP** |  |
| `skills/remotion-markup/gifs.md` | 3,914 | 1 | **KEEP** |  |
| `skills/remotion-markup/google-fonts.md` | 1,845 | 2 | **KEEP** |  |
| `skills/remotion-markup/html-in-canvas.md` | 3,107 | 1 | **KEEP** |  |
| `skills/remotion-markup/images.md` | 1,699 | 1 | **KEEP** |  |
| `skills/remotion-markup/light-leaks.md` | 3,279 | 0 | **DELETE** | No inbound link anywhere; lightLeak() is already listed in effects.md. |
| `skills/remotion-markup/local-fonts.md` | 1,664 | 1 | **KEEP** |  |
| `skills/remotion-markup/lottie.md` | 1,937 | 1 | **KEEP** |  |
| `skills/remotion-markup/measuring-dom-nodes.md` | 1,126 | 1 | **KEEP** |  |
| `skills/remotion-markup/measuring-text.md` | 2,932 | 1 | **KEEP** |  |
| `skills/remotion-markup/motion-blur.md` | 2,170 | 1 | **KEEP** |  |
| `skills/remotion-markup/multi-scene-video.md` | 1,166 | 2 | **KEEP** |  |
| `skills/remotion-markup/parameters.md` | 2,534 | 1 | **KEEP** |  |
| `skills/remotion-markup/sequencing.md` | 1,424 | 1 | **KEEP** |  |
| `skills/remotion-markup/sfx.md` | 1,954 | 2 | **KEEP+** | Lists meme sounds (vine-boom, bruh) that sound-design guidance says to avoid. |
| `skills/remotion-markup/silence-detection.md` | 2,724 | 2 | **KEEP** |  |
| `skills/remotion-markup/text-highlights.md` | 1,978 | 1 | **KEEP** |  |
| `skills/remotion-markup/timing-props.md` | 3,495 | 2 | **KEEP** |  |
| `skills/remotion-markup/timing.md` | 3,446 | 1 | **KEEP** |  |
| `skills/remotion-markup/transitions.md` | 9,163 | 4 | **KEEP** |  |
| `skills/remotion-markup/video-editing.md` | 5,970 | 5 | **KEEP** |  |
| `skills/remotion-markup/voiceover.md` | 4,118 | 2 | **KEEP?** | ElevenLabs TTS path, only valid when the user authorises new speech. |
| **skills/remotion-multimedia** | | | | |
| `skills/remotion-multimedia/SKILL.md` | 838 | 15 | **KEEP?** | Optional; workflow metadata needs are already met by ffprobe in finalize/verify_export. |
| `skills/remotion-multimedia/agents/openai.yaml` | 317 | 1 | **KEEP** |  |
| `skills/remotion-multimedia/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| `skills/remotion-multimedia/get-audio-duration.md` | 1,638 | 2 | **KEEP?** | 85% overlap with get-video-duration.md. |
| `skills/remotion-multimedia/get-video-dimensions.md` | 1,928 | 2 | **KEEP?** |  |
| `skills/remotion-multimedia/get-video-duration.md` | 1,600 | 2 | **KEEP?** |  |
| **skills/remotion-preproduction** | | | | |
| `skills/remotion-preproduction/SKILL.md` | 1,762 | 14 | **MERGE** | Duplicates the "understand and plan" phase of video-director; not linked from the router. |
| `skills/remotion-preproduction/agents/openai.yaml` | 275 | 1 | **MERGE** | Follows the skill. |
| `skills/remotion-preproduction/references/creative-diagnosis.md` | 723 | 0 | **DELETE** | No inbound reference; content restated in production-quality.md. |
| `skills/remotion-preproduction/references/design-rules.md` | 1,605 | 2 | **ROUTE** | Reachable only through remotion-themes and its own skill; it is the one substantive file in this skill. |
| `skills/remotion-preproduction/references/motion-principles.md` | 1,154 | 0 | **DELETE** | No inbound reference; content restated in camera-and-continuity.md. |
| `skills/remotion-preproduction/references/qa.md` | 670 | 0 | **DELETE** | No inbound reference; content restated in production-quality.md and the QA template. |
| `skills/remotion-preproduction/references/script-and-copy.md` | 661 | 0 | **DELETE** | No inbound reference; content restated in video-scriptwriting. |
| `skills/remotion-preproduction/references/storyboard-timing-specs.md` | 1,118 | 0 | **DELETE** | No inbound reference; content restated in plan-schema.md and camera-and-continuity.md. |
| **skills/remotion-render** | | | | |
| `skills/remotion-render/SKILL.md` | 1,093 | 15 | **KEEP** |  |
| `skills/remotion-render/agents/openai.yaml` | 286 | 1 | **KEEP** |  |
| `skills/remotion-render/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| `skills/remotion-render/transparent-videos.md` | 2,398 | 1 | **KEEP?** | Only for alpha delivery. |
| **skills/remotion-sound-design** | | | | |
| `skills/remotion-sound-design/SKILL.md` | 1,516 | 15 | **ROUTE** | Zero inbound links; its five references are never linked from it. |
| `skills/remotion-sound-design/agents/openai.yaml` | 255 | 1 | **KEEP** |  |
| `skills/remotion-sound-design/assets/SfxTrack.tsx` | 614 | 1 | **ROUTE** | Only documented in the unrouted remotion-audio.md. |
| `skills/remotion-sound-design/assets/example-cues.json` | 741 | 0 | **DELETE** | No reference anywhere; schema differs from auto_sfx output and it includes a "CTA" chime example the workflow forbids assuming. |
| `skills/remotion-sound-design/references/mixing-and-loudness.md` | 3,111 | 0 | **ROUTE** | Unique loudness/EQ targets; no inbound path. |
| `skills/remotion-sound-design/references/remotion-audio.md` | 3,110 | 0 | **ROUTE** | Unique ducking/cue-list guidance; no inbound path. |
| `skills/remotion-sound-design/references/sound-design-principles.md` | 1,044 | 0 | **ROUTE** | No inbound path; overlaps SKILL.md. |
| `skills/remotion-sound-design/references/sources-and-licensing.md` | 1,908 | 0 | **ROUTE** | Only description of synth_sfx.py and licence checking; no inbound path. |
| `skills/remotion-sound-design/references/why-it-sounds-bad.md` | 2,642 | 0 | **ROUTE** | No inbound path; overlaps mixing-and-loudness.md. |
| `skills/remotion-sound-design/scripts/audit_audio.py` | 4,232 | 3 | **KEEP** | Called by finalize.py. |
| `skills/remotion-sound-design/scripts/mix_preview.py` | 4,654 | 3 | **KEEP** | Called by auto_sfx.py. |
| `skills/remotion-sound-design/scripts/synth_sfx.py` | 13,799 | 1 | **KEEP?** | Never invoked by the workflow; numpy/scipy dependency; only mentioned in the unrouted licensing file. |
| **skills/remotion-studio** | | | | |
| `skills/remotion-studio/SKILL.md` | 1,744 | 15 | **KEEP** |  |
| `skills/remotion-studio/agents/openai.yaml` | 237 | 1 | **KEEP** |  |
| `skills/remotion-studio/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| **skills/remotion-themes** | | | | |
| `skills/remotion-themes/SKILL.md` | 989 | 15 | **ROUTE** | Zero inbound links: no skill or router points at the Cut Paper/art-direction skill. |
| `skills/remotion-themes/agents/openai.yaml` | 308 | 1 | **KEEP** | Registration only. |
| `skills/remotion-themes/themes/_shared/fonts/Bricolage-OFL.txt` | 4,403 | 0 | **KEEP+** | Required only while Bricolage.ttf ships. |
| `skills/remotion-themes/themes/_shared/fonts/Bricolage.ttf` | 408,496 | 0 | **DELETE?** | 408 KB, half the compressed bundle; never named in any document, so an agent cannot discover it. |
| `skills/remotion-themes/themes/_shared/fonts/Caveat-OFL.txt` | 4,385 | 0 | **KEEP+** | Required only while Caveat.ttf ships. |
| `skills/remotion-themes/themes/_shared/fonts/Caveat.ttf` | 403,648 | 0 | **DELETE?** | 404 KB, never named in any document. |
| `skills/remotion-themes/themes/_shared/fonts/DMSans-OFL.txt` | 4,482 | 0 | **KEEP** |  |
| `skills/remotion-themes/themes/_shared/fonts/DMSans.ttf` | 240,164 | 2 | **KEEP+** | Used only by tools/check_fresh_project.py as a synthetic fixture. |
| `skills/remotion-themes/themes/cut-paper/THEME.md` | 1,094 | 2 | **ROUTE** | Only reachable from the unrouted themes skill. |
| **skills/remotion-upgrade** | | | | |
| `skills/remotion-upgrade/SKILL.md` | 2,390 | 15 | **KEEP+** | `npx skills update ...` still lists the removed `remotion-saas` skill. |
| `skills/remotion-upgrade/agents/openai.yaml` | 282 | 1 | **KEEP** |  |
| `skills/remotion-upgrade/assets/remotion-icon.svg` | 1,303 | 0 | **KEEP** |  |
| **skills/video-director** | | | | |
| `skills/video-director/SKILL.md` | 3,645 | 14 | **KEEP** | Entry point. |
| `skills/video-director/agents/openai.yaml` | 294 | 1 | **KEEP** |  |
| `skills/video-director/assets/plan.template.json` | 1,674 | 4 | **KEEP** | Neutral schema-3 starter. |
| `skills/video-director/assets/qa-report.template.md` | 1,356 | 4 | **KEEP** |  |
| `skills/video-director/profiles/channel.json` | 1,370 | 1 | **KEEP** | Optional sourced profile. |
| `skills/video-director/references/asset-first-workflow.md` | 4,582 | 3 | **KEEP** |  |
| `skills/video-director/references/camera-and-continuity.md` | 1,824 | 5 | **KEEP** |  |
| `skills/video-director/references/current-workflow.md` | 5,214 | 100 | **KEEP** | Canonical architecture. |
| `skills/video-director/references/delivery-workflow.md` | 2,853 | 2 | **KEEP** |  |
| `skills/video-director/references/plan-schema.md` | 3,194 | 3 | **KEEP** |  |
| `skills/video-director/references/production-quality.md` | 2,546 | 3 | **KEEP** |  |
| `skills/video-director/scripts/after_vo.py` | 2,425 | 4 | **KEEP+** | Legacy schema<3 branch and --tail/--music only serve the paused-estimator path. |
| `skills/video-director/scripts/auto_sfx.py` | 3,540 | 7 | **KEEP** |  |
| `skills/video-director/scripts/check_assets.py` | 10,758 | 5 | **KEEP** |  |
| `skills/video-director/scripts/doctor.py` | 2,711 | 0 | **KEEP+** | Zero references in any document or script; environment check is never instructed. |
| `skills/video-director/scripts/finalize.py` | 5,722 | 3 | **KEEP** |  |
| `skills/video-director/scripts/init_video.py` | 2,582 | 6 | **KEEP+** | Hidden `--no-sfx` argument is a 2.x leftover used only by a test. |
| `skills/video-director/scripts/legacy_pause_estimator.py` | 6,451 | 1 | **DELETE** | Previous-version pause estimator; schema 3 explicitly forbids inferring cuts from pauses. Dead unless an old scene plan is re-run. |
| `skills/video-director/scripts/plan_docs.py` | 10,918 | 4 | **KEEP** |  |
| `skills/video-director/scripts/verify_export.py` | 3,535 | 2 | **KEEP** |  |
| **skills/video-scriptwriting** | | | | |
| `skills/video-scriptwriting/SKILL.md` | 2,222 | 15 | **KEEP** |  |
| `skills/video-scriptwriting/agents/openai.yaml` | 230 | 1 | **KEEP** |  |
| `skills/video-scriptwriting/references/hooks.md` | 894 | 1 | **KEEP** |  |
| `skills/video-scriptwriting/references/rewrite-examples.md` | 846 | 0 | **ROUTE** | No inbound link or mention anywhere. |
| `skills/video-scriptwriting/references/script-format-and-checks.md` | 946 | 1 | **KEEP** |  |
| `skills/video-scriptwriting/references/structures-and-formats.md` | 1,007 | 0 | **ROUTE** | No inbound link or mention anywhere. |
| `skills/video-scriptwriting/references/voice-and-craft.md` | 1,316 | 1 | **KEEP** |  |
| **tests** | | | | |
| `tests/asset_fixture.py` | 1,869 | 0 | **KEEP** |  |
| `tests/plan_fixture.py` | 2,240 | 0 | **KEEP** |  |
| `tests/test_decisions.py` | 6,675 | 0 | **KEEP** | 13 tests. |
| `tests/test_pipeline.py` | 10,903 | 0 | **KEEP** | 10 tests; class skipped without ffmpeg. |
| `tests/test_project_assets.py` | 5,079 | 0 | **KEEP** | 11 tests. |
| **tools** | | | | |
| `tools/audit_references.py` | 1,807 | 0 | **KEEP+** | Not listed among the documented verification commands. |
| `tools/check_fresh_project.py` | 5,481 | 2 | **KEEP** |  |
| `tools/package_release.py` | 1,959 | 1 | **KEEP** |  |
| `tools/validate_bundle.py` | 7,074 | 2 | **KEEP+** | Asserts zero `asset_catalog_entries`/`story_asset_packages`, i.e. it enforces absence of deleted v2 features. |

---

## 11. Applied — release 3.1.0

Every recommendation in §3–§5 was executed on request, including removal of the four skills. Result: **184 → 139 files**, **2,001,900 → 500,496 bytes raw**, **973,214 → 231,602 bytes zipped (−76 %)**, **16 → 12 skills**, **99 → 75 Markdown files**. New SHA-256 `86f1681e1499ad643243c9176a50fc178ea73859c802311562080a51955020c1`.

### Removed

| Path | Bytes | Why |
|---|---:|---|
| `PLAN.md` | 1,714 | v1.5.0 plan contradicting the 3.0.0 asset policy (§2.1) |
| `skills/video-scriptwriting/` (SKILL + 5 refs + agent) | 7,461 | superseded by `video-director`; speech guidance folded into its `SKILL.md` |
| `skills/remotion-preproduction/` (SKILL + 6 refs + agent) | 7,968 | superseded planning layer; 5 of 6 refs were duplicates (§2.5) |
| `skills/remotion-themes/` (SKILL + THEME + 3 fonts + 3 notices) | 1,067,969 | art direction lives in `current-workflow.md`; fonts were named nowhere (§5.1) |
| `skills/remotion-sound-design/` (SKILL + 5 refs + 2 assets + agent + `synth_sfx.py`; the other two scripts moved) | 28,740 | sound treatment lives in `delivery-workflow.md`; 5 refs were unrouted, `synth_sfx.py`/`SfxTrack.tsx`/`example-cues.json` unused |
| `skills/remotion-markup/light-leaks.md` | 3,279 | `lightLeak()` already catalogued in `effects.md` (§3.1) |
| `skills/remotion-markup/assets/useProjectFonts.ts` | 623 | referenced only by the test harness; `local-fonts.md` teaches `@remotion/fonts` (§3.3) |
| `skills/video-director/scripts/legacy_pause_estimator.py` | 6,451 | schema-3 forbids inferring cuts from pauses (§2.3) |
| `skills/video-director/scripts/doctor.py` | 2,711 | referenced by nothing anywhere (§3.3) |
| `skills/remotion-maps/techniques/maptiler/assets/sample-data/` | 346,216 | generated geography with unstated provenance (§6.4) |
| `…/maptiler/assets/RiverReveal.tsx`, `example-Root.tsx` | 11,869 | hard-imported the deleted sample data |
| `…/{mapbox,maplibre,maptiler}/references/render-stability.md` ×3 → 1 shared | −7,558 | byte-identical copies (§4.1) |

### Preserved, not deleted — and why

`audit_audio.py` (4,232 B) and `mix_preview.py` (4,654 B) moved from the removed sound skill into `skills/video-director/scripts/`, because the retained pipeline calls them: `finalize.py` loudness-normalizes and audits through `audit_audio.py`, and `auto_sfx.py` builds the preview mix and `sfxCues.ts` through `mix_preview.py`. Deleting the skill without this move would have broken `finalize.py` and five tests. Callers and `tests/test_pipeline.py` were updated; the delivery guidance now states that both scripts ship beside the director's own.

### Code and doc changes

- `after_vo.py`: legacy branch deleted, `--tail`/`--music` dropped, a non-schema-3 plan now exits with a clear message instead of being re-estimated (and no longer auto-emits sound).
- `init_video.py`: undocumented `--no-sfx` removed.
- `video-director/SKILL.md`: speech/audience/opening/causal-clarity guidance folded in where the two removed links were; version 3.1.0.
- `remotion-markup/SKILL.md`: Cut Paper/font sentence replaced with "no family is bundled".
- `delivery-workflow.md`: names `mix_preview.py` / `audit_audio.py` and states no sound library ships.
- `remotion-maps`: one shared `references/render-stability.md`; maptiler docs now describe `prep-geo.mjs` generation instead of deleted demo components; `MapTilerVectorElement.ts` comment fixed.
- `tools/validate_bundle.py`: 12 expected skills, theme/font assertions removed, a guard added so a `themes`/`reusable-assets`/`catalog` folder cannot re-enter, `themes: []` enforced.
- `tools/audit_references.py`: **the tautological routing check is replaced by a real link-graph walk plus filename-mention fallback** — the exact defect from §6.1. The obsolete-token list now also rejects every removed name.
- `tools/check_fresh_project.py`: bundled font removed from the fixture (the compile test's purpose, the missing-caption-style rejection, is unchanged); `public/fonts` assertion dropped.
- `CHANGELOG.md`: 3.1.0 entry added above 1.2.1 (§2.2 fixed). `README.md`, `VALIDATION.md`, `THIRD_PARTY.md`, `RELEASE.json` (3.1.0, `skill_count: 12`, `themes: []`) updated; `FRESH-PROJECT-CHECK.json` and `TREATMENT-EVALUATION.json` keep their 3.0.0 version fields because they record that run. Font/dataset claims in `THIRD_PARTY.md` are now true rather than aspirational (§3.3, §6.4).
- All `studio-reference:3.0.0` markers and the decision-architecture header moved to 3.1.0.

### Verification after the change

```
python3 tools/validate_bundle.py   → PASS: 12 skills; 139 files; links, JSON, Python syntax, artwork policy and manifest
python3 tools/audit_references.py  → PASS: 75 Markdown files; reachability and obsolete-reference checks
python3 -m unittest discover -s tests → Ran 24 tests, 3 errors, 1 skipped
```

The three errors and the skipped class are the ffmpeg/numpy/scipy dependency noted in §6.3, unchanged from the pre-edit baseline (identical 24/3/1 result). `package_release.py` regenerated `MANIFEST.json` and the ZIP, which was re-extracted and re-validated before delivery.

### Deliberately not done

- `asset_catalog_entries` / `story_asset_packages` stay in `RELEASE.json` and `validate_bundle.py`: with the theme inventory gone, they are now the only guard that no reusable visual library re-enters.
- `channel.json` keeps its `cut-paper` medium preference: it records a user request, not a bundled theme.
- The optional upstream skills (`maps`, `multimedia`, `docs`, `upgrade`, `transcribe`/SRT/`display-captions` paths, `voiceover.md`) remain — removing third-party licensed material is a product decision, not hygiene.
