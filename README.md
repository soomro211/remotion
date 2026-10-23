# Remotion Video Studio

## Finalized skills — 1.3.0

**[Download the complete skill ZIP](https://github.com/soomro211/remotion/raw/refs/heads/main/remotion-video-studio-final.zip)**
· [SHA-256 checksum](remotion-video-studio-final.zip.sha256)

One archive containing all 16 skills, **Cut Paper as the only theme**, fonts and
notices, 29 searchable reusable asset entries, editable React/SVG components, current
production references, silent animation previews, helper scripts and checks.
The former map theme, its datasets, dedicated helper and unused fonts are removed
from this release. Use the finalized ZIP; no second archive is needed.

The workflow defaults to topic-specific paper worlds, smooth subject animation,
purposeful camera movement and transitions, accurate smooth subtitles above
bottom controls (default y=1500 at 1080×1920, leaving 272px below the plate),
brief optional opening titles and no persistent chapter headings,
progress bars or scene counters. It includes a jointed, distance-driven walking rig with planted feet and editable
appearance/actions; modular people, hands, scenery, props and explanation tools;
and an edit-preserving selective installer. Current references use the latest
components and lower captions. The Industrial Revolution study remains historical.

### Install

1. Extract the ZIP once.
2. Copy the contents of its `remotion-video-studio/skills/` into your agent's
   skills directory: Codex `~/.codex/skills/` or project `.agents/skills/`, or
   Claude Code `~/.claude/skills/`. Keep all sibling folders together.
3. Review customized local copies before replacing them, then reload the agent
   session if needed. Read the included README for requirements and validation.

Correct layout: `.agents/skills/video-director/SKILL.md` alongside
`.agents/skills/remotion-themes/SKILL.md`.

### Use

> Use video-director. Make a 50-second video about [topic] in the cut-paper theme.

Approve the plan and provide its voice take. The agent carries the production
through alignment, illustration, animation, captions, render and final QA without
requiring each improvement to be requested again. Existing approval carries forward.

The finalized package passed 17 regression tests, TypeScript/ESLint, 14,412 sampled
gait poses, 71 browser frame checks, fresh-source rendering with relocated skills,
encoded-preview decode checks, and clean-extraction/manifest validation. All 100
Markdown references are audited against the current workflow. Source/image hashes
flag stale previews after shared-asset edits.
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
