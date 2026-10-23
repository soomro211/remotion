# Remotion Video Studio

## Finalized skills — 1.2.0

**[Download the complete skill ZIP](https://github.com/soomro211/remotion/raw/refs/heads/main/remotion-video-studio-final.zip)**
· [SHA-256 checksum](remotion-video-studio-final.zip.sha256)

One archive containing all 16 skills, **Cut Paper as the only theme**, fonts and
notices, theme previews, approved production references, helper scripts and checks.
The former map theme, its datasets, dedicated helper and unused fonts are removed
from this release. Older split ZIP uploads are historical; use the finalized ZIP.

The workflow defaults to topic-specific paper worlds, smooth subject animation,
purposeful camera movement and transitions, accurate smooth subtitles above
bottom controls, brief optional opening titles and no persistent chapter headings,
progress bars or scene counters. It includes concrete camera/continuity patterns
from the approved Industrial Revolution film and an evidence-based QA checklist.

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

The finalized package passed eight regression tests, TypeScript/browser smoke
checks, local-link and asset checks, and clean-extraction/manifest validation.
Detailed evidence and limitations are in its `VALIDATION.md`. Good instructions
support better work; each new film still requires visual judgment and render review.

## Production reference

[Industrial Revolution video](videos/industrial-revolution.mp4)
· [Subtitles](videos/industrial-revolution.srt)
