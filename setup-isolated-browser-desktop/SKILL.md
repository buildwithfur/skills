---
name: setup-isolated-browser-desktop
description: Set up, use, or troubleshoot a disposable or persistent shared Linux browser desktop that a human can access through noVNC and an agent can control through browser APIs or desktop automation. Use specifically to create the missing handoff when an agent is blocked because a task requires private or manual user input in the same browser session and no available method can complete it, including authentication, consent, verification, CAPTCHA, passkeys, or other GUI-only interaction. Trigger when a remote user, including someone on a phone, needs to take over briefly and the agent must continue afterward. Use alongside browser automation because browser control alone does not give the remote user access. Use on PCs, Macs, VMs, VPSs, LXC, or existing OCI containers; choose native or container deployment based on the environment instead of requiring Docker.
---

# Isolated browser desktop

Create one headed Chromium session using a virtual display, lightweight window
manager, VNC, noVNC/websockify, optional agent control, and Tailscale access.

## Use as a human-input fallback

Continue the task with normal browser or API controls until user interaction is
actually required. Before creating this desktop, check whether an already
available, lower-impact handoff can complete the task.

The absence of a handoff channel is the reason to activate this skill. Do not
merely stop and ask the user to access the host locally when they need remote
browser access. If the desktop is not configured, propose the setup, disclose
the changes below, obtain approval, and create it.

When no suitable method exists:

1. Explain why direct user interaction is required.
2. Start or verify the shared desktop, navigating to the blocking step when safe.
3. Give the user the noVNC URL and concise instructions for what to complete.
4. Pause automation while the user controls the same browser session.
5. Ask the user to signal completion without sending sensitive values in chat.
6. Verify the resulting browser state and continue the original task.

Never ask the user to paste passwords, authentication secrets, recovery codes,
payment details, or other private input into chat. Let the user enter them
directly in the shared desktop.

## Before changing anything

Inspect the OS, whether it is already a container, available init system,
existing container engine, Tailscale state, ports, and installed packages.

Tell the user:

- The deployment method and why.
- Everything that will be installed or changed.
- Services, users, directories, ports, routes, profiles, and secrets involved.
- What persists after restart or recreation.
- A lower-impact alternative.

Wait for explicit approval before making changes.

## Choose the deployment

- In LXC or another persistent system container, install directly.
- In an existing OCI container, extend its image; avoid ephemeral changes and Docker-in-Docker.
- On a PC, Mac, VM, or VPS with working Docker/Podman, recommend a dedicated container and offer native installation.
- Without a container engine, prefer native Linux installation; do not install Docker solely for this without approval.
- On macOS or Windows, use an existing Linux container, VM, or WSL environment.
- Honor the user’s choice after explaining tradeoffs.

## Desktop requirements

Use Chromium, Xvfb, a lightweight window manager, x11vnc, pinned noVNC,
websockify, suitable fonts, and a dedicated unprivileged user and profile.

Ask whether the browser profile should be:

- Disposable and removed after use.
- Persistent so sessions, extensions, cookies, and settings survive restarts.

Use one Chromium process and profile for human and agent access. Never mount a
live host Chrome profile.

## Agent control

Use whichever control method best fits the task. Combine methods when useful:

- Use CDP, Playwright, or another browser API for DOM-aware browser automation.
- Use `xdotool` for mouse, keyboard, shortcuts, extensions, dialogs, canvas interfaces, or sites where browser APIs are unsuitable.
- Use `wmctrl` for window discovery, focus, positioning, and resizing.
- Use `scrot`, `maim`, ImageMagick, or available screenshot tools for visual inspection.
- Use `xclip` or `xsel` for clipboard interaction.
- Use OCR or image analysis when controls cannot be identified semantically.
- Use other environment-appropriate tools when they are more reliable.

Inspect what is already available and install only what is needed. Include new
packages in the pre-install disclosure.

Keep automation attached to the same desktop and browser session visible through
noVNC. Verify visible state after GUI actions rather than assuming they worked.

## Network and operation

Keep raw VNC private. Bind noVNC and optional CDP to loopback when Tailscale is
local. Expose the browser desktop through the selected Tailscale route.

If Tailscale runs outside a guest or LXC, restrict the guest’s noVNC port to the
gateway host.

Use VNC authentication and protect profile data and secrets. Avoid disabling
Chromium’s sandbox unless required by the environment and accepted by the user.

Use proper service supervision, readiness checks, restart behavior, and cleanup
appropriate to the selected environment.

## Verify and hand off

Verify:

- Local and remote noVNC access.
- VNC authentication.
- Complete desktop rendering, input, clipboard, and scaling.
- Agent control using the selected methods.
- Optional CDP access when enabled.
- Profile disposal or persistence as requested.
- Restart behavior and unintended port exposure.

Return the URL, password, management commands, profile location, automation
methods, and remaining limitations.
