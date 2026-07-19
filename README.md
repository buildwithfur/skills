# Build With Fur skills

Remote desktop skills that give a user browser-based access when an agent needs
private or manual GUI input and no existing handoff method can complete the task.

## Available skills

- `setup-isolated-browser-desktop` creates a shared Linux browser desktop that
  the user and agent can operate in the same session.
- `setup-macos-web-screen-sharing` exposes the user's real Mac session through a
  browser, preserving access to local profiles, extensions, password managers,
  and native apps.

## Install with the skills CLI

Node.js and npm are required. Choose skills interactively:

```bash
npx skills add buildwithfur/skills
```

List the available skills without installing:

```bash
npx skills add buildwithfur/skills --list
```

Install one globally for Codex:

```bash
npx skills add buildwithfur/skills \
  --skill setup-isolated-browser-desktop \
  --agent codex \
  --global
```

```bash
npx skills add buildwithfur/skills \
  --skill setup-macos-web-screen-sharing \
  --agent codex \
  --global
```

Install both:

```bash
npx skills add buildwithfur/skills \
  --skill '*' \
  --agent codex \
  --global
```

Without `--global`, the CLI installs into the current project.

## Install from inside Codex

Ask Codex to install either GitHub folder:

```text
$skill-installer install https://github.com/buildwithfur/skills/tree/main/setup-isolated-browser-desktop
```

```text
$skill-installer install https://github.com/buildwithfur/skills/tree/main/setup-macos-web-screen-sharing
```

Start a new Codex session after installation.
