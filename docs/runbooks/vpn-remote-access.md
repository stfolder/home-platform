# Forge VPN Remote Access Runbook

## Status

In progress for story #18.

Validated so far:

- pfSense WireGuard is installed and running.
- COX upstream port forwarding delivers WireGuard UDP traffic to pfSense.
- MacBook external WireGuard access works from iPhone hotspot.
- iPad cellular WireGuard peer handshakes with pfSense.
- iPad reaches Forge browser VS Code by IP and `forge.home.arpa` over VPN.
- iPad Safari reaches Synology DSM over VPN.
- MacBook reaches Raspberry Pi `tinyberry` over VPN by SSH.
- VPN disconnect removes MacBook access to Forge private services.
- Independent peer revocation was tested.
- pfSense reboot recovery was tested successfully.

Remaining validation is limited to final bookkeeping and any optional future services that are not currently present on the LAN.

Related stories:

- [#15 Validate IntelliJ, VS Code, and iPad remote development workflows](https://github.com/stfolder/home-platform/issues/15)
- [#18 Establish trusted-device VPN access to the home network](https://github.com/stfolder/home-platform/issues/18)
- [#20 Deploy a private browser VS Code workspace on Forge](https://github.com/stfolder/home-platform/issues/20)
- [#21 Add trusted HTTPS for Forge browser VS Code](https://github.com/stfolder/home-platform/issues/21)

## Design

Story #18 uses a trusted-device routed VPN model.

```text
Away from home:
  approved MacBook/iPad -> WireGuard -> pfSense -> home LAN

Public Internet:
  pfSense WireGuard endpoint only
  no direct Forge, Synology, Raspberry Pi, Mac GUI, KVM, SSH, database, Docker, or browser IDE exposure
```

This is intentionally not a guest or zero-trust model. MacBook and iPad are trusted administrator devices with broad routed access to the home LAN. Host authentication and host firewalls still matter.

## Network Plan

| Network | Value |
|---|---|
| Home LAN | `10.42.42.0/24` |
| pfSense LAN | `10.42.42.1` |
| pfSense WAN behind COX router | `192.168.0.105/24` |
| COX-side gateway | `192.168.0.1` |
| VPN subnet | `10.20.30.0/24` |
| pfSense WireGuard address | `10.20.30.1/24` |
| MacBook WireGuard address | `10.20.30.2/32` |
| iPad WireGuard address | `10.20.30.3/32` |
| WireGuard UDP port | `51820` |
| VPN DNS resolver | `10.42.42.1` |
| Split-tunnel routes | `10.42.42.0/24`, `10.20.30.0/24` |

The COX router is upstream of pfSense. The COX router forwards UDP `51820` to pfSense WAN `192.168.0.105`. pfSense remains the VPN endpoint and the home-network policy owner.

## Baseline

pfSense:

| Check | Result |
|---|---|
| Hostname | `gateway.home.arpa` |
| Version | pfSense CE `2.7.2-RELEASE` |
| Update notice | `2.8.1` available |
| WAN | DHCP, `192.168.0.105/24` |
| WAN gateway | `192.168.0.1` |
| LAN | `10.42.42.1/24` |
| DNS Resolver | enabled |
| Installed packages before #18 | none |
| VPN menu before WireGuard install | IPsec, L2TP, OpenVPN |
| NAT Port Forward before #18 | empty |
| WAN rules before #18 | no pass rules; default private/bogon blocks present |

DNS Resolver host overrides:

| Host | Result |
|---|---|
| `forge.home.arpa` | `10.42.42.77` |
| `nas1.home.arpa` | `10.42.42.5` |
| `postgre.home.arpa` | alias for `nas1.home.arpa` |
| `home.arpa` domain override | `10.42.42.1` |

## COX Upstream Router

pfSense WAN is on the COX router's internal network. Inbound WireGuard requires upstream forwarding.

COX router setting:

| Setting | Value |
|---|---|
| Port forward | UDP `51820` |
| Forward target | pfSense WAN `192.168.0.105` |
| Forwarded service | WireGuard only |

Do not forward SSH, Cockpit, browser VS Code, Synology, KVM, remote desktop, Docker, databases, or any other home service on the COX router.

## pfSense WireGuard Setup

WireGuard package:

| Check | Result |
|---|---|
| Package | `pfSense-pkg-WireGuard` |
| Version | `0.2.1` |
| Menu | `VPN > WireGuard` present |

Tunnel:

| Field | Value |
|---|---|
| Description | `WG_HOME_TRUSTED` |
| Interface | `tun_wg0` |
| Address | `10.20.30.1/24` |
| Listen port | `51820` |
| Peers | `macbook-trusted`, `ipad-trusted` |

Peer assignments:

| Peer | Allowed IP |
|---|---|
| `macbook-trusted` | `10.20.30.2/32` |
| `ipad-trusted` | `10.20.30.3/32` |

Peer private keys, preshared keys, full profiles, and QR codes must remain outside Git.

## pfSense Firewall Policy

WAN rule:

| Field | Value |
|---|---|
| Action | pass |
| Interface | WAN |
| Protocol | IPv4 UDP |
| Source | any |
| Source port | any |
| Destination | WAN address |
| Destination port | `51820` |
| Purpose | allow WireGuard to pfSense |

The source port must be `any`. Mobile WireGuard clients use random high source ports. A rule that requires source port `51820` will not match.

WireGuard tab rule:

| Field | Value |
|---|---|
| Action | pass |
| Interface | WireGuard |
| Protocol | any |
| Source | `10.20.30.0/24` |
| Destination | `10.42.42.0/24` |
| Purpose | allow trusted WireGuard peers to home LAN |

## Forge Host Firewall

Forge firewalld initially allowed SSH, Cockpit, and code-server only from the LAN subnet `10.42.42.0/24`. VPN clients arrive from `10.20.30.0/24`, so Forge also needs VPN-scoped rich rules.

Required Forge rules:

```bash
sudo firewall-cmd --permanent --zone=FedoraServer \
  --add-rich-rule='rule family="ipv4" source address="10.20.30.0/24" service name="ssh" accept'

sudo firewall-cmd --permanent --zone=FedoraServer \
  --add-rich-rule='rule family="ipv4" source address="10.20.30.0/24" service name="cockpit" accept'

sudo firewall-cmd --permanent --zone=FedoraServer \
  --add-rich-rule='rule family="ipv4" source address="10.20.30.0/24" port port="8787" protocol="tcp" accept'

sudo firewall-cmd --reload
```

These rules preserve host-level scoping: Forge allows VPN clients without opening services to all sources.

## Validation Evidence

### COX Forwarding

Status: PASS.

pfSense WAN packet capture saw external cellular WireGuard initiation packets arriving at pfSense:

```text
external-cellular-client:<ephemeral-port> > 192.168.0.105:51820 UDP length 148
```

This proves the COX UDP port forward reaches pfSense. Public client IPs and public endpoint addresses are intentionally not committed.

### iPad Cellular WireGuard

Status: PASS.

After correcting the WAN rule source port to `any`, pfSense WireGuard status showed the iPad handshake as active.

The iPad, with Wi-Fi disabled and WireGuard enabled over cellular, reached Forge browser VS Code:

| Test | Result |
|---|---|
| `http://10.42.42.77:8787/` | PASS |
| `http://forge.home.arpa:8787/` | PASS |

This confirms:

- WireGuard endpoint is reachable externally through the COX router.
- pfSense routes VPN clients to the home LAN.
- pfSense DNS Resolver answers private `home.arpa` names for VPN clients.
- Forge host firewall accepts the VPN subnet for browser VS Code.

## MacBook Client

Local MacBook peer material was generated outside the repository under:

```text
/Users/serge/.wireguard/home-platform/
```

Tracked files must never include WireGuard private keys or full client configs.

MacBook config shape:

```ini
[Interface]
PrivateKey = <redacted>
Address = 10.20.30.2/24
DNS = 10.42.42.1

[Peer]
PublicKey = <pfSense tunnel public key>
AllowedIPs = 10.42.42.0/24, 10.20.30.0/24
Endpoint = <public endpoint>:51820
PersistentKeepalive = 25
```

### MacBook External Validation

Status: PASS.

Validated from MacBook using iPhone hotspot as an external network.

| Check | Result |
|---|---|
| Default route before VPN | iPhone hotspot gateway, not home LAN |
| VPN routes | `10.20.30.0/24` and `10.42.42.0/24` via WireGuard |
| `dig @10.42.42.1 nas1.home.arpa` | PASS |
| `dig @10.42.42.1 gateway.home.arpa` | PASS |
| `dig @10.42.42.1 forge.home.arpa` | one transient timeout observed |
| `ping 10.42.42.77` | PASS, 0% packet loss |
| `nc forge.home.arpa 22` | PASS |
| `nc forge.home.arpa 8787` | PASS |
| `curl -I http://forge.home.arpa:8787/` | PASS, `302 Found` to `./login` |
| `ssh forge 'hostname; whoami; date -Is'` | PASS, returned `forge`, `serge`, timestamp |

Representative Forge latency over iPhone hotspot:

```text
round-trip min/avg/max/stddev = 37.768/69.501/104.518/27.349 ms
```

After bringing the VPN down, MacBook access to Forge private services failed as expected:

| Check after VPN down | Result |
|---|---|
| `nc forge.home.arpa 22` | timed out |
| `nc forge.home.arpa 8787` | timed out |
| `ping 10.42.42.77` | 100% packet loss |

Conclusion: MacBook external split-tunnel access to Forge is functional, and disconnecting the VPN removes private Forge reachability.

## Synology Validation

Status: PASS.

Validated so far:

| Check | Result |
|---|---|
| iPad Safari over WireGuard cellular VPN | DSM web console reachable |
| MacBook over WireGuard external VPN | Synology reachable |
| MacBook SMB/file workflow | PASS |
| Additional Synology service | PostgreSQL for the finance app is reachable |
| Private host | `nas1.home.arpa` / `10.42.42.5` |
| Synology authentication | remains required |

Notes:

- iPad file/package workflow is not required because the practical iPad Synology workflow is DSM web access.
- MacBook SMB covers the preferred file workflow requirement.
- PostgreSQL access covers an additional intentionally enabled Synology-hosted service.

No direct Synology public forwarding was introduced.

## Raspberry Pi Validation

Status: PASS.

Validated so far:

| Check | Result |
|---|---|
| Host | `tinyberry` |
| Private address | `10.42.42.177` |
| Address type | static IP |
| Client | MacBook over WireGuard external VPN |
| Administration path | SSH reachable |
| Application endpoint | Finance app reachable at `http://10.42.42.177/` |

Notes:

- Private DNS name can be added later if desired; static IP is currently documented.
- No Raspberry Pi public forwarding was introduced.

## Operations

### Preferred Client Toggle

Use the WireGuard apps for normal daily operation:

| Client | Preferred enable/disable path |
|---|---|
| MacBook | WireGuard macOS app toggle |
| iPad | WireGuard iPadOS app toggle |

The CLI remains useful for diagnostics, but the app toggle is the normal operator path. Revocation remains a pfSense action: disable the individual peer under `VPN > WireGuard > Peers`.

### Bring Up MacBook WireGuard

```bash
sudo /opt/homebrew/bin/wg-quick up "$HOME/.wireguard/home-platform/wg-home-trusted.conf"
```

### Show MacBook WireGuard State

```bash
sudo /opt/homebrew/bin/wg show
```

Do not paste private keys. Redact public endpoint addresses if publishing is not desired.

### Bring Down MacBook WireGuard

```bash
sudo /opt/homebrew/bin/wg-quick down "$HOME/.wireguard/home-platform/wg-home-trusted.conf"
```

### pfSense Packet Capture

Use this when debugging handshake:

```text
Diagnostics > Packet Capture
Interface: WAN
Filter preset: Custom Filter
Protocol: UDP
Port: 51820
Packet count: 50
Name lookup: unchecked
```

Interpretation:

| Capture result | Meaning |
|---|---|
| No UDP `51820` packets | COX port forward, public endpoint, or upstream issue |
| UDP `51820` arrives but no handshake | pfSense WAN rule, tunnel, peer, or key issue |
| Handshake works but LAN fails | WireGuard firewall rule, LAN routing, or destination host firewall issue |
| IP works but hostname fails | DNS Resolver or client DNS issue |

## Remaining Work

- Mac GUI access is deferred because no target Mac GUI service is currently present on the LAN.
- Network KVM access is deferred because no KVM service is currently present on the LAN.
- Update #15 and #20 when outside-LAN workflows are complete.

## Revocation, Negative Testing, And Reboot

### Peer Revocation

Status: PASS.

Independent peer revocation was tested. Disabling one WireGuard peer in pfSense does not require removing the tunnel or breaking the other trusted device. Re-enable the peer after validation if the device remains approved.

Operational path:

```text
VPN > WireGuard > Peers > edit peer > Enable unchecked > Save > Apply
```

### VPN Disconnect Negative Test

Status: PASS.

MacBook external testing confirmed that private Forge access disappears after WireGuard disconnect:

| Check after VPN down | Result |
|---|---|
| Forge SSH | timed out |
| Forge browser VS Code TCP `8787` | timed out |
| Forge ping | 100% packet loss |

### pfSense Reboot Recovery

Status: PASS.

After a normal pfSense reboot:

| Check | Result |
|---|---|
| pfSense reboot | PASS |
| WireGuard tunnel after reboot | present |
| iPad cellular to `forge.home.arpa:8787` | PASS |
| SSH over VPN after reboot | PASS |
| MacBook over VPN after reboot | PASS |
| Failed services or alerts | none observed |

Conclusion: pfSense reboot restores WireGuard, DNS, routes, and policy for the validated clients.

## Deferred LAN Services

Mac GUI and network KVM validation are deferred. They remain part of the long-term architecture, but there is no current Mac GUI remote-desktop service or network KVM service present on the LAN to validate in story #18.

When those services exist, validate them through the existing trusted-device VPN path and keep direct public exposure prohibited.

## Security Notes

- Do not commit WireGuard private keys.
- Do not commit preshared keys.
- Do not commit complete client profiles.
- Do not commit QR codes.
- Do not commit raw pfSense exports.
- Keep COX router forwarding limited to UDP `51820` for pfSense WireGuard.
- Keep direct public exposure prohibited for Forge, Synology, Raspberry Pi, Mac GUI, KVM, Docker, databases, and browser IDE services.
