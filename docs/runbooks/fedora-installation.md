# Fedora Server Installation Runbook

## Status

Runbook status: Fedora Server installed and validated for story #10.

Related stories:

- [#9 Prepare Forge for Fedora installation](https://github.com/stfolder/home-platform/issues/9)
- [#10 Install and validate Fedora Server on Forge](https://github.com/stfolder/home-platform/issues/10)

Follow-up:

- [#16 Investigate Forge 2 TB HDD SMART sector warnings](https://github.com/stfolder/home-platform/issues/16)

This runbook records the Fedora Server installation and validation results for Forge. The original pre-install instructions are retained as historical installation procedure.

## References

- Fedora Server download page: https://fedoraproject.org/server/download/
- Fedora installation documentation: https://docs.fedoraproject.org/
- Forge hardware preparation: `docs/hardware/forge.md`
- Sprint reference: `docs/sprints/sprint-001.md`

## Installation Scope

Install Fedora Server on Forge as the home engineering platform development workstation.

Target:

- Architecture: x86_64.
- Temporary hostname: `fedora`.
- Local DNS name after later LAN configuration: `forge.home.arpa`.
- Installation target disk: Samsung 970 EVO Plus.

Out of scope for this runbook:

- Ansible provisioning.
- pfSense automation.
- KVM automation.
- NVIDIA driver installation.
- Kubernetes.
- Public service exposure.

## Pre-Install Stop Conditions

Stop and do not install Fedora if any of the following are true:

- The target installation disk has not been documented and confirmed.
- Any disk has an unknown disposition.
- Important data has not been backed up.
- The Fedora image checksum has not been verified.
- Wired Ethernet is unavailable.
- UPS power path is not understood.
- Firmware settings cannot be reviewed.
- There is uncertainty about which disk will be formatted.

## Fedora Download Source

Use the official Fedora Server download page:

```text
https://fedoraproject.org/server/download/
```

Confirm the current stable Fedora Server release on installation day before downloading. Record the selected version in `docs/hardware/forge.md`.

Preferred image:

```text
Fedora Server x86_64 DVD ISO
```

Alternative image:

```text
Fedora Server x86_64 Network Install ISO
```

Use the DVD ISO unless there is a reason to prefer the smaller network installer.

## Checksum Verification Procedure

Fedora states that downloaded images should be verified for security and integrity, and that images are signed with Fedora Project OpenPGP keys.

### Option A: Fedora Media Writer

Use Fedora Media Writer when available. It is the simplest path for creating Fedora installation media and reduces manual USB-writing mistakes.

Procedure:

1. Install Fedora Media Writer on the MacBook.
2. Select Fedora Server.
3. Select the current stable x86_64 Fedora Server image.
4. Let Fedora Media Writer download and write the USB media.
5. Record completion in `docs/hardware/forge.md`.

### Option B: Manual ISO Verification

Use this path if downloading the ISO directly.

1. Download the Fedora Server x86_64 ISO from the official Fedora Server download page.
2. Download the matching checksum file from Fedora's verification instructions for that image.
3. Import or refresh Fedora's signing keys if required by the verification instructions.
4. Verify the checksum file signature.
5. Verify the ISO checksum.

Example commands on macOS or Linux, adjusted for the actual downloaded filenames:

```bash
gpg --verify Fedora-Server-CHECKSUM
shasum -a 256 --check Fedora-Server-CHECKSUM
```

If `shasum --check` does not support the checksum file format on the MacBook, calculate the ISO checksum directly and compare it manually with the checksum published by Fedora:

```bash
shasum -a 256 Fedora-Server-dvd-x86_64-*.iso
```

Do not use the ISO if verification fails or if the checksum source does not match the downloaded image.

## USB Creation Procedure

### Preferred: Fedora Media Writer

1. Insert a USB drive of at least 4 GB.
2. Open Fedora Media Writer.
3. Choose Fedora Server x86_64.
4. Select the USB drive.
5. Confirm the USB drive can be erased.
6. Write the image.
7. Eject the USB drive cleanly.

### Manual: macOS `diskutil` and `dd`

Use this only when Fedora Media Writer is unavailable.

1. Insert the USB drive.
2. Identify the USB disk:

```bash
diskutil list
```

3. Unmount the USB disk, replacing `N` with the correct disk number:

```bash
diskutil unmountDisk /dev/diskN
```

4. Write the ISO to the raw device:

```bash
sudo dd if=Fedora-Server-dvd-x86_64.iso of=/dev/rdiskN bs=4m status=progress
```

5. Flush writes and eject:

```bash
sync
diskutil eject /dev/diskN
```

The `dd` command is destructive. Confirm the USB disk number before pressing Enter.

## BIOS or UEFI Preparation

Before booting the installer:

1. Connect Forge to wired Ethernet.
2. Confirm Forge and pfSense are on UPS-backed power.
3. Enter BIOS or UEFI setup.
4. Confirm UEFI boot mode.
5. Enable VT-x.
6. Enable VT-d if available.
7. Review Secure Boot and record the decision in `docs/hardware/forge.md`.
8. Review detected disks and update the disk inventory in `docs/hardware/forge.md`.
9. Evaluate whether a BIOS or UEFI update is needed.
10. Set USB installation media as the temporary first boot device.
11. Save settings and reboot to the Fedora installer.

Recommended safety step:

- Disconnect non-target internal disks before installation when practical.

## Installation Steps

1. Boot from Fedora Server USB media.
2. Select the Fedora Server installer.
3. Choose language and keyboard layout.
4. Configure installation destination.
5. Match the installer disk list against `docs/hardware/forge.md`.
6. Select only the disk marked `wipe`.
7. Do not select disks marked `keep` or `reuse later`.
8. Use automatic partitioning unless a later requirement calls for a custom layout.
9. Configure networking if prompted.
10. Set hostname to:

```text
forge
```

11. Create a non-root administrative user.
12. Use a strong temporary local password for first login.
13. Do not enable public remote access during installation.
14. Begin installation only after the target disk is confirmed a final time.
15. Reboot when installation completes.
16. Remove USB installation media.
17. Restore boot order so the Fedora target disk boots first.

## Initial Login Procedure

1. Log in locally on Forge using the non-root administrative user created during installation.
2. Confirm hostname:

```bash
hostnamectl
```

3. Confirm Fedora version:

```bash
cat /etc/fedora-release
```

4. Confirm network link:

```bash
ip addr
ip route
```

5. Confirm package manager works:

```bash
sudo dnf check-update
```

6. Apply initial updates:

```bash
sudo dnf upgrade --refresh
```

7. Reboot if updates require it:

```bash
sudo reboot
```

## Post-Install Validation

### Installed System Summary

| Item | Value |
|---|---|
| Fedora edition | Fedora Server |
| Fedora version | Fedora 44 |
| Installation date | 2026-07-11 |
| Current hostname | `forge` |
| Target disk | Samsung 970 EVO Plus, serial `S4P2NF0M604252A` |
| Boot mode | UEFI |
| Partitioning | GPT |
| Root filesystem | XFS on LVM logical volume `fedora-root` |
| SELinux | Enforcing |
| Firewall | firewalld running |

### Partition Layout

| Device | Size | Type | Filesystem | Mount point | UUID | PARTUUID |
|---|---:|---|---|---|---|---|
| `/dev/nvme1n1` | 465.8 GB | GPT disk | n/a | n/a | n/a | n/a |
| `/dev/nvme1n1p1` | 600 MB | EFI System | vfat | `/boot/efi` | `6135-9E17` | `17bb94a4-c674-433d-b9af-ce9ba4bf1321` |
| `/dev/nvme1n1p2` | 2 GB | Linux filesystem | xfs | `/boot` | `ebb82fe3-08f0-462a-8149-2508809db012` | `23831933-1f13-46d6-a7d9-4005b863432c` |
| `/dev/nvme1n1p3` | 463.2 GB | Linux LVM | LVM2_member | LVM physical volume | `PFWKNG-abdj-P0br-xC3r-dHWv-yWC9-dGyStZ` | `44d1d82c-f457-4ccd-82fc-185609bd87aa` |
| `/dev/mapper/fedora-root` | 15 GB | LVM logical volume | xfs | `/` | `a6baaddb-7edf-42a6-8406-e355ccc61467` | n/a |

Linux device names can change across hardware changes or firmware events. Prefer model, serial number, filesystem UUID, or PARTUUID for durable identification.

After reboot:

```bash
hostnamectl
cat /etc/fedora-release
timedatectl
localectl
sudo -v
getenforce
sudo firewall-cmd --state
ip addr
ip route
ping -c 4 1.1.1.1
ping -c 4 fedoraproject.org
sudo dnf upgrade --refresh
sudo reboot
```

After the controlled reboot:

```bash
hostnamectl
systemctl --failed
journalctl -p crit -b --no-pager
lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
free -h
lscpu
```

Validation passes when:

- Forge boots from the Fedora target disk.
- Temporary hostname is assigned for #10; stable hostname and LAN identity are handled by #11.
- Wired Ethernet has link and an IP address.
- Default route exists.
- DNS resolution works.
- No unexpected failed systemd services are present.
- Disk layout matches the selected target disk.
- Non-target disks were not formatted.
- CPU and RAM match expected inventory.

### Validation Results

Date: 2026-07-11
Validator: Sergey
Validation source: local console on Forge

Validation results captured from the local console on Forge.

| Check                        | Result | Expected or observed value / notes                                                                                                                                                                                     |
| ---------------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Fedora version               | PASS   | Fedora release 44 (Forty Four)                                                                                                                                                                                         |
| Hostname                     | PASS   | `fedora` was used as the temporary #10 hostname; current permanent hostname is `forge` after #11.                                                                                                                       |
| Timezone                     | PASS   | `America/Los_Angeles`                                                                                                                                                                                                  |
| Locale                       | PASS   | System Locale: LANG=en_US.UTF-8, VC Keymap: ru                                                                                                                                                                         |
| Installation disk            | PASS   | Fedora installed on Samsung 970 EVO Plus.                                                                                                                                                                              |
| Partitioning                 | PASS   | GPT partitioning with UEFI boot.                                                                                                                                                                                       |
| Filesystems                  | PASS   | XFS for `/boot` and `/`, with `/` on LVM logical volume `fedora-root`. Reason: default Fedora Server installation, chosen for maturity, enterprise adoption, Docker/Kubernetes compatibility, and reproducibility over snapshot-oriented workflows. |
| Installation media removed   | PASS   | yes.                                                                                                                                                                                                                   |
| Boot without intervention    | PASS   | Forge boots from internal Fedora target disk without user intervention.                                                                                                                                                |
| Non-root administrative user | PASS   | Non-root admin user exists; username intentionally not documented if preferred.                                                                                                                                        |
| sudo access                  | PASS   | Administrative user can run `sudo`.                                                                                                                                                                                    |
| Wired Ethernet               | PASS   | Wired interface is set up and has link and an IP address.                                                                                                                                                              |
| Default route                | PASS   | Default route exists.                                                                                                                                                                                                  |
| Internet connectivity        | PASS   | `ping -c 4 1.1.1.1` succeeds.                                                                                                                                                                                          |
| DNS resolution               | PASS   | `ping -c 4 fedoraproject.org` succeeds.                                                                                                                                                                                |
| Package updates              | PASS   | `sudo dnf upgrade --refresh` completed successfully.                                                                                                                                                                   |
| Controlled reboot            | PASS   | Reboot completed successfully after updates.                                                                                                                                                                           |
| Failed systemd units         | PASS   | `systemctl --failed` shows no unresolved failed units.                                                                                                                                                                 |
| Critical journal errors      | PASS with caveat | Non-target 2 TB HDD `/dev/sda` reports 664 currently unreadable pending sectors and 664 offline uncorrectable sectors. SMART overall health reports `PASSED`; investigation is tracked in #16 before using this disk for platform storage. |
| SELinux                      | PASS   | `getenforce` returns `Enforcing`.                                                                                                                                                                                      |
| Firewall                     | PASS   | `sudo firewall-cmd --state` returns `running`.                                                                                                                                                                         |
| CPU                          | PASS   | Intel Core i7-9700K.                                                                                                                                                                                                   |
| RAM                          | PASS   | 32 GB.                                                                                                                                                                                                                 |

## Known Caveats

- Disk names such as `/dev/sda` and `/dev/nvme0n1` can change between boots. Prefer disk size, model, serial number, and physical port information when identifying the installation target.
- Fedora Server may list multiple installation profiles or package groups. Keep the initial installation minimal.
- Secure Boot should not be changed casually. Record the decision and reason.
- NVIDIA GPU driver work is deferred and should not be mixed into the base OS installation.
- Network interface names may not be predictable before installation. Stable LAN addressing is handled in story #11.
- The non-target 2 TB HDD is identified as `/dev/sda` and reports 664 pending sectors plus 664 offline uncorrectable sectors in the critical journal. SMART overall health reports `PASSED`; investigation is tracked in #16 before using this disk for platform storage.

## Recovery Notes

If the installer is booted but no destructive changes have been made:

1. Cancel installation.
2. Power down.
3. Review disk inventory and firmware settings.
4. Update `docs/hardware/forge.md`.

If installation fails after formatting the target disk:

1. Reboot from Fedora USB media.
2. Select the same approved target disk.
3. Reinstall Fedora Server.
4. Do not modify disks marked `keep` or `reuse later`.

If the machine cannot boot after installation:

1. Enter BIOS or UEFI setup.
2. Confirm UEFI mode.
3. Confirm the Fedora target disk is first in boot order.
4. Boot the Fedora USB media in rescue mode if needed.
5. Record the issue and create a follow-up GitHub issue if it cannot be resolved immediately.

If hardware compatibility issues appear:

1. Record the symptom, firmware version, and connected hardware.
2. Remove optional hardware where practical.
3. Retry with minimal hardware.
4. Defer optional GPU installation until the base server is stable.

## Evidence Captured

The following evidence was captured for #9 and #10:

- Target installation disk identifier and disposition.
- Disposition of detected disks.
- Wired Ethernet verification.
- UPS verification.
- UEFI mode verification.
- VT-x and VT-d verification.
- Secure Boot decision.
- Boot order verification.
- BIOS or UEFI update decision.
- Fedora media creation and checksum verification.
- Backup and recovery confirmation.
- Fedora Server installation result.
- Post-install validation result.
- 2 TB HDD SMART/journal caveat tracked in #16.
