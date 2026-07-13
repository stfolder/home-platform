# Forge Network Identity

## Status

Network identity validation is complete for story #11.

Related stories:

- [#10 Install and validate Fedora Server on Forge](https://github.com/stfolder/home-platform/issues/10)
- [#11 Establish Forge's permanent network identity](https://github.com/stfolder/home-platform/issues/11)
- [#12 Establish secure SSH access from the MacBook](https://github.com/stfolder/home-platform/issues/12)
- [#17 Generate Forge system inventory from the running host](https://github.com/stfolder/home-platform/issues/17)

This document records the human-reviewed network identity for Forge. Unique machine identifiers, such as full MAC addresses and generated hardware identifiers, are intentionally excluded from this public repository. Exact values belong in private or host-local inventory produced by #17.

## Identity

| Field | Value | Status | Notes |
|---|---|---|---|
| Permanent hostname | `forge` | Configured | Set on Fedora with `hostnamectl`. |
| Canonical local DNS name | `forge.home.arpa` | Configured | Uses `.home.arpa` to avoid `.local` mDNS conflicts. |
| Temporary install hostname | `fedora` | Retired | Used during #10 only. |
| Stable LAN address | `10.42.42.77` | Configured | Reserved by DHCP, not statically configured in Fedora. |
| Addressing strategy | DHCP reservation | Configured | pfSense is the source of truth. |
| Fedora static IP configuration | None | Required | Static IP must not be configured inside Fedora unless an ADR documents the exception. |

## Network Interface

| Field | Value | Status | Notes |
|---|---|---|---|
| Primary interface | `eno2` | Verified from Forge | Wired Ethernet interface. |
| Reservation identifier | Verified wired-interface MAC | Verified from Forge | Exact MAC is retained only in pfSense and private or host-local inventory. |
| Interface source command | `ip link`, `nmcli device status`, `nmcli device show` | Reference | Full generated inventory is deferred to #17. |

## Host Configuration Baseline

| Check | Value | Status | Notes |
|---|---|---|---|
| RTC mode | UTC | Configured | `timedatectl` reports `RTC in local TZ: no`. |
| Timezone | `America/Los_Angeles` | Configured | NTP is active and synchronized. |
| Wired connection | `eno2` connected | Configured | Default route uses `eno2` via `10.42.42.1`. |
| Wi-Fi connection | `wlo1` disconnected | Configured | Wi-Fi is not part of Forge's stable network identity. |
| firewalld | `running` | Verified | `sudo firewall-cmd --state` returns `running`. |
| Failed systemd units | none | Verified | `systemctl --failed` reports zero failed units after cleanup. |

## pfSense Configuration

| Item | Value |
|---|---|
| Configuration owner | pfSense |
| Configuration area | DHCP static mapping / reservation for the LAN interface |
| Hostname | `forge` |
| Domain | `home.arpa` |
| DNS name | `forge.home.arpa` |
| Reservation identity | Forge's verified wired-interface MAC; exact value retained in pfSense |
| Reserved IP | `10.42.42.77` |

Do not store pfSense credentials, backup exports containing secrets, VPN keys, private certificates, or complete unique hardware identifiers in this public repository.

## Validation Procedure

Run on Forge after changing the hostname and renewing DHCP:

```bash
hostnamectl
cat /etc/hostname
ip addr
ip route
resolvectl status
```

Run from an approved LAN client:

```bash
getent hosts forge.home.arpa
ping forge.home.arpa
ssh forge.home.arpa
```

Run from at least one additional LAN device:

```bash
ping forge.home.arpa
```

SSH validation for story #11 only proves hostname resolution reaches the host. SSH key hardening and remote-access security are documented in [Forge SSH Access Runbook](../runbooks/ssh-access.md).

## Validation Results

| Check | Result | Evidence / notes |
|---|---|---|
| Permanent hostname is `forge` | PASS | `hostnamectl` and `/etc/hostname` verified on Forge. |
| Hostname persists after reboot | PASS | Verified after reboot. |
| DHCP reservation configured | PASS | pfSense reservation created for Forge's verified wired interface. |
| Fedora does not use static IP | PASS | Addressing remains DHCP-based. |
| Canonical DNS name resolves | PASS | `forge.home.arpa` resolves to `10.42.42.77` from LAN. |
| Forward lookup from LAN succeeds | PASS | `ping forge.home.arpa` from the Mac resolved `10.42.42.77`. |
| DHCP lease renewal survives | PASS | Verified after lease renewal or reboot. |
| Ping by hostname succeeds | PASS | `ping forge.home.arpa` from the Mac returned replies from `10.42.42.77`. |
| SSH hostname resolution succeeds | PASS | `ssh forge.home.arpa` reaches the host and returns hostname `forge`. |
| Additional LAN device validates connectivity | PASS | Verified by user during story #11 validation. |
| Stable route uses wired Ethernet | PASS | Default route is via `10.42.42.1` on `eno2` with source `10.42.42.77`. |
| Wi-Fi is disconnected | PASS | `nmcli device status` reports `wlo1` disconnected. |

## Recovery Procedure

If Forge loses its reservation or DNS entry:

1. Log in locally to Forge.
2. Confirm the hostname:

```bash
hostnamectl
cat /etc/hostname
```

3. Identify the wired interface and MAC address:

```bash
ip link
nmcli device status
nmcli device show
```

4. In pfSense, recreate the LAN DHCP reservation for the verified wired-interface MAC address.
5. Set the hostname to `forge` and local domain to `home.arpa`.
6. Renew DHCP on Forge:

```bash
sudo nmcli networking off
sudo nmcli networking on
```

7. Validate `forge.home.arpa` from at least one LAN client.

## Security Notes

- No credentials, private keys, pfSense backup exports containing secrets, DHCP lease files, or complete unique hardware identifiers are committed.
- Forge uses DHCP reservation rather than a static Fedora IP configuration.
- Stable network identity does not imply public exposure.
- Remote access security, SSH key policy, and firewall hardening are handled by later stories.
- Secure SSH access from the MacBook was completed in story #12; broader firewall hardening remains separate.
