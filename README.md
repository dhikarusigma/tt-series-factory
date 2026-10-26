# tt-series-factory

A Claude Code skill that makes series of short vertical videos (TikTok / Reels / Shorts) end to end: script, 2D scenes in Remotion, ElevenLabs voice-over, music, subtitles, a render under 29 MB, analytics review and best posting time.

## Example
[`example_part1_webinar.mp4`](example_part1_webinar.mp4) is part 1 of the animated series "The Webinar You Can't Turn Off", made with this workflow. It reached 135k views on TikTok (downscaled to 540x960 here to keep the repository small). Characters are cartoon avatars.

## Install
1. Download this repository (Code → Download ZIP) or clone it into `~/.claude/skills/tt-series-factory/` (Windows: `C:\Users\<name>\.claude\skills\tt-series-factory\`) so that `SKILL.md` sits directly in that folder.
2. Restart Claude Code. The skill is picked up by its description: ask "make a series of videos", "render part 2" or "fix the bugs in the video".
3. You need Node.js 20+, Python 3.12+, ffmpeg and a Remotion project. Details are in `SKILL.md`.

## What is inside
- the process from references and analytics to publishing;
- keeping timings in sync with the real voice-over;
- emotion in ElevenLabs v4 (tags, stability) and audio quality checks;
- a checklist of visual bugs and Windows pitfalls.

Issues and suggestions are welcome.
