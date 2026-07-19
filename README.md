# Reusable agent skills

Reusable agent skills for browser-based human handoffs and practical social-video
research workflows.

## Available skills

### Browser and desktop handoff

- `setup-isolated-browser-desktop` creates a shared Linux browser desktop that
  the user and agent can operate in the same session.
- `setup-macos-web-screen-sharing` exposes the user's real Mac session through a
  browser, preserving access to local profiles, extensions, password managers,
  and native apps.

### Social-video research

- `download-social-video` downloads public TikTok, Instagram video/Reel, and
  YouTube videos, prioritising original-language captions before transcription.
- `download-instagram-carousel` retrieves the original assets from a public
  Instagram carousel in its true slide order.
- `review-video-keyframes` combines a transcript with scene-detected keyframes
  to review hooks, pacing, proof, composition, and payoff in short-form video.

## Install with the skills CLI

Node.js and npm are required. Choose skills interactively:

```bash
npx skills add buildwithfur/skills
```

List the available skills without installing:

```bash
npx skills add buildwithfur/skills --list
```

Install a specific skill globally for Codex:

```bash
npx skills add buildwithfur/skills \
  --skill download-social-video \
  --agent codex \
  --global
```

Install every skill:

```bash
npx skills add buildwithfur/skills \
  --skill '*' \
  --agent codex \
  --global
```

Without `--global`, the CLI installs into the current project.

## Install from inside Codex

Ask Codex to install a skill by its GitHub folder:

```text
$skill-installer install https://github.com/buildwithfur/skills/tree/main/<skill-name>
```

Start a new Codex session after installation.
