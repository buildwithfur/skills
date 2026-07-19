---
name: review-video-keyframes
description: Review short-form videos from ordinary image inputs by combining a transcript when available with keyframes extracted using FFmpeg. Use for TikTok, Reels, Shorts, and reference videos when assessing the hook, pacing, composition, on-screen text, transitions, demonstrations, and payoff without requiring a native-video model or provider-specific API.
---

# Review video keyframes

Keep this workflow portable. Require only `ffmpeg` and `ffprobe`; use the
environment's normal image-inspection capability for the extracted JPG files.
Downloading and transcription are optional helpers, not dependencies.

Before installing anything, tell the user what is missing and why. Typical
package names are `ffmpeg` on macOS, Linux, and Windows package managers.

## Workflow

### 1. Acquire and inspect

Accept a local video or public URL. For a URL, use `$download-social-video` when
available; otherwise ask for a local copy. Inspect the source:

```sh
ffprobe -v error -show_streams -show_format '<video.mp4>'
```

Use existing subtitles or captions when available. If a transcription tool is
already present, it may be used. Do not block visual review when no transcript
can be obtained; clearly state that spoken-content analysis is limited.

If the source has no useful audio or subtitles and communicates through
burned-in text, treat ordered frames as the primary content rather than merely
visual evidence. Sample every 2 seconds initially, identify slide/text changes,
then inspect denser frames only around transitions. Reconstruct text in
chronological order and distinguish exact readable text from paraphrase.

### 2. Extract representative frames

```sh
mkdir -p /tmp/video-review/frames
ffmpeg -i '<video.mp4>' \
  -vf "select='gt(scene,0.3)',showinfo" \
  -fps_mode vfr -q:v 2 \
  /tmp/video-review/frames/frame_%03d.jpg
```

Start around `0.3`. Try `0.2` for slow talking-head footage or `0.4` for a fast
montage. If scene detection produces too few useful frames, add fixed-interval
samples every 3–5 seconds. Preserve timestamps using `showinfo`, filenames, or a
simple manifest.

### 3. Inspect selectively

Inspect 3–6 representative JPGs first:

- Hook in the first 1–3 seconds
- First visible evidence or demonstration
- Important transitions
- Key proof or instructional moment
- Final payoff or call to action

Inspect more frames only when a cut changes the claim, text is unreadable, or
the transcript and visuals appear misaligned.

### 4. Review with evidence

For each important moment, record the timestamp, visible subject, composition,
text legibility, narrative role, and match with the spoken line when known.

Do not infer screen content that cannot be read. Mark it as unclear and request a higher-resolution source if it materially affects the review.

Judge:

- hook clarity and specificity
- structure: hook → evidence/demo → takeaway
- whether claims are visibly supported
- pacing and whether cuts improve comprehension
- caption and overlay legibility at phone size
- whether the ending delivers the promise

Return a concise, prioritized review with exact timestamps. Separate observed
evidence from recommendations, and state whether transcript analysis was
available.

## Guardrails

- Do not require Gemini, Qwen, DeepSeek, or another provider-specific service.
- Do not assume the host accepts video input.
- Do not review every frame unless the task genuinely requires it.
- Do not treat a thumbnail or poster as a source-video keyframe.
- Do not infer missing words from blurry or occluded burned-in text.
- Do not claim knowledge of retention or audience behavior without analytics.
