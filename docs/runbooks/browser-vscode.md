# Forge Browser VS Code Runbook

## Status

Implemented for story #20 on the Forge LAN. Outside-LAN access remains deferred until the VPN-only access story is complete.

Related stories:

- [#15 Validate IntelliJ, VS Code, and iPad remote development workflows](https://github.com/stfolder/home-platform/issues/15)
- [#18 Enable VPN-only remote access](https://github.com/stfolder/home-platform/issues/18)
- [#20 Deploy a private browser VS Code workspace on Forge](https://github.com/stfolder/home-platform/issues/20)

## Course Frame

Story #20 adds a browser-accessible VS Code workspace to Forge. The goal is not to turn Forge into a public carnival ride with a terminal button. The goal is a private, authenticated, LAN/VPN-only editor for approved devices, especially iPad Safari.

Security posture:

- no public Internet exposure
- no router port forwarding
- no public tunnel
- no Docker TCP listener
- no service running as root
- no credentials in Git

The browser IDE is a loaded power tool with a web UI. Treat it like a motorcycle with no traction control: useful, fast, and very uninterested in your excuses if you point it at a wall.

## Architecture Decision

Selected implementation: **code-server**.

Rationale:

- The story explicitly prefers code-server unless a material blocker appears.
- code-server has built-in password authentication by default.
- code-server's official guidance explicitly warns not to expose it without authentication and encryption because terminal access can take over the machine.
- code-server supports direct host-native installation, which matches the story's preference for a service running under `serge`.
- code-server has specific iPad guidance in its documentation set.
- OpenVSCode Server is valuable as an upstream-like browser VS Code server, but its documented quick-start leans Docker-first and can run without authentication unless a connection token is configured.
- For Forge, the boring single-user authenticated service is better than the purist upstream-ish option that needs more babysitting. We are building a private workbench, not auditioning for a conference talk.
- The final deployment uses the official code-server RPM, `code-server@serge.service`, an explicit Forge LAN bind, password authentication, and a LAN-scoped firewall rule.

Sources reviewed:

- code-server GitHub repository and current release listing.
- code-server install and usage/security documentation.
- OpenVSCode Server GitHub repository and setup/security notes.

## Baseline Capture

### Part 1: Forge Browser IDE Baseline

Status: PASS.

Captured on Forge before installing a browser IDE.

| Check | Result |
|---|---|
| Host | `forge` |
| User | `serge` |
| Capture time | `2026-07-14T11:23:03-07:00` |
| Working directory | `/home/serge/forge-labs/story-15/remote-dev-validation` |

System health:

| Check | Result |
|---|---|
| `firewalld` | active and enabled |
| `sshd` | active and enabled |
| `cockpit.socket` | active and enabled |
| `docker` | active and enabled |
| `dnf5-automatic.timer` | active and enabled |
| Failed units | `fwupd-refresh.service` failed |

The `fwupd-refresh.service` failure also appeared transiently during #15 and cleared after reboot. It was rechecked during final validation, and the final service-health check reported no failed units.

Existing browser IDE commands/packages/services:

| Check | Result |
|---|---|
| `code-server` | not installed |
| `openvscode-server` | not installed |
| `coder` | not installed |
| Browser IDE RPM packages | none |
| Browser IDE system services | none |
| Browser IDE user services | none |
| Node.js | available through `nvm`: `/home/serge/.nvm/versions/node/v24.18.0/bin/node` |
| npm | available through `nvm`: `/home/serge/.nvm/versions/node/v24.18.0/bin/npm` |

Existing remote-editor state from #15:

| Component | Result |
|---|---|
| VS Code Remote SSH server | running under `serge` from `/home/serge/.vscode-server` |
| VS Code Remote SSH data size | `2.2G` |
| IntelliJ Remote Development backend | loopback-only listeners from existing remote session |
| Browser IDE permanent install | none |

Listening sockets before browser IDE:

| Socket | Process | Notes |
|---|---|---|
| TCP `:22` | `sshd` | expected LAN-scoped SSH |
| TCP `:9090` | Cockpit/systemd socket | expected LAN-scoped Cockpit |
| TCP/UDP loopback `:53` | `systemd-resolved` | local resolver |
| UDP loopback `:323` | `chronyd` | local time service |
| TCP loopback ports | VS Code Remote SSH and IntelliJ backend | expected #15 IDE sessions |

Firewall baseline:

| Check | Runtime | Permanent |
|---|---|---|
| Zone | `FedoraServer` | `FedoraServer` |
| Interface | `eno2` | none listed in permanent view |
| Services | none | none |
| Ports | none | none |
| Forward | `no` | `no` |
| Rich rule: LAN SSH | present for `10.42.42.0/24` | present for `10.42.42.0/24` |
| Rich rule: LAN Cockpit | present for `10.42.42.0/24` | present for `10.42.42.0/24` |

Docker baseline:

| Resource | Result |
|---|---|
| Containers | none |
| Images | `postgres:latest`, retained #15 Dev Container images |
| Volumes | `vscode` |
| Networks | defaults only: `bridge`, `host`, `none` |

Representative repository:

| Check | Result |
|---|---|
| Path | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| Latest commit | `e1d8a44 Record dev container feature lockfile` |
| Java | OpenJDK `25.0.3` |
| Maven | `3.9.11` from Fedora, using Java `25.0.3` |

## Product Comparison

### code-server

Status: SELECTED unless install validation reveals a blocker.

Relevant current official guidance:

- It runs VS Code in the browser.
- Requirements are modest: Linux, WebSockets, 1 GB RAM, and 2 vCPUs.
- The install script can use the system package manager where possible.
- Default exposure is conservative: it listens on localhost and uses password authentication.
- Official docs warn that exposing code-server directly without authentication/encryption can allow takeover through the terminal.
- SSH forwarding is recommended for secure access, but iPad use requires another access pattern because iPad Safari is not an SSH port-forwarding workflow.
- Self-signed certificates are specifically called out as problematic for iPads.

Fit for Forge:

| Criterion | Result |
|---|---|
| Native install under `serge` | good fit |
| Built-in authentication | good fit |
| iPad-focused docs | good fit |
| LAN/VPN private listener | good fit with firewalld rich rule |
| Extension ecosystem | practical for validation |
| Operational friction | low |

### OpenVSCode Server

Status: NOT SELECTED for the permanent #20 install unless code-server blocks.

Relevant current official guidance:

- It provides VS Code in a browser and is based on upstream VS Code server architecture.
- Documented Docker quick start maps port `3000:3000`.
- Linux install is available by downloading a release tarball and running `./bin/openvscode-server`.
- Default port is `3000`.
- Host defaults to localhost; remote access requires changing the host binding.
- It can run without authentication; connection token or token file is optional but must be configured for secure use.

Fit for Forge:

| Criterion | Result |
|---|---|
| Native install under `serge` | possible, but more manual |
| Built-in authentication | token-based, less friendly than code-server password flow |
| Docker-first docs | less aligned with this story |
| Extension ecosystem | OpenVSX-oriented; may be fine, but must be validated |
| Operational friction | higher for this single-user private service |

Conclusion: OpenVSCode Server is the cleaner upstream-ish browser VS Code server, but code-server is the better private single-user Forge appliance. OpenVSCode is kept as the spare tire, not mounted on the bike.

## Final Installation Model

Validated deployment:

| Item | Value |
|---|---|
| Product | code-server |
| Version | `4.128.0` with embedded Code `1.128.0` |
| Install method | official install script installing the GitHub release RPM |
| Process owner | `serge` |
| Service owner | system service `code-server@serge.service`, running as `serge` |
| Bind | `10.42.42.77` |
| Port | `8787` |
| Authentication | password auth enabled |
| TLS | HTTP on private LAN; outside-LAN access deferred to VPN-only path |
| Firewall | LAN-scoped rich rule for `10.42.42.0/24` to TCP `8787` |
| Boot behavior | systemd override waits for `network-online.target` and `NetworkManager-wait-online.service` |
| Public access | forbidden |

## Closure Notes

- Story #20 is complete for private LAN browser VS Code access.
- Outside-LAN access remains deferred until VPN-only access is available.
- Browser insecure-context warnings are expected while code-server is served over plain HTTP on the private LAN.
- HTTPS or localhost-tunnel access should be tracked as follow-up hardening if browser clipboard or webview behavior becomes important.

## Installation Validation

### Part 2: code-server Install Dry Run

Status: PASS.

Dry-run command:

```bash
curl -fsSL https://code-server.dev/install.sh | sh -s -- --dry-run
```

Dry-run result:

| Check | Result |
|---|---|
| Detected OS | Fedora Linux 44 Server Edition |
| Selected code-server version | `v4.128.0` |
| Selected package | `code-server-4.128.0-amd64.rpm` |
| Package source | GitHub release download |
| Planned cache path | `~/.cache/code-server/code-server-4.128.0-amd64.rpm` |
| Planned install command | `sudo rpm -U ~/.cache/code-server/code-server-4.128.0-amd64.rpm` |
| Suggested service command | `sudo systemctl enable --now code-server@$USER` |
| `code-server` command after dry run | not present |
| code-server RPM after dry run | not present |

Decision: the official install script dry run is acceptable. It chooses a concrete RPM release and a simple RPM installation path. This is boring in the best possible way: no mystery curl confetti, no container identity circus, no daemon pretending it needs root to edit a README.

### Part 3: code-server Installation

Status: PASS.

Install command:

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

Install result:

| Check | Result |
|---|---|
| Detected OS | Fedora Linux 44 Server Edition |
| Installed version | `4.128.0` |
| Embedded Code version | `1.128.0` |
| Commit/build | `cb22f74650a539d6f824d5944ec34d9e74844f66` |
| Package | `code-server-4.128.0-1.x86_64` |
| Binary | `/usr/bin/code-server` |
| Package source | GitHub release RPM via official install script |
| Default config generated | `/home/serge/.config/code-server/config.yaml` |

Systemd unit:

```ini
[Unit]
Description=code-server
After=network.target

[Service]
Type=exec
ExecStart=/usr/bin/code-server
Restart=always
User=%i

[Install]
WantedBy=default.target
```

The package-owned unit is a templated system service. For `code-server@serge.service`, systemd runs `/usr/bin/code-server` as `serge`. This matches the story requirement: the browser IDE should have the same Forge filesystem identity as the development user without running as root.

Important behavior: running `code-server --version` generated a default user config. The generated config must be inspected and replaced with an intentional configuration before enabling the service. Letting a default remote-shell UI wander onto the LAN is the kind of move that makes future-you ask whether past-you was supervised.

### Part 4: Default Config Inspection

Status: PASS with required credential rotation.

Default config:

| Check | Result |
|---|---|
| Config path | `/home/serge/.config/code-server/config.yaml` |
| Owner | `serge:serge` |
| Initial permissions | `0644` |
| Default bind | `127.0.0.1:8080` |
| Default auth | `password` |
| Default cert | `false` |

The generated default config included a plain password. Because it was displayed during validation, it must be treated as disclosed and replaced before the service is started.

Config path permissions:

| Path | Owner | Permissions |
|---|---|---|
| `/home/serge` | `serge:serge` | `0700` |
| `/home/serge/.config` | `serge:serge` | `0700` |
| `/home/serge/.config/code-server` | `serge:serge` | `0755` before hardening |
| `config.yaml` | `serge:serge` | `0644` before hardening |

Network candidate:

| Check | Result |
|---|---|
| Interface | `eno2` |
| LAN address | `10.42.42.77/24` |
| Default gateway | `10.42.42.1` |
| Initially considered port | `8080` |
| Existing listener on `8080` | none |
| Final selected port | `8787` |
| Existing LAN listeners | SSH `22`, Cockpit `9090` |

Decision:

- Bind code-server to `10.42.42.77:8787`.
- Add a LAN-scoped firewalld rich rule for `10.42.42.0/24`.
- Rotate the generated password before starting the service.
- Restrict code-server config directory and files to the `serge` user.
- Avoid port `8080` because it is too common for application development and would create needless future collisions.

Binding to the explicit Forge LAN address avoids listening on Docker bridges or future interfaces. If the Forge DHCP reservation changes, update this config. That is a better failure mode than letting the IDE listen on every interface like it just discovered nightclub confidence.

### Part 5: Harden Config And Start Service

Status: PASS.

Configuration:

| Check | Result |
|---|---|
| Config directory | `/home/serge/.config/code-server` |
| Config directory permissions | `0700` |
| Config file | `/home/serge/.config/code-server/config.yaml` |
| Config file permissions | `0600` |
| Password file | `/home/serge/.config/code-server/password.txt` |
| Password file permissions | `0600` |
| Bind address | `10.42.42.77:8787` |
| Auth | `password` |
| Cert | `false` |
| Password handling | rotated and redacted from docs |

Service:

| Check | Result |
|---|---|
| Unit | `code-server@serge.service` |
| Enabled | yes |
| Active | running |
| Process owner | `serge` |
| Main process | `/usr/lib/code-server/lib/node /usr/lib/code-server` |

The first listener check was run immediately after service start and did not show port `8787`, while the process was still present. A follow-up check confirmed the listener after startup settled.

Follow-up status:

| Check | Result |
|---|---|
| Service uptime at check | about 2 minutes |
| Tasks | 22 |
| Memory | about 54 MB |
| Logs: version | `code-server 4.128.0 cb22f74650a539d6f824d5944ec34d9e74844f66` |
| Logs: user data dir | `/home/serge/.local/share/code-server` |
| Logs: config file | `/home/serge/.config/code-server/config.yaml` |
| Logs: listener | `http://10.42.42.77:8787/` |
| Logs: auth | enabled |
| Logs: HTTPS | not serving HTTPS |
| Session socket | `/home/serge/.local/share/code-server/code-server-ipc.sock` |
| Socket check | `LISTEN 10.42.42.77:8787` |
| Local HTTP check | `302 Found` to `./login` |

Conclusion: code-server is running as `serge`, listening only on the chosen Forge LAN address and port, and requiring authentication. It is not serving HTTPS; this is accepted temporarily for private LAN validation and must remain VPN-only outside the LAN after #18. No public exposure is allowed.

### Part 6: LAN-Scoped Firewall Rule

Status: PASS.

Firewall change:

```bash
sudo firewall-cmd --permanent --zone=FedoraServer \
  --add-rich-rule='rule family="ipv4" source address="10.42.42.0/24" port port="8787" protocol="tcp" accept'
sudo firewall-cmd --reload
```

Result:

| Check | Runtime | Permanent |
|---|---|---|
| Zone | `FedoraServer` | `FedoraServer` |
| Interface | `eno2` | none listed in permanent view |
| Services | none | none |
| Ports | none | none |
| Forward | `no` | `no` |
| Rich rule: LAN SSH | present | present |
| Rich rule: LAN Cockpit | present | present |
| Rich rule: LAN code-server | `10.42.42.0/24` to TCP `8787` | `10.42.42.0/24` to TCP `8787` |

Listener check:

| Socket | Process | Result |
|---|---|---|
| `10.42.42.77:8787` | code-server `MainThread` | expected |
| `:22` | `sshd` | expected |
| `:9090` | Cockpit/systemd | expected |

Conclusion: code-server is the only intentional new LAN-reachable service, and it is constrained by both explicit bind address and LAN-scoped firewalld rule.

### Part 7: MacBook And iPad Browser Authentication

Status: PASS.

MacBook unauthenticated HTTP check:

| Check | Result |
|---|---|
| URL | `http://forge.home.arpa:8787/` |
| HTTP status | `302 Found` |
| Redirect | `./login` |
| Private browser requires auth | yes |
| Login works | yes |
| Repository folder opens | yes |

MacBook port reachability:

| Port | Result |
|---:|---|
| `8787` | reachable |
| `8080` | refused |
| `3000` | refused |

iPad Safari:

| Check | Result |
|---|---|
| Login required | yes |
| Login works | yes |
| Repository folder opens | yes |
| Result parity with MacBook | same |

Integrated terminal evidence:

| Check | Result |
|---|---|
| Host | `forge` |
| User | `serge` |
| Working directory | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| Java | OpenJDK `25.0.3` |
| Maven | `3.9.11`, using Java `25.0.3` |

Conclusion: approved LAN clients can reach code-server, unauthenticated clients are redirected to login, and the browser IDE uses the Forge toolchain rather than local client runtimes.

### Part 8: iPad Safari Development Workflow

Status: PASS.

iPad usability:

| Check | Result |
|---|---|
| Hardware keyboard typing | acceptable |
| Selection / scrolling / context menus | usable |
| Paste into integrated terminal | works |
| Copy out of integrated terminal | did not work during validation |
| Annoyance | iPadOS keyboard menu consumes bottom-screen space |

Development workflow:

| Check | Result |
|---|---|
| Repository | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| File edited from iPad Safari | `src/main/java/dev/forge/remote/MissionBriefing.java` |
| Commit | `d09559b Validate browser VS Code iPad workflow` |
| `mvn test -q` after commit | PASS, exit status `0` |
| App output | `Forge remote development is online. Debugger helmet: polished. Browser deck: online.` |
| Final Git status | clean |

Evidence capture workaround:

- Copying output from the iPad integrated terminal did not work reliably.
- Evidence was captured from MacBook SSH after the iPad made the edit and commit.
- The source in `HEAD` matched the browser edit, proving the browser workflow changed the Forge repository.
- A malformed paste created one accidental untracked file; it was removed precisely without using broad cleanup.

The accidental file was:

```text
er IDE Maven test =="
```

Conclusion: iPad Safari can perform the meaningful browser-IDE workflow: edit source, use the integrated terminal, run Maven tests, run the app, and produce a local Git commit on Forge. The copy-out limitation and bottom keyboard bar are real iPadOS usability annoyances, but they do not block the workflow. This is not desktop parity; it is a usable thin-client cockpit with a few buttons placed by someone who has clearly never worn gloves.

### Part 9: Browser Reconnect And Session Behavior

Status: PASS.

iPad Safari reconnect:

| Check | Result |
|---|---|
| Closed/reopened browser tab | PASS |
| Session behavior | stayed logged in |
| Repository still usable | yes |
| Integrated terminal still usable | yes |

Service evidence:

| Check | Result |
|---|---|
| Service | `code-server@serge.service` |
| State | active/running |
| Uptime at check | about 31 minutes |
| Tasks | 87 |
| Memory | about 517 MB, peak about 814 MB |
| Listener | `10.42.42.77:8787` |
| Process owner | `serge` |
| Logs | client disconnected and reconnected cleanly from `10.42.42.174` |

Repository evidence after reconnect:

| Check | Result |
|---|---|
| Latest commit | `d09559b Validate browser VS Code iPad workflow` |
| Maven test | PASS |
| Test exit status | `ssh-captured-mvn-status=0` |

Log note: code-server logged missing `vsda` web files during reconnect. The session still reconnected and the workflow remained usable, so this is recorded as non-blocking noise unless it repeats as a functional issue.

### Part 10: Reboot Failure And Network-Online Override

Status: PASS.

Initial reboot result:

| Check | Result |
|---|---|
| `firewalld`, `sshd`, `cockpit.socket`, `docker`, `dnf5-automatic.timer` | active |
| `code-server@serge` | failed |
| Listener on `8787` | absent |
| Firewall runtime/permanent | code-server LAN rule persisted |

Failure logs:

```text
listen EADDRNOTAVAIL: address not available 10.42.42.77:8787
```

Root cause:

- code-server started at `12:23:12`.
- `NetworkManager-wait-online.service` completed at `12:23:18`.
- code-server tried to bind `10.42.42.77:8787` before `eno2` had the address.

Decision:

- Keep the explicit bind to `10.42.42.77:8787`.
- Add a systemd instance override so `code-server@serge` waits for `network-online.target` and `NetworkManager-wait-online.service`.
- Do not loosen the bind to `0.0.0.0` unless the explicit bind remains unreliable after the ordering fix.

Override:

```ini
[Unit]
Wants=network-online.target
After=network-online.target NetworkManager-wait-online.service
```

Override location:

```text
/etc/systemd/system/code-server@serge.service.d/override.conf
```

Validation after applying override:

| Check | Result |
|---|---|
| Override visible in `systemctl cat code-server@serge` | yes |
| Service restart | PASS |
| Listener | `10.42.42.77:8787` |
| Auth redirect | `302 Found` to `./login` |
| Process owner | `serge` |

This is the right kind of fix: wait for the network instead of making the service listen everywhere and hoping the firewall stays the only responsible adult in the room.

### Part 10B: Reboot Re-Test

Status: PASS.

Pre-reboot check:

| Check | Result |
|---|---|
| Override present | yes |
| Service enabled | yes |
| Service active | yes |
| Listener | `10.42.42.77:8787` |

Post-reboot health:

| Check | Result |
|---|---|
| Failed systemd units | none |
| `firewalld`, `sshd`, `cockpit.socket`, `docker`, `dnf5-automatic.timer`, `code-server@serge` | active |
| All listed services | enabled |
| code-server active after reboot | yes |
| code-server memory at check | about 106 MB |
| code-server listener | `10.42.42.77:8787` |
| Auth redirect | `302 Found` to `./login` |
| iPad Safari page open after reboot | PASS |

Firewall after reboot:

| Check | Runtime | Permanent |
|---|---|---|
| Zone | `FedoraServer` | `FedoraServer` |
| Services | none | none |
| Ports | none | none |
| Forward | `no` | `no` |
| LAN SSH rule | present | present |
| LAN Cockpit rule | present | present |
| LAN code-server TCP `8787` rule | present | present |

Conclusion: code-server survives reboot and the network-online override fixes the explicit-bind startup race.

## Operations

### Part 11: Operational Procedures

Status: PASS.

Status and logs:

```bash
systemctl status code-server@serge --no-pager
systemctl is-active code-server@serge
systemctl is-enabled code-server@serge
journalctl -u code-server@serge -n 80 --no-pager
```

Validated results:

| Check | Result |
|---|---|
| Service active | yes |
| Service enabled | yes |
| Restart validation | PASS |
| Listener after restart | `10.42.42.77:8787` |
| Auth redirect after restart | `302 Found` to `./login` |

Data and configuration:

| Path | Size / permissions | Purpose |
|---|---|---|
| `/home/serge/.config/code-server` | `8.0K`; config/password files `0600` | configuration and local password note |
| `/home/serge/.local/share/code-server` | `2.1M` during validation | user data, logs, sessions, IPC socket |
| `/home/serge/.cache/code-server` | `192M` during validation | installer/package cache |

Package ownership:

| Check | Result |
|---|---|
| Package | `code-server-4.128.0-1.x86_64` |
| Vendor | Coder |
| License | MIT |
| Install date | `2026-07-14 11:30:24 PDT` |
| Installed size | about `646 MB` |
| Upstream URL | `https://github.com/coder/code-server` |
| RPM signature | none |

Restart:

```bash
sudo systemctl restart code-server@serge
sleep 3
systemctl is-active code-server@serge
curl -I --max-time 5 http://10.42.42.77:8787/
```

Password rotation:

```bash
CODE_SERVER_PASSWORD="$(openssl rand -base64 32)"
sed -i "s/^password: .*/password: ${CODE_SERVER_PASSWORD}/" "$HOME/.config/code-server/config.yaml"
printf '%s\n' "$CODE_SERVER_PASSWORD" > "$HOME/.config/code-server/password.txt"
chmod 600 "$HOME/.config/code-server/config.yaml" "$HOME/.config/code-server/password.txt"
unset CODE_SERVER_PASSWORD
sudo systemctl restart code-server@serge
```

Do not paste the password into docs, screenshots, shell transcripts, or Git. This is a remote shell in browser clothing; leaking the password is not “oops,” it is “congratulations, the browser now has keys to the workshop.”

Disable without uninstalling:

```bash
sudo systemctl disable --now code-server@serge
```

Re-enable:

```bash
sudo systemctl enable --now code-server@serge
```

Firewall removal:

```bash
sudo firewall-cmd --permanent --zone=FedoraServer \
  --remove-rich-rule='rule family="ipv4" source address="10.42.42.0/24" port port="8787" protocol="tcp" accept'
sudo firewall-cmd --reload
```

Uninstall:

```bash
sudo systemctl disable --now code-server@serge
sudo rm -rf /etc/systemd/system/code-server@serge.service.d
sudo systemctl daemon-reload
sudo rpm -e code-server
```

Optional user-data cleanup after uninstall:

```bash
rm -rf "$HOME/.config/code-server" "$HOME/.local/share/code-server" "$HOME/.cache/code-server"
```

Upgrade path:

- Review the current code-server release and install documentation.
- Re-run the official install script or install the selected RPM deliberately.
- Restart `code-server@serge`.
- Revalidate auth, listener, firewall, iPad login, and representative repository workflow.

Rollback limitation: because this was installed from an unsigned upstream RPM rather than a versioned Fedora repository, rollback means intentionally installing a previous upstream RPM or removing the package. Do not assume `dnf history undo` is the clean rollback story here.

## Final Validation

### Part 12: Final Exposure And Health Check

Status: PASS.

Service health:

| Check | Result |
|---|---|
| Failed systemd units | none |
| `firewalld`, `sshd`, `cockpit.socket`, `docker`, `dnf5-automatic.timer`, `code-server@serge` | active |
| All listed services | enabled |

Final listeners:

| Socket | Process | Result |
|---|---|---|
| TCP `:22` | `sshd` | expected LAN-scoped SSH |
| TCP `:9090` | Cockpit/systemd socket | expected LAN-scoped Cockpit |
| TCP `10.42.42.77:8787` | code-server | expected LAN-scoped browser IDE |
| TCP/UDP loopback `:53` | `systemd-resolved` | expected local resolver |
| UDP loopback `:323` | `chronyd` | expected local time socket |

Final firewall state:

| Check | Runtime | Permanent |
|---|---|---|
| Zone | `FedoraServer` | `FedoraServer` |
| Interface | `eno2` | none listed in permanent view |
| Services | none | none |
| Ports | none | none |
| Forward | `no` | `no` |
| LAN SSH rule | present | present |
| LAN Cockpit rule | present | present |
| LAN code-server TCP `8787` rule | present | present |

Authentication:

| Check | Result |
|---|---|
| `curl -I http://10.42.42.77:8787/` | `302 Found` |
| Redirect | `./login` |

Representative repository:

| Check | Result |
|---|---|
| Path | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| Latest commit | `d09559b Validate browser VS Code iPad workflow` |
| Git status | clean; no status output before log |

Conclusion: code-server is operational, authenticated, reboot-safe, LAN-scoped, and validated from iPad Safari against the #15 representative repository. The browser IDE is now in the house, but it is wearing a badge, standing behind a locked door, and not yelling its port number at the Internet.
