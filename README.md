# Remotion Video Studio — 3.1.0

[Download the workflow ZIP](remotion-video-studio-final.zip) · [SHA-256](remotion-video-studio-final.zip.sha256)

This release: 231,602 bytes, 139 files, 12 skills — previously 973,214 bytes, 184 files, 16 skills. Every skill, reference, theme, font and dataset the 3.0.0 routing did not reach is gone; the retained workflow is unchanged. See the [complete audit](AUDIT-3.0.0.md).

The current brief drives creative decisions. User requirements retain their source and scope; technical checks are separate from artistic judgment. Palette, type, setting, cast, narrator, visual method, shot structure, captions and sound are project decisions.

## Architecture
Understand purpose/audience/scope → write speech and visual shots independently → develop contextual art direction → prepare/review project artwork → align speech and assemble → review creativity and engineering separately → deliver.

Colors follow setting, materials, lighting, context, tone and readable hierarchy. References have explicit roles and limits. The starter injects no creative treatment, aspect ratio, duration, scenery, font or sound. Caption styling and sound events are supplied explicitly by the project.

## What ships
`video-director` is the workflow: the decision architecture, project-local asset gate, delivery checks and ten helper scripts. The eleven upstream Remotion skills (`best-practices` router, `create`, `markup`, `studio`, `render`, `captions`, `interactivity`, `maps`, `multimedia`, `docs`, `upgrade`) are optional implementation references.

No font, theme, artwork, sound library or map dataset is bundled. Original artwork stays inside its project.

## What 3.1.0 removed
Removed as unreachable or superseded, all recorded in the ZIP's `CHANGELOG.md`:

- **Skills** `video-scriptwriting`, `remotion-preproduction` (that phase is `video-director`; speech guidance is now stated there), `remotion-themes` (art direction is defined in `references/current-workflow.md`; Cut Paper theme and the three fonts — 812 KB — no longer ship), `remotion-sound-design` (sound treatment is in `delivery-workflow.md`; the two scripts the pipeline actually runs moved to `video-director/scripts/`).
- **References:** five `remotion-preproduction` files, five `remotion-sound-design` files, `light-leaks.md`, `useProjectFonts.ts`, `example-cues.json`, `SfxTrack.tsx`.
- **Stale tooling:** the 1.5.0 `PLAN.md`, `legacy_pause_estimator.py` and the schema<3 branch in `after_vo.py` (a legacy plan is no longer re-estimated), the undocumented `--no-sfx` flag, the unreferenced `doctor.py`, `synth_sfx.py`.
- **Bundled data:** the 333 KB generated `country-meta.json` and its demo components; the three duplicate `render-stability.md` copies became one shared reference.

`tools/audit_references.py` now walks the link graph instead of testing for a boilerplate line, so an unrouted file fails the check.

## Installation
Extract the ZIP, back up customizations and replace the old studio-owned folders in `~/.codex/skills/`, `.agents/skills/`, or `~/.claude/skills/`. Keep unrelated skills and completed films. Delete the four removed skill folders if they are still installed.

Use video-director with the current topic, audience and purpose. The optional `channel` profile carries this user's sourced preferences for paper construction, lower smooth subtitles, purposeful display text and opening attention. It is selected explicitly and contains no palette, font, cast, location or story template.

Schema 3 separates `narration` from `shots`. Voice registration preserves the take without choosing shot cuts. Older project sources/exports remain intact.

## Verification
```bash
python3 tools/validate_bundle.py
python3 tools/audit_references.py
python3 -m unittest discover -s tests -v
python3 tools/package_release.py --out ../remotion-video-studio-final.zip
```

Recorded for 3.0.0: 34 regressions passed, a relocated project compiled and rendered with explicit caption styling, three controlled contextual treatment studies ran at 1x. The 3.1.0 tree passes bundle validation (12 skills, 139 files, manifest), reachability audit (75 Markdown files) and the 34 regression tests (24 collected here; the failures are the 3 tests plus the 10-test pipeline class that require ffmpeg, numpy and scipy, which this environment lacked). Technical tests are not a guarantee of future creative quality; each real film requires its own review.
