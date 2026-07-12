# Forge Hardware Inventory

## Status

Installed state: Fedora Server installed and validated for story #10.

Related stories:

- [#9 Prepare Forge for Fedora installation](https://github.com/stfolder/home-platform/issues/9)
- [#10 Install and validate Fedora Server on Forge](https://github.com/stfolder/home-platform/issues/10)

Open follow-up:

- [#16 Investigate Forge 2 TB HDD SMART sector warnings](https://github.com/stfolder/home-platform/issues/16)

## Purpose and Role

Forge is the dedicated Fedora Server development workstation for the home engineering platform.

Primary responsibilities:

- Remote development through SSH and IDE tooling.
- Local development repositories.
- Docker Compose for local dependencies and temporary testing services.
- Java, Angular, TypeScript, Python, and shell-based development.
- AI coding agents launched from application repository roots.
- Optional future local AI experimentation.

Non-responsibilities:

- Production-like application hosting.
- Publicly exposed services.
- pfSense automation.
- KVM automation.
- Kubernetes deployment hosting during the initial Fedora installation phase.

## Hardware Inventory

| Component      | Current value            | Verification status               | Notes                                                                |
| -------------- | ------------------------ | --------------------------------- | -------------------------------------------------------------------- |
| Hostname       | `fedora`                 | Temporary                         | Temporary hostname for #10; stable identity is owned by #11.         |
| Local DNS name | `forge.home.arpa`        | Planned                           | To be configured after Fedora installation and LAN addressing story. |
| CPU            | Intel Core i7-9700K      | Confirmed                         | Confirmed during installation validation.                            |
| RAM            | 32 GB                    | Confirmed                         | Confirmed during installation validation.                            |
| GPU            | NVIDIA RTX 2080 Super    | Optional                          | Installation decision recorded below.                                |
| Network        | Wired Ethernet           | Verified                          | Wired Ethernet is set up for the installed Fedora system.            |
| Power          | UPS-backed               | Verified                          | Forge and pfSense are on UPS-backed power.                           |
| Target OS      | Fedora Server 44, x86_64 | Installed                         | Fedora Server installed on the Samsung 970 EVO Plus.                 |

## Disk Inventory and Disposition

Disk inventory after Fedora installation:

| Disk identifier      | Size   | Model or serial | Current contents                  | Disposition              | Notes |
| -------------------- | -----: | --------------- | --------------------------------- | ------------------------ | ----- |
| Samsung 970 EVO Plus | 512 GB | TBD             | Fedora Server 44                  | installed OS             | Fedora Server target disk. Reliable and fast. |
| Intel 660p           | 1 TB   | TBD             | Unknown; no important data expected | reuse later after checks | Separate SSD from the failing 2 TB HDD. It previously behaved inconsistently under Windows but normally under Linux. Verify identity and health before formatting or assigning platform storage. |
| HDD                  | 2 TB   | TBD             | Empty                             | investigate before use   | Identified as `/dev/sda`; SMART overall health reports `PASSED`, but journal reports 664 pending and 664 offline uncorrectable sectors. Investigation is tracked in #16. |

The Intel 660p and the 2 TB HDD are distinct devices. Issue #16 applies only to the 2 TB HDD that produced the critical SMART and journal warnings.

## Installed Partition Layout

Fedora Server is installed on the Samsung 970 EVO Plus using GPT partitioning and UEFI boot.

| Partition pattern | Mount point | Purpose |
|---|---|---|
| `nvme?n1p1` | `/boot/efi` | EFI system partition |
| `nvme?n1p2` | `/boot` | Boot partition |
| `nvme?n1p3` | `/` | Root filesystem |

These names record the observed partition pattern, not a stable disk identity. Confirm current device names with `lsblk`; use model, serial number, filesystem UUID, or PARTUUID for durable identification when those values are captured.

Allowed dispositions:

- `keep`: do not format, overwrite, repartition, or select during installation.
- `wipe`: approved destructive target before installation; all existing contents may be destroyed.
- `installed OS`: current operating-system disk after successful installation.
- `reuse later after checks`: preserve until identity and health checks pass; may then be repurposed.
- `investigate before use`: quarantine from platform storage until a documented concern is resolved.

## Target Installation Disk

Target installation disk: **Samsung 970 EVO Plus**

Fedora Server has been installed on this disk.

Validation evidence:

- Firmware storage list reviewed.
- Fedora installer disk list reviewed.
- Disk size and model matched the selected target.
- Fedora boots from the selected target disk without installation media.
- Non-target disks were not selected for the Fedora installation.

## Firmware Configuration

| Setting          | Required state                                        | Current state        | Notes                                                                        |
| ---------------- | ----------------------------------------------------- | -------------------- | ---------------------------------------------------------------------------- |
| Boot mode        | UEFI                                                  | Confirmed            | Legacy/CSM boot should not be used for the Fedora install.                   |
| VT-x             | Enabled                                               | Confirmed enabled    | Required for virtualization and local development workloads.                 |
| VT-d             | Enabled when supported                                | Confirmed enabled    | Useful for device isolation and future advanced virtualization.              |
| Secure Boot      | Disabled                                              | Decision recorded    | Secure Boot remains disabled for the initial Fedora installation.            |
| Boot order       | USB first for installation, target disk after install | Verified             | Fedora target disk is the normal boot device after installation.             |
| BIOS/UEFI update | Evaluate before install                               | Evaluated            | Update only if needed for stability, storage, network, or GPU compatibility. |

## Installation Decisions

### RTX 2080 Super

Decision: **defer installation until after base Fedora Server is stable**.

Rationale:

- Story #9 and Sprint 1 focus on bringing Forge online safely.
- GPU support is a later platform phase.
- Deferring reduces variables during first OS installation.
- Fedora can be validated first with CPU, RAM, storage, network, and UPS only.

### Secure Boot

Decision: **disabled**.

Rationale:

- Simplifies the initial Fedora Server installation.
- Avoids Secure Boot friction during future approved NVIDIA driver work.
- The initial risk is accepted temporarily within the physical home-network boundary.
- VPN-only remote access, SSH hardening, and firewall policy are established by later Platform Foundation stories and must not be treated as completed by story #10.

### Installation Media

Media used:

- Fedora Server x86_64 installation image.
- Fedora Server 44 was selected from the official Fedora Server source.
- Media creation and checksum verification were completed before installation.

## Planned Hardware Upgrades

| Upgrade               | Timing                      | Status      | Notes                                            |
| --------------------- | --------------------------- | ----------- | ------------------------------------------------ |
| NVIDIA RTX 2080 Super | After base server stability | Deferred    | Install during the later GPU and local AI phase. |
| Intel 660p reuse      | After health checks         | Deferred    | Verify identity and health before formatting.    |
| Additional storage    | After disk inventory        | Not planned | Add only when backup and disposition are clear.  |
| Additional RAM        | Future                      | Not planned | Current 32 GB is sufficient for Phase 1.         |

## Installation Readiness Checklist

### Hardware

- [x] Target installation disk is explicitly identified.
- [x] Existing disks have explicit safe dispositions.
- [x] RTX 2080 Super installation decision is recorded.
- [x] Wired Ethernet connectivity is verified.
- [x] UPS connection is verified.

### Firmware

- [x] UEFI mode is confirmed.
- [x] VT-x is enabled.
- [x] VT-d is enabled.
- [x] Secure Boot decision is documented.
- [x] Boot order is verified.
- [x] BIOS or UEFI update requirement is evaluated.

### Installation Media

- [x] Fedora Server installation media is created.
- [x] Downloaded image checksum is verified.

### Recovery

- [x] Important data is backed up.
- [x] Recovery or rollback approach is documented.
- [x] No uncertainty remains regarding which disk was formatted.

## Installed-State Checklist

- [x] Fedora Server is installed on the Samsung 970 EVO Plus.
- [x] GPT partitioning and UEFI boot are confirmed.
- [x] Installation media is removed.
- [x] Forge boots without user intervention.
- [x] Non-root administrative user exists.
- [x] Administrative user can run `sudo`.
- [x] Temporary hostname is assigned for #10.
- [x] Timezone and locale are configured.
- [x] Wired Ethernet is set up.
- [x] SELinux is enabled and enforcing.
- [x] firewalld is running.
- [x] Intel 660p is preserved for later reuse after checks.
- [x] Non-target 2 TB HDD SMART and journal warning is documented for follow-up in #16.

## Recovery and Rollback Summary

Before installation:

1. Back up any data from disks marked `keep`, `reuse later after checks`, or `investigate before use`.
2. Export or photograph firmware settings if changing boot mode, virtualization, Secure Boot, or boot order.
3. Confirm active project repositories are pushed to Git remotes.
4. Confirm the Samsung 970 EVO Plus is the only approved destructive installation target.
5. Prefer disconnecting non-target internal disks during installation if practical.

Rollback approach:

- If installation fails before disk formatting, power down and boot the previous system or recovery media.
- If installation fails after target disk formatting, reinstall Fedora Server on the same approved target disk.
- Do not attempt recovery writes to preserved or quarantined disks.
- Restore personal development data from Git remotes and documented backups after Fedora is stable.

## Change History

| Date       | Change | Author |
| ---------- | ------ | ------ |
| 2026-07-10 | Initial Forge hardware preparation document for story #9. | Codex |
| 2026-07-11 | Recorded Fedora Server installation validation state for story #10. | Codex |
| 2026-07-11 | Clarified that Intel 660p is a reusable SSD separate from the failing 2 TB HDD; corrected disk dispositions and security wording. | Nyx |