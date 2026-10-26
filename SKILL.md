---
name: tt-series-factory
description: Builds series of short vertical videos (TikTok/Reels/Shorts) end to end with Claude alone: script, 2D scenes in Remotion, ElevenLabs voice-over, music and sound effects, subtitles, a final mp4 under 29 MB, plus analytics review and best posting time. Use when asked to "make a series", "build scenes for the voice-over", "render part N", "fix the bugs in the video" or "pick a posting time".
---

# Short-series factory (Remotion + ElevenLabs + ffmpeg)

This skill describes how we built two animated series with Claude Code: "The Webinar You Can't Turn Off" (the first episode reached 135k views on TikTok) and "Solar Storm". Swap the heroes, the genre and the texts and the process stays the same. A rendered example of the first episode is in this repository: `example_part1_webinar.mp4`.

## 0. What you need on the computer
- Claude Code (desktop app, Code tab), Windows or Mac.
- Node.js 20+, Python 3.12+, ffmpeg on PATH.
- A Remotion (React) project. Minimum: `remotion`, `@remotion/cli`, `@remotion/renderer`. For 3D: `three`, `@react-three/fiber`, `@remotion/three`.
- Optional connections: the ElevenLabs connector (voice-over, voice search), the Claude in Chrome extension (signs in to websites with your own account), the in-app Browser pane.
- Claude's memory folder (`memory/`): Claude writes the owner's taste and decisions there. Ask it to save every correction ("remember that I dislike X").

Layout the process relies on (names are free to change):
```
project/
  src/series/        scenes (Sun1.tsx, Sun2.tsx), timings (sun1_tl.ts), character rig (flat.tsx), looks.ts
  tools/stills.mjs   frames for review:   node tools/stills.mjs <Comp> <outDir> 100,400,800 --scale=0.35
  tools/render.mjs   render:              node tools/render.mjs <Comp> final out/x.mp4   (picks CRF to stay under 29 MB)
  public/            music, logo, sfx
scripts/             screenplays (.md), voice/, music generator (mk_music_sun.py), mixing
```

## 1. Workflow for a series
1. **Reference and analytics.** Ask the owner for 2-3 reference videos and screenshots of analytics (retention, views per part). Draw conclusions from data: what hooks in the first seconds, where the drop is.
2. **Story.** A bright, strange or dangerous image in an ordinary setting plus a real plot. Promise the outcome in the first 3 seconds. Chapters, "Rule N", a countdown clock. No recycled ideas, no "we have seen this". 2-3 heroes, not six.
3. **Script.** Short lines, one thought each. Hook: time + consequence ("At 07:35 the sky over the school will light up, at 07:41 the city goes dark..."). Check safety facts against official sources.
4. **Timing file** `sunN_tl.ts`: array of lines, estimated length `chars/13.5*24` frames, a `REAL` map for real durations, `SUN*_S` (starts), `SUN*_D` (durations), `SUN*_FRAMES`. Every scene is computed from it.
5. **Scenes.** Split into scenes A-F at `SC` boundaries. Each has a detailed background, characters in an ordinary place, plates/rules, a HUD countdown, subtitles, sounds.
6. **Music** is procedural (numpy): drone, ticking, heartbeat, a build-up to the climax. The climax is bound to a line index.
7. **Voice-over**, then rebuild from the real durations (section 3).
8. **Review**: contact sheet of frames, then batch fixes (section 4).
9. **Render** under 29 MB, send to the owner, publish (section 5).

## 2. Characters (flat rig)
- Profile body; head with emotions `emo`: `calm/talk/worry/shock/smile/angry/think`. A neutral mouth is a straight line. Beards and caps are set in `looks.ts` (`beard`, `bigBeard`, `cap`, `capBack`, `capColor`).
- Body modes: `stand`, `point`, `walk`. Do not use `sit`: the character looks like lying on the floor.
- A hero is recognised by 2-3 traits (bandana, backwards cap, beard), not by a complicated face.
- Do not draw a front-facing face on a profile body; we tried and rolled it back.
- Draw long hair as side locks, not a solid panel behind the head: otherwise it shows under the chin and looks like a beard.

## 3. Voice-over (ElevenLabs)
- Free account: library voices are not available through the API, only on the website. Check `check_subscription` for the character limit and the reset date. A paid plan removes both limits.
- On the website: open Text to Speech, pick the voice, paste one line, Generate. One file per line. Download files only after the owner allows it.
- **Emotion** (Eleven v4): tags in square brackets before the phrase: `[calm, ominous]`, `[urgent]`, `[whispers]`, `[sighs]`; an ellipsis adds a pause and weight; CAPITAL letters add emphasis; tags are free text, combine two. Lower stability (Creative) gives more emotion. Speed 0.7-1.2. v4 has no `<break>` tags.
- **Normalisation:** trim edges with `silenceremove`, `loudnorm I=-18`, mono 44.1 kHz. The real length of each line in frames (`ceil(sec*24)`) goes into `REAL`.
- Then rebuild the music (`python scripts/mk_music_sun.py <part> <climax_index>`) and mix the voice on top (ffmpeg, two passes).

## 4. Quality checks: how we catch bugs
The owner watches like a viewer and nitpicks the same things every time. Check them yourself:
- **Contact sheet.** Run `stills.mjs` on key frames (step 100-120), tile them into a grid and look at every scene.
- **Look for:** text or plates covering faces; captions longer than 3 lines; objects out of the frame; objects floating in the air (desks and monitors must stand on the floor); empty backgrounds; identical smiling faces without emotion; a "static PNG ball" instead of animation; labels cut at the edge.
- **Animation.** Waves and flashes start from the centre of the source. Effects must live: noise, flicker, pulse.
- **Fix in batches.** Collect all remarks, fix, render once. After a complaint, look for the same bug across the whole video, not only in the named scene.
- **Measure, do not guess.** Compare against a reference frame instead of "looks fine".

## 5. Publishing and analytics
- Open "Analytics → Followers → Most active times" (hours and days). Post 1-2 hours before the peak to collect the first views. For a school audience that is usually weekdays 15:30-16:30 and weekends 11:30-12:30 local time, but account data beats generic advice.
- Never publish several parts at once. End a part with a short card "PART N · COMING SOON", not "already on my profile".
- Series continuations often flop: part 1 gets reach, then views drop. Make every part a story with its own outcome.

## 6. Windows pitfalls
- Bash is Git Bash while Python and Node are native Windows: `/tmp` is not visible to them. Keep temporary files in `%TEMP%` via `cygpath -w`.
- Do not patch complex files with heredoc Python containing `\U` and quotes: use Edit, or write the script to a file.
- Do not run renders in parallel. To stop a render find the `tools/render.mjs` process by its command line; never kill every `node.exe`.
- If a render exceeds 29 MB the renderer raises CRF itself; do not cut the duration without agreement.

## 7. Talking to the owner
Short, to the point, with numbers. Admit mistakes plainly. Offer nothing extra and do not ask again about things already decided. Fix first, then a short report: "what I fixed / what is left".

## 8. Voice-over notes from practice
- Tags such as `[firm]`, `[grave]`, `[low, quiet voice]` on a cloned voice (PVC) sometimes produce a dull or hissy sound, "like a cheap microphone". For rule lines write no tags and keep stability near 0.5.
- A check without listening: file spectrum via ffmpeg + numpy (energy share above 6 kHz, 99% roll-off). It is only a hint because the words change the spectrum; the final call is by ear.
- With a free ElevenLabs account library voices are web-only; watch the remaining character count.
