# Remotion Video Studio — 1.6.0

**[Download the finalized skill ZIP](https://github.com/soomro211/remotion/raw/refs/heads/main/remotion-video-studio-final.zip)** · [SHA-256](remotion-video-studio-final.zip.sha256)

16 skills, Cut Paper as the sole theme, **171 editable asset/tool entries** with searchable PNG/SVG
previews, local fonts, eight silent studies and source/render checks.

This release adds **65 entries**: 13 buildings, 12 locations, 10 road/floor surfaces, 16 equipment
and everyday props, four human interactions, a complete pouring scene, six effects and three
movement/transition tools. It repairs gestures, wrist orientation, pouring, the bed, airplane and
basic shop/school/hospital façades. New locations include city, industry, interiors, harbor, coast,
desert, canyon and winter. Appearances are controls, not inflated counts.

Lift/place/push/pull share hand/object contacts. Pouring follows the moving spout, keeps a level
source surface, delays received volume and respects the receiver rim. Effects have deterministic,
finite lifetimes. Walking retains planted feet and wheels follow distance.

## Install or update

1. Extract the ZIP; back up customized installed skills.
2. Merge saved installed story packages into the extracted library with `asset_workflow.py sync`
   before replacement. The included asset-first guide gives exact commands.
3. Copy `remotion-video-studio/skills/` into Codex `~/.codex/skills/`, project `.agents/skills/`,
   or Claude Code `~/.claude/skills/`. Keep sibling folders and the merged archive together.

## Use

> Use video-director. Make a video about [topic] in the cut-paper theme.

The workflow plans a human script/storyboard, then creates/reviews distinctive editable art and
saves it in the **main skill set's assets/library/** before scene assembly. Topic-specific art stays
there too. Existing authorization carries forward; standalone assets need no narration. Future
agents can restore source without the original videos, and updates preserve saved additions.

Defaults include smooth subject action, motivated camera/transitions, varied settings, original
voice preservation, actual-audio caption timing and lower subtitles. At 1080×1920 the caption plate
uses x80/y1500/w890/h148. No persistent chapter headings, counters, progress UI or forced outro.

```bash
python3 skills/remotion-themes/scripts/assets.py list --search factory
python3 skills/remotion-themes/scripts/assets.py copy /path/to/video --assets factory,industrial-yard,forklift,lift,pouring,steam,camera,captions
python3 skills/remotion-themes/scripts/asset_workflow.py search --query river
```

The ZIP includes `VALIDATION.md`, current source-generated references and a clean-project rendering
check. Passing technical checks does not guarantee a future film's artistic quality or facts; each
film still requires actual review. Private films, voice takes, credentials and caches are excluded.
