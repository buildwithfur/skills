---
name: download-social-video
description: Download and inspect a single public TikTok, Instagram video/Reel, or YouTube video for review, transcription, summarization, editing, or archiving. Use when a user shares a social-video link and asks what it says or contains, even if they do not explicitly request a download. Check original-language subtitles before transcription, identify carousels and burned-in-text videos, and report access restrictions honestly.
---

# Download social video

Treat a shared social URL and the user's question as a fresh source-specific
task. Acquire the source or captions before answering from it. Do not infer the
content from its URL, thumbnail, or surrounding conversation.

## Prepare

1. Confirm that the URL is an individual post or video, not a profile/channel.
2. Check that `yt-dlp`, `ffmpeg`, and `ffprobe` exist before relying on them.
3. Tell the user what is missing and why before installing software.
4. Keep output in a task-specific directory. Do not overwrite unrelated files.

Never declare a link inaccessible before attempting the primary public
extraction path once. If it fails, report the command category and exact error.

## 1. Check captions, then download

Always inspect subtitles before extracting audio or installing transcription
software:

```sh
yt-dlp --list-subs '<video-url>'
```

Prefer the spoken/original-language track. Do not silently choose an English
translation. If several plausible original tracks exist and the choice matters,
ask the user.

Download the source:

```sh
mkdir -p /tmp/social-video

yt-dlp \
  --no-playlist \
  --merge-output-format mp4 \
  -o '/tmp/social-video/%(extractor)s_%(id)s.%(ext)s' \
  '<video-url>'
```

If the user specifies a destination, use it instead of `/tmp/social-video`.

### Download available subtitles

**TikTok** — use `--all-subs`; `--write-auto-subs` can hang:

```sh
yt-dlp --no-playlist --write-subs --all-subs --sub-format vtt --skip-download \
  --extractor-args 'tiktok:api_hostname=www.tiktok.com' \
  -o '/tmp/social-video/%(extractor)s_%(id)s.%(ext)s' \
  '<tiktok-url>'
```

**YouTube** — use the original language code shown by `--list-subs` (often `en-orig` rather than `en`):

```sh
yt-dlp --no-playlist --write-subs --write-auto-subs \
  --sub-lang '<original-language-code>' --sub-format vtt --skip-download \
  -o '/tmp/social-video/%(extractor)s_%(id)s.%(ext)s' \
  '<youtube-url>'
```

For a summary, read a short VTT directly; timestamps rarely obstruct
comprehension. Deduplicate and clean it only when the user needs a polished
transcript or saved text. YouTube auto-captions are more likely than TikTok
captions to contain repeated rolling lines.

## 2. Classify the fallback

When captions are unavailable, inspect the downloaded media:

```sh
ffprobe -v error -show_streams -show_format '<video.mp4>'
```

Choose one path:

- **Audio exists:** transcribe it with an already-available speech-to-text tool.
- **No audio, but visible text exists:** extract frames and use
  `$review-video-keyframes` to reconstruct the burned-in text.
- **Audio and burned-in captions both exist:** prefer transcription for
  completeness, then use frames only where the visuals add information.
- **No usable video was downloaded:** follow the access-failure guidance below.

### Transcribe audio

```sh
# Extract audio from the downloaded file.
ffmpeg -i '<video.mp4>' -vn -c:a libmp3lame /tmp/social-video/audio.mp3

# Example Whisper command; select a model appropriate to the environment.
whisper /tmp/social-video/audio.mp3 --model tiny --language '<language>' \
  --output_format txt --output_dir /tmp/social-video
```

For short clips, start with `tiny`. Retry with `base` only if accuracy is clearly poor.

### Inspect burned-in text

For a text-led video with no subtitles or useful audio:

```sh
mkdir -p /tmp/social-video/frames
ffmpeg -i '<video.mp4>' -vf "fps=1/2" -q:v 2 \
  /tmp/social-video/frames/frame_%03d.jpg
```

Inspect a sparse sequence first, then zoom into slide changes or unreadable
overlays. Do not analyse every frame by default.

## Platform-specific handling

### TikTok photo posts

A TikTok short URL may resolve to `/photo/` instead of `/video/`. This is a
carousel, so use `$download-instagram-carousel` only if it explicitly supports
the source platform; otherwise use a platform-appropriate image downloader. A
portable fallback is:

```sh
uvx --from gallery-dl gallery-dl -d /tmp/tiktok-carousel '<tiktok-url>'
```

Review the downloaded slides in order. Do not force a transcript workflow onto
a post whose meaning lives in images.

Do not pass a TikTok profile URL to `yt-dlp`; request or locate an individual
post URL instead.

### Instagram restrictions

Private, age-gated, deleted, region-restricted, or login-required Reels can return an empty media response even when they work in the user's own logged-in app.

An `Instagram sent an empty media response` error can mean the post is gated or
the extractor is outdated; it is not proof that the post does not exist. If
`yt-dlp` reports that it is outdated, tell the user, update it through its
existing package manager with approval, and retry once. Do not cycle through
deprecated Instagram endpoints.

After one public attempt, report the actual error and use only capabilities the
current environment explicitly exposes. Safe options include:

- The user sends the downloaded video file directly.
- A user-controlled authenticated browser is used for visible review.
- An authenticated download mechanism is used only when the environment
  explicitly permits it and the user authorized that mechanism.

Do not ask the user to paste passwords, session cookies, or authentication
tokens into chat. Do not extract or decrypt a browser cookie database unless
that capability is explicitly authorized by the environment and user.

## Verify and report

```sh
# Check that the file is a playable video and has non-zero duration.
ffprobe -v error -show_entries format=duration,size -of default=nw=1 \
  /tmp/social-video/<downloaded-file>.mp4
```

Report:

- saved path and media type
- duration and file size
- caption language and whether it is original or translated
- transcript status
- whether content was reconstructed from burned-in text
- each relevant access failure and the fallback used

Never fabricate a transcript, claim a download succeeded when it did not, or
substitute a poster/thumbnail for the source video.
