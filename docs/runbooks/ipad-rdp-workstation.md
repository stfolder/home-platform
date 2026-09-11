# Forge iPad RDP Workstation Runbook

## Status

The native IntelliJ RDP path is working from an iPad on the trusted home LAN.

Validated on 2026-09-11:

- Fedora 44 runs `xrdp` with the `xorgxrdp` Xorg backend.
- A dedicated XFCE desktop starts for the Forge development user.
- The iPad establishes a TLS-protected RDP connection and receives a resizable desktop.
- IntelliJ IDEA Ultimate runs directly on Forge with its own desktop configuration and cache paths.
- The xrdp clipboard channel is enabled and its channel process is running.
- Basic IntelliJ use from the iPad is practical; missing function-row keys and some modifier combinations remain client-input limitations.

Still to validate:

- Capture the effective firewalld rules with administrative privileges and confirm that TCP `3389` is allowed only from `10.42.42.0/24` and `10.20.30.0/24`.
- Connect through WireGuard from a genuinely external network and repeat the RDP acceptance test.
- Disconnect and reconnect deliberately, confirming that the same IntelliJ/XFCE session is recovered.
- Test clipboard text in both directions and record the iPad client's exact modifier-key behavior.
- Evaluate the planned i3 profile described below. i3 is not installed or selected yet.

Related documents:

- [Forge remote development runbook](remote-development.md)
- [Forge VPN remote access runbook](vpn-remote-access.md)
- [ADR-0004: Use WireGuard via pfSense for Remote Access](../adr/0004-use-wireguard-via-pfsense.md)
- [ADR-0009: Use xrdp for Private Graphical Access to Forge](../adr/0009-use-xrdp-for-private-graphical-access.md)

## Purpose

This path makes the iPad a thin graphical client while Forge remains the development workstation:

```text
iPad
  -> trusted LAN or WireGuard
  -> RDP on Forge TCP 3389
  -> persistent Xorg desktop session
  -> IntelliJ IDEA running directly on Forge
  -> Forge-local repositories, toolchains, containers, and databases
```

There is no remote filesystem layer between IntelliJ and the project. The iPad carries pixels and input; Forge owns the IDE process and development state.

## Security Boundary

RDP is an internal administrative service.

- Never forward TCP `3389` through the COX router or pfSense WAN.
- Never publish RDP through public DNS or a cloud relay.
- Allow it only from the trusted LAN subnet `10.42.42.0/24` and trusted WireGuard subnet `10.20.30.0/24`.
- Keep the WireGuard UDP endpoint as the only public administrative entry point.
- Keep SSH available as the recovery path while changing the graphical session.

xrdp currently listens on all Forge interfaces at TCP `3389`. The firewall and router policy therefore form the network boundary; both must be verified after changes.

Capture the existing runtime and permanent policy before making any change:

```bash
sudo firewall-cmd --zone=FedoraServer --list-all
sudo firewall-cmd --permanent --zone=FedoraServer --list-all
```

The intended firewalld policy is source-scoped rich rules rather than a global port allowance:

```bash
sudo firewall-cmd --permanent --zone=FedoraServer \
  --add-rich-rule='rule family="ipv4" source address="10.42.42.0/24" port port="3389" protocol="tcp" accept'

sudo firewall-cmd --permanent --zone=FedoraServer \
  --add-rich-rule='rule family="ipv4" source address="10.20.30.0/24" port port="3389" protocol="tcp" accept'

sudo firewall-cmd --check-config
sudo firewall-cmd --reload
```

Do not add the rich rules until the preflight confirms what already exists. Avoid duplicate rules. Remove any unscoped `3389/tcp` port or RDP service allowance if one exists, but keep an SSH recovery session open while changing the firewall.

Verify runtime and permanent state:

```bash
sudo firewall-cmd --zone=FedoraServer --list-all
sudo firewall-cmd --permanent --zone=FedoraServer --list-all
sudo ss -lntp | grep ':3389'
```

