# Remotion Video Studio — 1.5.0

**[Download the finalized skill ZIP](https://github.com/soomro211/remotion/raw/refs/heads/main/remotion-video-studio-final.zip)** · [SHA-256](remotion-video-studio-final.zip.sha256)

One package containing all 16 skills and Cut Paper as the sole theme. It includes **106 editable
asset/tool entries**, with **77 new universal additions** across everyday objects, transport,
technology, nature, food, buildings, people, data and useful motion. Source-generated PNG/SVG
previews, a searchable category catalog, four silent studies, local fonts and checks are included.

New controls cover hinges/opening, scissor pivots, clock hands, growth, expressions, gestures,
pouring, arc transfer, path drawing, stagger, finite attention/settling and distance-linked wheels.
Walking retains planted feet. Charts need caller-provided data; illustrative tools are not simulators.

The workflow creates/reviews distinctive art and saves it in the **main skill set's**
`remotion-themes/themes/cut-paper/assets/library/` before scene assembly. Topic-specific packages
stay there too, with editable source, dependencies, previews and review evidence. A fresh agent
needs only the updated skill set. Versioned snapshots, restore and conflict-preserving sync protect
reuse across videos and future skill/Git updates.

## Install or update

1. Extract the ZIP and back up customized installed skills.
2. Merge new installed story packages into the extracted library with `asset_workflow.py sync`
   before replacement; the included asset-first guide gives exact commands.
3. Copy the contents of `remotion-video-studio/skills/` into Codex `~/.codex/skills/` or project
   `.agents/skills/`, or Claude Code `~/.claude/skills/`. Keep sibling folders and the merged archive
   together. Reload the agent if required.

## Use

> Use video-director. Make a video about [topic] in the cut-paper theme.

One existing script/storyboard approval, then supplied voice, assets, actual-audio alignment,
scene assembly, smooth animation/camera/transitions/captions, render and QA. Existing authorization
carries forward. Standalone asset work needs no narration. No persistent chapter headings,
counters, film progress UI or unrequested outro. Lower subtitles default to x80/y1500/w890/h148
at 1080×1920; the original voice keeps its words, start and speed.

```bash
python3 skills/remotion-themes/scripts/assets.py list --search transport
python3 skills/remotion-themes/scripts/assets.py copy /path/to/video --assets person,journey,mug,car,arc-transfer,camera,captions
python3 skills/remotion-themes/scripts/asset_workflow.py search --query river
```

The four main workflow/quality documents are 64% shorter, with one shared production authority.
Obsolete topic-specific screenshots, the old drawing generator, deliberately bad walker, duplicate
plans and report prose are removed. Useful API references, source licenses and saved artwork remain.
The archive checks now cover CSS/glTF dependencies, finite dimensions and still-first previews;
angle wrapping, coincident limb targets, clipped contents and bounded motion endpoints are fixed.

This revision passes 33 Python regression tests, TypeScript/ESLint, 14,412 gait samples, 912 timed
motion samples, 424 catalog-state instances, browser contact/ID/caption checks, fresh-source
restoration/rendering, encoded-study decode and clean-package integrity checks. The included
`VALIDATION.md` records actual scope/limits. Each future film still needs factual and visual review.
