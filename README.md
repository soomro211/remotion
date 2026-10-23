# Remotion Video Studio

## Finalized skills — 1.4.0

**[Download the complete skill ZIP](https://github.com/soomro211/remotion/raw/refs/heads/main/remotion-video-studio-final.zip)**
· [SHA-256 checksum](remotion-video-studio-final.zip.sha256)

One archive containing all 16 skills, **Cut Paper as the only theme**, fonts and
notices, 29 searchable reusable asset entries, editable React/SVG components, current
production references, silent animation previews, helper scripts and checks.
It now requires an asset-first stage and includes a permanent, searchable story-art archive
inside the main skill set, with editable layered sources, previews and review evidence.
The former map theme, its datasets, dedicated helper and unused fonts are removed
from this release. Use the finalized ZIP; no second archive is needed.

The workflow creates and reviews distinctive layered assets, then saves them in
`remotion-themes/themes/cut-paper/assets/library/` before assembling scenes.
Topic-specific artwork is preserved there too, so future agents need only the
updated skill set. Each project uses a versioned snapshot.

Other defaults include topic-specific paper worlds, smooth subject animation,
purposeful camera movement and transitions, accurate smooth subtitles above
bottom controls (default y=1500 at 1080×1920, leaving 272px below the plate),
brief optional opening titles and no persistent chapter headings,
progress bars or scene counters. It includes a jointed, distance-driven walking rig with planted feet and editable
appearance/actions; modular people, hands, scenery, props and explanation tools;
and an edit-preserving selective installer. Current references use the latest
components and lower captions. The Industrial Revolution study remains historical.

### Install

1. Extract the ZIP once.
2. Before replacing installed skills, back up customizations and merge any new
   saved packages into the extracted archive with `asset_workflow.py sync`.
   The included asset-first guide gives the exact update commands.
3. Copy the contents of `remotion-video-studio/skills/` into Codex `~/.codex/skills/`
   or project `.agents/skills/`, or Claude Code `~/.claude/skills/`. Keep the sibling
   folders and merged asset library together. Reload the agent if required.

Correct layout: `.agents/skills/video-director/SKILL.md` alongside
`.agents/skills/remotion-themes/SKILL.md`.

### Use

> Use video-director. Make a 50-second video about [topic] in the cut-paper theme.

Approve the plan and provide its voice take. The agent carries the production
through asset creation/review/shared archiving, alignment, scene assembly, animation,
captions, render and final QA without
requiring each improvement to be requested again. Existing approval carries forward.

This revision passed 29 regression tests, including archive preservation, restoring
assets after deleting the original project, conflict-safe sync and readiness enforcement.
A fresh project imported and rendered saved artwork from relocated skills. All 103
Markdown references/asset records are inventoried; source/image hashes flag stale
previews. The unchanged core rig/camera/caption sources retain their earlier
measured motion/browser evidence.
Detailed evidence and limitations are in its `VALIDATION.md`. Good instructions
support better work; each new film still requires visual judgment and render review.

### Reuse assets

After extracting, open the bundled visual catalog, inspect the current theme images,
and search before rebuilding common assets:

```bash
python3 skills/remotion-themes/scripts/assets.py list --search walking
python3 skills/remotion-themes/scripts/assets.py copy /path/to/video --assets person,journey,tree,camera,captions
```

Read the included asset recipes for units, anchors, held items, source time and
physical contact. The small asset-review studio and four silent previews are
included in the same ZIP. Local component edits are protected during installation.

### Create and preserve story artwork

The single ZIP contains the growing permanent archive, including the layered river-valley
source example. Its packages carry editable source, dependency files, previews, metadata
and review evidence. They restore without access to their original films.

```bash
python3 skills/remotion-themes/scripts/asset_workflow.py search --query river
python3 skills/remotion-themes/scripts/asset_workflow.py init /path/to/video
python3 skills/remotion-themes/scripts/asset_workflow.py new /path/to/video --id river-cross-section
# Create and inspect the source/proofs, then complete the package review.
python3 skills/remotion-themes/scripts/asset_workflow.py archive /path/to/video --package assets/packages/river-cross-section/1.0.0 --requirement hero
python3 skills/video-director/scripts/check_assets.py --project /path/to/video
```

Save reviewed improvements as new versions. Before updating Git/skills, merge any additions
from installed `assets/library/` into the checkout; the package tool also accepts
`--merge-library /path/to/installed/library`. The included workflow/reference guide explains
restoring, adapting and syncing. Assets saved only in individual video projects do not
complete the workflow. Existing plan approval remains the only routine user gate.
