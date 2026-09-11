# ADR-0009: Use xrdp for Private Graphical Access to Forge

## Status

Accepted; final implementation verification is in progress.

## Context

Forge is the persistent development workstation, while the MacBook and iPad are client surfaces. SSH, browser VS Code, and JetBrains Remote Development cover many workflows, but the iPad has no native JetBrains remote client that provides the full IntelliJ experience.

The platform needs a graphical path that:

- Runs IntelliJ IDEA directly on Forge beside the repositories and toolchains.
- Works from an iPad with a keyboard and trackpad.
- Preserves graphical state across ordinary client disconnects.
- Performs acceptably over LAN and variable-quality WireGuard connections.
- Does not expose a remote-desktop service directly to the public internet.
- Remains independent from Forge's physical display and console session.
- Can be rolled back without disturbing the existing development environment.

## Decision

Use xrdp with the xorgxrdp Xorg backend for private graphical access to Forge.

- Use a dedicated per-user graphical session rather than sharing a physical console session.
- Keep XFCE as the initial validated and recovery desktop.
- Run a separate native IntelliJ IDEA installation with isolated desktop config, cache, plugin, and log paths.
- Keep xrdp session retention enabled so an ordinary disconnect does not intentionally destroy the desktop.
- Allow TCP `3389` only from the trusted LAN and WireGuard subnets through firewalld.
- Do not forward TCP `3389` on the upstream router or pfSense WAN.
- Keep WireGuard terminated on pfSense as the only public administrative entry point.
- Keep SSH as the recovery and diagnostic path.
- Evaluate a separate compositor-free i3 profile for a lighter, keyboard-first remote workflow after the XFCE baseline is fully accepted.
- Do not put Hyprland on the xrdp path. A future Wayland streaming experiment requires a separate architecture decision and must not replace the working RDP session implicitly.

## Rationale

xrdp matches the existing Fedora Xorg package path and provides a mature RDP client ecosystem on iPadOS. It supports session reconnection, dynamic resizing, clipboard channels, and efficient transport without requiring Forge to mirror a physical monitor.

Xorg is an intentional compatibility choice here. i3 can reproduce the useful keyboard-first and workspace behavior of Umbra while generating little graphical churn on an unstable connection. XFCE remains the dependable fallback and supplies a familiar recovery environment.

Keeping IntelliJ native to Forge removes the extra remote-filesystem or nested remote-IDE layer. The iPad sends input and receives display updates while builds, indexing, terminals, containers, and databases remain local to Forge.

## Consequences

### Positive

- The iPad can use the full native IntelliJ application.
- Forge owns project files, IDE state, toolchains, and execution.
- Client disconnects need not terminate the development session.
- RDP resolution can be tuned for the iPad independently from physical displays.
- The Xorg path is compatible with lightweight i3 and the packaged xrdp stack.
- XFCE provides a proven fallback while the keyboard-first profile evolves.
- No additional public entry point is required.

### Negative

- The graphical path depends on Forge, the home network, and WireGuard when away from home.
- xrdp listens on a network socket, so firewalld and router policy become mandatory security controls.
- iPadOS and the selected RDP client may intercept or omit some keyboard combinations.
- There is no physical function row on the current iPad keyboard.
- Xorg/i3 will not reproduce Hyprland's Wayland-native animations, gestures, and compositor effects.
- Native IntelliJ consumes more Forge memory than terminal or browser-only workflows.
- Session persistence, clipboard behavior, and external-network quality require ongoing acceptance testing.
- The accepted source-scoped firewall control is not fully verified until the effective privileged firewalld configuration is captured.

## Alternatives Considered

### Continue with JetBrains Remote Development only

Retained for MacBook use, but it does not provide a native iPad client and therefore cannot deliver the requested iPad IntelliJ interface directly.

### Browser VS Code

Retained as a useful private fallback. It does not replace IntelliJ for Java/Spring navigation, refactoring, debugging, and database workflows preferred by the user.

### VNC with XFCE

Viable and simple, but RDP was selected first for iPad client quality, resizing, clipboard integration, and bandwidth behavior.

### NoMachine

Viable challenger if xrdp proves unreliable. It adds another server/client stack and is unnecessary while the current RDP path is performing well.

### Hyprland with Sunshine/Moonlight or WayVNC

Potentially closer to Umbra's exact local experience, but it changes the transport and session architecture. It is deferred because xrdp is Xorg-based and the current priority is a light, resilient development session rather than compositor fidelity.

### Remote the MacBook

Rejected as the primary path because it adds another always-on workstation and can create a nested route from iPad to MacBook to Forge. Direct iPad-to-Forge RDP has fewer moving parts.

## Related ADRs

- ADR-0001: Use Fedora Server for the Development Server
- ADR-0002: Use Ansible for Configuration Management
- ADR-0004: Use WireGuard via pfSense for Remote Access
- ADR-0005: Separate Development, Deployment, and Platform Responsibilities
