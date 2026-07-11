# Fedora Server Installation Runbook

## Status

Runbook status: draft, pending physical pre-install verification.

Story: [#9 Prepare Forge for Fedora installation](https://github.com/stfolder/home-platform/issues/9)

This runbook prepares Forge for Fedora Server installation. Do not begin destructive installation steps until the readiness gates in this document and `docs/hardware/forge.md` are complete.

## References

- Fedora Server download page: https://fedoraproject.org/server/download/
- Fedora installation documentation: https://docs.fedoraproject.org/
- Forge hardware preparation: `docs/hardware/forge.md`
- Sprint reference: `docs/sprints/sprint-001.md`

## Installation Scope

Install Fedora Server on Forge as the home engineering platform development workstation.

Target:

- Architecture: x86_64.
- Hostname: `forge`.
- Local DNS name after later LAN configuration: `forge.home.arpa`.
- Installation target disk: must be copied from `docs/hardware/forge.md` after physical verification.

Out of scope for this runbook:

- Ansible provisioning.
- pfSense automation.
- KVM automation.
- NVIDIA driver installation.
- Kubernetes.
- Public service exposure.

## Pre-Install Stop Conditions

Stop and do not install Fedora if any of the following are true:

- The target installation disk is still `TBD`.
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

After reboot:

```bash
hostnamectl
cat /etc/fedora-release
ip addr
ip route
ping -c 4 1.1.1.1
ping -c 4 fedoraproject.org
systemctl --failed
lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
free -h
lscpu
```

Validation passes when:

- Forge boots from the Fedora target disk.
- Hostname is `forge`.
- Wired Ethernet has link and an IP address.
- Default route exists.
- DNS resolution works.
- No unexpected failed systemd services are present.
- Disk layout matches the selected target disk.
- Non-target disks were not formatted.
- CPU and RAM match expected inventory.

## Known Caveats

- Disk names such as `/dev/sda` and `/dev/nvme0n1` can change between boots. Prefer disk size, model, serial number, and physical port information when identifying the installation target.
- Fedora Server may list multiple installation profiles or package groups. Keep the initial installation minimal.
- Secure Boot should not be changed casually. Record the decision and reason.
- NVIDIA GPU driver work is deferred and should not be mixed into the base OS installation.
- Network interface names may not be predictable before installation. Stable LAN addressing is handled in story #11.

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

## Evidence to Capture

Before story #9 can move to review, capture the following in `docs/hardware/forge.md`:

- Target installation disk identifier, size, model, and serial when available.
- Disposition of every detected disk.
- Wired Ethernet verification.
- UPS verification.
- UEFI mode verification.
- VT-x verification and VT-d support evaluation.
- Secure Boot decision.
- Boot order verification.
- BIOS or UEFI update decision.
- Fedora media creation and checksum verification.
- Backup and recovery confirmation.
