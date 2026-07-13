# Forge Host Baseline Runbook

## Status

Review-ready for story #13.

Related stories:

- [#12 Establish secure SSH access from the MacBook](https://github.com/stfolder/home-platform/issues/12)
- [#13 Configure host firewall and update policy](https://github.com/stfolder/home-platform/issues/13)
- [#18 Validate VPN-only access path](https://github.com/stfolder/home-platform/issues/18)

## Policy Summary

Forge exposes only explicitly approved administration services on the home LAN. SSH is the primary administration path and must remain key-only as documented in [Forge SSH Access Runbook](ssh-access.md).

Approved inbound access for story #13:

| Service | Decision | Source boundary | Notes |
|---|---|---|---|
| SSH | Keep | `10.42.42.0/24` home LAN only | Required for administration and development access. |
| Cockpit | Keep | `10.42.42.0/24` home LAN only | Approved browser console for local administration and recovery. |
| DHCPv6 client allowance | Remove / defer | n/a | Forge currently uses IPv4 DHCP on the home LAN; no current story requires inbound DHCPv6 allowance. |
| Development ports | Defer | n/a | Open only when a future story owns a specific service and validation plan. |
| Database ports | Defer | n/a | No database service is intentionally exposed from Forge in this story. |
| Container daemon/API ports | Defer | n/a | No host Docker or container API is intentionally exposed. |
| AI, IDE, and application ports | Defer | n/a | Future stories must open only the ports they own. |

No router port-forward, public DNS, Internet-facing SSH, VPN configuration, fail2ban, SSH port change, or application-specific opening is included in this story.

## Baseline Capture

Captured on 2026-07-13 during story #13.

### Host And Services

| Check | Value |
|---|---|
| Hostname | `forge` |
| Primary interface | `eno2` |
| IPv4 address | `10.42.42.77/24` |
| Default route | `10.42.42.1` via `eno2` |
| Wi-Fi | disconnected |
| `firewalld` | enabled and active |
| `sshd` | enabled and active |
| Failed systemd units | none |

### firewalld Baseline Before Changes

| Item | Runtime value |
|---|---|
| State | `running` |
| Active zone | `FedoraServer` |
| Bound interface | `eno2` |
| Target | `default` |
| Services | `cockpit dhcpv6-client ssh` |
| Ports | none |
| Protocols | none |
| Rich rules | none |
| Forward | `yes` |
| Masquerade | `no` |

Permanent baseline differs from runtime because `eno2` is bound at runtime but not listed in the permanent zone configuration. Final policy must make runtime and permanent state consistent.

### Listening Socket Baseline Before Changes

| Socket | Process | Decision | Notes |
|---|---|---|---|
| TCP `0.0.0.0:22`, `[::]:22` | `sshd` | Keep, restrict through firewalld | Required for `ssh forge`. |
| TCP `*:9090` | `cockpit-tls` | Keep, restrict through firewalld | Approved browser console for local administration and recovery. |
| TCP/UDP `0.0.0.0:5355`, `[::]:5355` | `systemd-resolved` | Disable LLMNR | Local DNS uses `forge.home.arpa`; LLMNR is not needed. |
| TCP/UDP loopback `:53` | `systemd-resolved` | Keep | Local resolver stub; not externally exposed. |
| UDP loopback `:323` | `chronyd` | Keep | Local chrony socket; not externally exposed. |

### LAN Reachability Before Changes

Checked from the MacBook.

| Port | Result |
|---:|---|
| 22 | reachable |
| 9090 | reachable |
| 80 | refused |
| 443 | refused |
| 3000 | refused |
| 5432 | refused |
| 8080 | refused |
| 8443 | refused |

`nmap` was not available on the MacBook path used during capture, so `nc` was used for initial TCP checks.

## Firewall Implementation

Keep one existing `ssh forge` session open while applying changes. Use the local Forge console if SSH access is lost.

Create a firewalld backup:

```bash
sudo cp -a /etc/firewalld "/etc/firewalld.bak-story-13-$(date +%Y%m%d%H%M%S)"
```

Apply the permanent policy:

```bash
sudo firewall-cmd --permanent --zone=FedoraServer --add-interface=eno2
sudo firewall-cmd --permanent --zone=FedoraServer --remove-service=ssh
sudo firewall-cmd --permanent --zone=FedoraServer --remove-service=cockpit
sudo firewall-cmd --permanent --zone=FedoraServer --remove-service=dhcpv6-client
sudo firewall-cmd --permanent --zone=FedoraServer --remove-forward
sudo firewall-cmd --permanent --zone=FedoraServer --add-rich-rule='rule family="ipv4" source address="10.42.42.0/24" service name="ssh" accept'
sudo firewall-cmd --permanent --zone=FedoraServer --add-rich-rule='rule family="ipv4" source address="10.42.42.0/24" service name="cockpit" accept'
```

Validate and reload:

```bash
sudo firewall-cmd --check-config
sudo firewall-cmd --reload
```

Confirm runtime and permanent state:

```bash
sudo firewall-cmd --list-all
sudo firewall-cmd --list-all --permanent
```

From a second MacBook terminal:

```bash
ssh forge hostname
```

Expected result:

```text
forge
```

## Cockpit

Cockpit is kept enabled for the host baseline and restricted to the home LAN by firewalld rich rule:

```bash
sudo systemctl enable --now cockpit.socket
```

Validate:

```bash
systemctl is-enabled cockpit.socket
systemctl is-active cockpit.socket
```

Do not expose Cockpit through router port forwarding, public DNS, dynamic DNS, or Internet-facing firewall rules. If the source boundary changes, update the firewalld rich rule and this runbook in the same story.

## LLMNR

LLMNR is disabled because the platform uses the canonical DNS name `forge.home.arpa`.

Create the systemd-resolved drop-in:

```bash
sudo mkdir -p /etc/systemd/resolved.conf.d
sudo tee /etc/systemd/resolved.conf.d/10-home-platform.conf >/dev/null <<'EOF'
[Resolve]
LLMNR=no
EOF
sudo systemctl restart systemd-resolved
```

Validate that `:5355` listeners are gone:

```bash
sudo ss -lntup | grep 5355 || true
```

## Update Policy

Selected approach: automatic security updates without automatic reboot, with manual review for broader updates.

Rationale:

- Forge is a personal development server and should remain predictable.
- Security updates should not depend entirely on memory.
- Unattended reboot could interrupt work or break remote access at an awkward time.
- Full upgrades remain manual so package changes can be reviewed.

Install and configure DNF automatic updates. Fedora 44 uses DNF5 and the `dnf5-automatic.timer`; host-specific overrides are read from `/etc/dnf/automatic.conf`, which may need to be created.

```bash
sudo dnf install -y dnf-automatic
if [ -f /etc/dnf/automatic.conf ]; then
  sudo cp -a /etc/dnf/automatic.conf "/etc/dnf/automatic.conf.bak-story-13-$(date +%Y%m%d%H%M%S)"
fi
sudo tee /etc/dnf/automatic.conf >/dev/null <<'EOF'
[commands]
apply_updates = yes
download_updates = yes
upgrade_type = security
reboot = never

[emitters]
emit_via = motd
emit_no_updates = no
EOF
sudo systemctl enable --now dnf5-automatic.timer
```

Validate:

```bash
rpm -q dnf5-plugin-automatic
grep -E '^(upgrade_type|apply_updates|reboot) =' /etc/dnf/automatic.conf
systemctl is-enabled dnf5-automatic.timer
systemctl is-active dnf5-automatic.timer
systemctl list-timers 'dnf5-automatic*' --no-pager
```

Automatic reboot must remain disabled.

## Manual Maintenance

Refresh metadata and apply regular updates manually:

```bash
sudo dnf upgrade --refresh
```

Inspect recent package activity:

```bash
dnf history list
dnf history info last
```

Check whether a reboot is advisable:

```bash
needs-restarting -r
```

If `needs-restarting` is unavailable, install the supporting package deliberately:

```bash
sudo dnf install -y dnf-utils
```

Fedora package rollback is not guaranteed to be complete or risk-free. Prefer fixing forward unless a specific package transaction is clearly reversible and the risk is understood.

## Validation Checklist

After applying firewall and update changes:

```bash
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
sudo firewall-cmd --list-all --permanent
sudo ss -lntup
systemctl is-enabled firewalld sshd
systemctl is-active firewalld sshd
systemctl --failed --no-pager
ssh forge hostname
```

From the MacBook:

```bash
for p in 22 9090 80 443 3000 5432 8080 8443; do
  nc -vz -G 2 forge.home.arpa "$p"
done
```

Expected LAN reachability after changes:

| Port | Expected result |
|---:|---|
| 22 | reachable from `10.42.42.0/24` |
| 9090 | reachable from `10.42.42.0/24` |
| 80 | not reachable |
| 443 | not reachable |
| 3000 | not reachable |
| 5432 | not reachable |
| 8080 | not reachable |
| 8443 | not reachable |

Reboot Forge after validation:

```bash
sudo systemctl reboot
```

From the MacBook, wait for Forge to return:

```bash
until ssh -o BatchMode=yes -o ConnectTimeout=5 forge 'hostname && uptime'; do
  echo "Waiting for Forge..."
  sleep 10
done
```

Repeat the firewall, socket, update, failed-unit, and LAN reachability checks after reboot.

## Recovery

If SSH is blocked:

1. Log in through the local Forge console.
2. Inspect the firewalld backup under `/etc/firewalld.bak-story-13-*`.
3. Restore or adjust the `FedoraServer` zone.
4. Validate configuration:

```bash
sudo firewall-cmd --check-config
```

5. Reload firewalld:

```bash
sudo firewall-cmd --reload
```

6. Validate from the MacBook:

```bash
ssh forge hostname
```

If firewalld itself is preventing recovery and local console access is available, temporarily stop firewalld only long enough to restore the intended configuration:

```bash
sudo systemctl stop firewalld
```

Re-enable and validate firewalld before considering the host recovered:

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --state
```

## Results

Current story #13 validation results:

| Check | Result | Evidence / notes |
|---|---|---|
| Runtime firewalld policy matches permanent policy | PASS | `FedoraServer` runtime and permanent both bind `eno2`, have no broad services, set `forward: no`, and allow only LAN-scoped SSH and Cockpit rich rules. |
| SSH is LAN-scoped | PASS | Rich rule allows `ssh` only from `10.42.42.0/24`. |
| Cockpit is LAN-scoped | PASS | Rich rule allows `cockpit` only from `10.42.42.0/24`. |
| Broad `ssh`, `cockpit`, and `dhcpv6-client` services removed | PASS | `services:` is empty in runtime and permanent `FedoraServer` zone output. |
| LLMNR disabled | PASS | `sudo ss -lntup` shows no TCP or UDP listener on `:5355`. |
| Approved administration ports reachable from LAN | PASS | MacBook checks show TCP `22` and `9090` reachable. |
| Common development and application ports closed | PASS | MacBook checks show TCP `80`, `443`, `3000`, `5432`, `8080`, and `8443` refused. |
| SSH survives firewall reload | PASS | Fresh `ssh forge` session returned `forge` after reload. |
| Automatic security updates configured | PASS | `dnf5-plugin-automatic` installed; `dnf5-automatic.timer` enabled and active. |
| Automatic reboot disabled | PASS | `/etc/dnf/automatic.conf` sets `reboot = never`. |
| Update scope is security-only | PASS | `/etc/dnf/automatic.conf` sets `upgrade_type = security` and `apply_updates = yes`. |
| Firewall policy survives reboot | PASS | Post-reboot runtime and permanent `FedoraServer` zone both retain empty `services`, `forward: no`, interface `eno2`, and LAN-scoped SSH/Cockpit rich rules. |
| SSH survives reboot | PASS | Fresh `ssh forge` session succeeded after reboot. |
| Approved LAN reachability survives reboot | PASS | MacBook checks show TCP `22` and `9090` reachable after reboot. |
| Unapproved LAN ports remain closed after reboot | PASS | MacBook checks show TCP `80`, `443`, `3000`, `5432`, `8080`, and `8443` refused after reboot. |
| Update timer survives reboot | PASS | `dnf5-automatic.timer` remains enabled and active after reboot. |
| Failed units | PASS | `systemctl --failed` reports zero failed units. |

Story #13 host firewall and update-policy validation is complete.