Expected result:

- xrdp listens on TCP `3389`.
- No unscoped `ports: 3389/tcp` or broad RDP service entry exists.
- Two rich rules admit only the trusted LAN and WireGuard subnets.

## Installed Baseline

| Component | Validated value |
|---|---|
| Operating system | Fedora Linux 44 Server Edition |
| `xrdp` | `0.10.6.1-3.fc44.x86_64` |
| `xorgxrdp` | `0.10.5-1.fc44.x86_64` |
| `xfce4-session` | `4.20.3-2.fc44.x86_64` |
| `xfwm4` | `4.20.0-4.fc44.x86_64` |
| RDP listener | TCP `3389` |
| Xorg display | `:10` during validation |
| IntelliJ IDEA | Ultimate `2026.1.4` |

Service state during validation:

| Service | Enabled | Active |
|---|---:|---:|
| `xrdp.service` | yes | yes |
| `xrdp-sesman.service` | indirectly managed with xrdp | yes |

The service uses the packaged xrdp certificate and negotiated TLS 1.3 with the iPad client during the successful connection.

## Desktop Session

The per-user `~/.Xclients` starts XFCE without changing Forge's console environment:

```sh
#!/bin/sh
export XDG_CURRENT_DESKTOP=XFCE
export XDG_SESSION_DESKTOP=xfce
export DESKTOP_SESSION=xfce
exec dbus-run-session -- startxfce4
```

The file is executable and user-owned. xrdp created an Xorg session on display `:10` with these active components:

- `Xorg` using `/etc/X11/xrdp/xorg.conf`
- `xrdp-chansrv`
- `xfce4-session`
- `xfwm4`

The packaged session policy is reconnect-oriented:

```ini
KillDisconnected=false
DisconnectedTimeLimit=0
IdleTimeLimit=0
Policy=UB
```

These values retain disconnected sessions and permit a matching user/color-depth client to rejoin. A deliberate disconnect/reconnect test is still required before treating persistence as fully accepted.

## RDP Transport Characteristics

The successful iPad connection established:

| Check | Result |
|---|---|
| Client identity | iPad |
| Keyboard layout reported by client | US, `0x00000409` |
| TLS | TLS 1.3, `TLS_AES_256_GCM_SHA384` |
| Graphics support | RDP graphics pipeline accepted |
| Codec selected | RemoteFX mode with OpenH264 software encoder available |
| Initial surface | `2240x1473` |
| Resized desktop | `2240x1536` |
| Resize support | accepted |
| Clipboard channel | `cliprdr=true`; `xrdp-chansrv` active |

For unstable connections, prefer a moderate logical resolution rather than maximizing the iPad's native pixel count. Avoid video, animated wallpapers, transparency, and desktop effects; screen changes cost more bandwidth than an idle desktop.

## IntelliJ Desktop Installation

The desktop IntelliJ installation is intentionally separate from JetBrains Remote Development backends:

```text
~/.local/opt/idea-IU-2026.1.4-forge-desktop
```

Launcher:

```text
~/.local/bin/forge-idea
```

Desktop entry:

```text
~/.local/share/applications/forge-intellij-idea.desktop
```

The wrapper points IntelliJ at isolated desktop-session state:

```properties
idea.config.path=/home/serge/.config/JetBrains/IntelliJIdea2026.1-forge-desktop
idea.system.path=/home/serge/.cache/JetBrains/IntelliJIdea2026.1-forge-desktop
idea.plugins.path=/home/serge/.local/share/JetBrains/IntelliJIdea2026.1-forge-desktop
idea.log.path=/home/serge/.cache/JetBrains/IntelliJIdea2026.1-forge-desktop/log
```

This prevents the native desktop instance from competing with the existing Remote Development backend for config, caches, plugins, or logs.

During validation, the native IntelliJ process and its Maven helper were active inside the RDP desktop.

## iPad Connection

