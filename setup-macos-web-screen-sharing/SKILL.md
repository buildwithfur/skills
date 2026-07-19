---
name: setup-macos-web-screen-sharing
description: Set up, use, or troubleshoot browser access to a real Mac desktop using macOS Screen Sharing, noVNC/websockify, and Tailscale. Use specifically to create the missing handoff when an agent is blocked because a task requires private or manual user input and no available method can complete it, especially when the task depends on the user's existing Mac session, Chrome profile, password manager, extensions, Keychain, or native apps. Trigger when a remote user, including someone on a phone, needs to take over briefly and the agent must continue afterward. Use alongside Chrome or computer control because agent control alone does not give the remote user access. Support native or container gateways without requiring a native VNC client.
---

# macOS web screen sharing

Create:

Phone browser → Tailscale → noVNC/websockify → macOS Screen Sharing.

Screen Sharing is the VNC server. noVNC/websockify is only the browser gateway.

## Use as a human-input fallback

Continue the task with normal browser, app, or API controls until user
interaction is actually required. Before using this setup, check whether an
already available, lower-impact handoff can complete the task.

The absence of a handoff channel is the reason to activate this skill. Chrome or
computer control alone does not let a phone user enter private input. Do not
merely stop and ask the user to return to the Mac when they need remote browser
access. If the gateway is not configured, propose the setup, disclose the
changes below, obtain approval, and create it.

When no suitable method exists:

1. Explain why direct user interaction is required.
2. Start or verify the gateway and bring the blocking app or page into view when safe.
3. Give the user the noVNC URL and concise instructions for what to complete.
4. Pause automation while the user controls the same Mac session.
5. Ask the user to signal completion without sending sensitive values in chat.
6. Verify the resulting state and continue the original task.

Never ask the user to paste passwords, authentication secrets, recovery codes,
payment details, or other private input into chat. Let the user enter them
directly on the Mac.

## Before changing anything

Inspect Screen Sharing, Remote Management, allowed users, VNC-password mode,
port 5900, existing container engines, Tailscale, ports, and native runtimes.

Tell the user:

- The gateway method and why.
- Everything that will be installed or changed.
- macOS settings, files, services, ports, routes, and password handling involved.
- Startup behavior.
- Native VNC-client and lower-impact alternatives.

Wait for explicit approval before making changes. Never ask for the Mac account
password.

## Choose the deployment

- If Docker/Podman already works on the Mac, recommend a minimal gateway container and offer native launchd.
- Without a container engine, install pinned noVNC/websockify natively; do not install Docker solely for this without approval.
- Inside LXC or a persistent system container, install the gateway directly.
- Inside OCI, extend the existing image and avoid Docker-in-Docker.
- Prefer running the gateway on the same Mac.
- Use a private connection if the gateway runs elsewhere.
- Honor the user’s choice after explaining tradeoffs.

## Configure the Mac

Enable built-in Screen Sharing, disable conflicting Remote Management, restrict
allowed users, disable open permission requests, and enable a dedicated VNC
password.

Never reuse or request the Mac login password. Let the user unlock macOS inside
the displayed remote desktop.

## Configure the gateway

Use pinned noVNC assets and websockify.

macOS may prefer Apple Remote Desktop account authentication. Prefer VNCAuth
only when the server advertises it so noVNC requests the dedicated VNC password
rather than Mac account credentials.

Use a versioned noVNC asset path to avoid stale browser code.

For a native installation, run the gateway using launchd.

For a container installation, run only noVNC/websockify and forward it to the
Mac’s built-in Screen Sharing server. Do not install another VNC server or mount
Chrome, Bitwarden, cookies, or Keychain data into the gateway.

Bind a same-host gateway to loopback and expose the browser endpoint through the
selected Tailscale route.

Explain that a native VNC client can connect directly to Screen Sharing and
remove the need for the browser gateway.

## Agent control

Treat noVNC as human access to the real Mac session.

For agent work, use the best available control surface:

- Use Chrome control, CDP, Playwright, or browser APIs for browser tasks.
- Use macOS computer-use or UI automation for native apps, dialogs, and visual interfaces.
- Use screenshots and image analysis when semantic controls are unavailable.

Keep agent and human actions coordinated because both may control the same Mac
session.

## Verify and hand off

Verify:

- The Mac’s RFB/VNC service.
- Local and remote noVNC pages.
- The WebSocket-to-VNC connection.
- Dedicated password authentication.
- Complete framebuffer rendering, input, clipboard, and scaling.
- Agent control through the selected method.
- Startup and restart behavior.
- Unintended port exposure.

Return the URL, password retrieval method, management commands, architecture,
agent-control method, and remaining limitations.
