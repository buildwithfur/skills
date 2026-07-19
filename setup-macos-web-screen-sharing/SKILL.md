---
name: setup-macos-web-screen-sharing
description: Set up or troubleshoot browser access to a real Mac desktop using macOS Screen Sharing, noVNC/websockify, and Tailscale. Use when someone wants their existing Mac session, Chrome profile, password manager, or Mac apps from a phone browser without requiring a native VNC client; support native or container gateways.
---

# macOS web screen sharing

Create:

Phone browser → Tailscale → noVNC/websockify → macOS Screen Sharing.

Screen Sharing is the VNC server. noVNC/websockify is only the browser gateway.

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