On the trusted home LAN:

```text
Host: forge.home.arpa
Port: 3389
User: serge
Session: Xorg
```

Away from home:

1. Connect the trusted iPad WireGuard peer.
2. Confirm that `forge.home.arpa` resolves through the home DNS resolver.
3. Connect the RDP client to `forge.home.arpa:3389`.
4. Disconnect WireGuard after the session and confirm that RDP is no longer reachable.

Do not save the Forge password in this public repository. Whether the iPad RDP client stores it locally is a client-security decision.

## iPad Acceptance Test

- [x] Establish an RDP session from the iPad on the trusted LAN.
- [x] Start native IntelliJ IDEA on Forge.
- [x] Use IntelliJ interactively from the iPad.
- [x] Obtain a resizable high-resolution desktop.
- [ ] Connect through WireGuard from an external network.
- [ ] Verify two-way plain-text clipboard behavior.
- [ ] Record Command, Option, Control, Shift, Escape, and Caps Lock behavior.
- [ ] Define replacements for useful function-row shortcuts.
- [ ] Validate right click, selection, dragging, and trackpad scrolling.
- [ ] Validate IntelliJ terminal, build, run, debug, and database tools in this native session.
- [ ] Disconnect and reconnect to the same session.
- [ ] Reboot Forge and validate predictable recovery.
- [ ] Test a constrained or unstable network connection.

## Planned i3 Profile

The next experiment is an additional lightweight i3 X11 profile, not a replacement for the validated XFCE session.

Design requirements:

- Keep XFCE selectable as the recovery desktop.
- Run i3 without a compositor, blur, transparency, or animations.
- Retain only required helpers for notifications, settings, authentication, and clipboard integration.
- Use a minimal, slowly refreshed status bar.
- Recreate Umbra's meaningful interaction model: directional focus, deterministic tiling, numbered workspaces, launcher, fullscreen, and move-to-workspace actions.
- Bind only key combinations that the iPad RDP client reliably transmits; do not depend on a function row.
- Keep configuration under user scope until the profile is validated and ready for Ansible ownership.

Hyprland is not part of this RDP path. It is a Wayland compositor and would require a different remote-display architecture such as Sunshine/Moonlight or WayVNC. That can be evaluated separately if native Hyprland behavior becomes more important than xrdp compatibility.

## Operational Checks

Service and process check:

```bash
systemctl is-enabled xrdp.service
systemctl is-active xrdp.service xrdp-sesman.service
pgrep -a -f 'Xorg|xrdp|xfce4-session|xfwm4|xrdp-chansrv'
```

Recent logs:

```bash
sudo journalctl -u xrdp -u xrdp-sesman --since today --no-pager
```

The Fedora 44 package emitted a `pam_lastlog.so` load warning during the successful login. Authentication and session creation still completed. Treat it as a follow-up only if it causes login or audit behavior to fail.

## Rollback

The safest immediate rollback disables network access before removing desktop state:

```bash
sudo systemctl disable --now xrdp.service

sudo firewall-cmd --permanent --zone=FedoraServer \
  --remove-rich-rule='rule family="ipv4" source address="10.42.42.0/24" port port="3389" protocol="tcp" accept'

sudo firewall-cmd --permanent --zone=FedoraServer \
  --remove-rich-rule='rule family="ipv4" source address="10.20.30.0/24" port port="3389" protocol="tcp" accept'

sudo firewall-cmd --check-config
sudo firewall-cmd --reload
```

Confirm that TCP `3389` is no longer listening or reachable.

Package removal is optional and should be reviewed separately:

```bash
sudo dnf remove xrdp xorgxrdp
```

Do not automatically remove the XFCE package group, IntelliJ installation, projects, caches, or user configuration. Preserve `~/.Xclients` and the desktop IntelliJ state until their ownership and reuse are reviewed. The planned i3 profile must have its own bounded rollback and must never strand the user without the working XFCE session.
