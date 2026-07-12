# Forge Hardware Preparation

## Status

Preparation status: in progress.

Story: [#9 Prepare Forge for Fedora installation](https://github.com/stfolder/home-platform/issues/9)

Forge must not be installed until every item in the installation readiness checklist is complete and the target installation disk is explicitly identified.

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
| Hostname       | `forge`                  | Planned                           | Canonical hostname from architecture.                                |
| Local DNS name | `forge.home.arpa`        | Planned                           | To be configured after Fedora installation and LAN addressing story. |
| CPU            | Intel Core i7-9700K      | Confirmed                         | Verify again in Fedora after installation.                           |
| RAM            | 32 GB                    | Known from platform specification | Verify in firmware before install.                                   |
| GPU            | NVIDIA RTX 2080 Super    | Optional                          | Installation decision recorded below.                                |
| Network        | Wired Ethernet preferred | Verified                          | Wired Ethernet connectivity is available for installation.           |
| Power          | UPS-backed               | Verified                          | Forge and pfSense are on UPS-backed power.                           |
| Target OS      | Fedora Server, x86_64    | Planned                           | Use official Fedora Server media.                                    |

## Disk Inventory and Disposition

Complete this table before booting Fedora installation media.

| Disk identifier      |  Size | Model or serial | Current contents                  | Disposition | Notes                                                                                                                                    |
| -------------------- | ----: | --------------- | --------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Samsung 970 EVO Plus | 512Gb | TBD             | Windows                           | wipe        | Identify from firmware and installer before formatting. Reliable and fast                                                                |
| ---                  |  ---: | ---             | --------------------------------- | ---         | ---                                                                                                                                      |
| Intel 660p           |   1Tb | TBD             | Don't remember. Nothing important | wipe        | Identify from firmware and installer before formatting. Glitched under windows, worked normally under linux. May be needs in diagnostics |
| HDD                  |   2Tb | TBD             | Empty                             | reuse later | Available for future storage after Fedora Server is installed and stable. Do not select as the Fedora installation target.               |

Allowed dispositions:

- `keep`: do not format, overwrite, repartition, or select during installation.
- `wipe`: approved Fedora installation target; all existing contents may be destroyed.
- `reuse later`: preserve for now; may be repurposed after Fedora is stable and backups are confirmed.

## Target Installation Disk

Target installation disk: **Samsung 970 EVO Plus**

The Fedora installer must not proceed past disk selection until this field is updated with the exact disk identifier, size, and model or serial number.

Required evidence:

- Firmware storage list reviewed.
- Fedora installer disk list reviewed.
- Disk size and model or serial match this document.
- All non-target disks are either disconnected or explicitly marked `keep`.

## Firmware Configuration

| Setting          | Required state                                        | Current state        | Notes                                                                        |
| ---------------- | ----------------------------------------------------- | -------------------- | ---------------------------------------------------------------------------- |
| Boot mode        | UEFI                                                  | Confirmed            | Legacy/CSM boot should not be used for the Fedora install.                   |
| VT-x             | Enabled                                               | Confirmed enabled    | Required for virtualization and local development workloads.                 |
| VT-d             | Enabled when supported                                | Confirmed enabled    | Useful for device isolation and future advanced virtualization.              |
| Secure Boot      | Disabled                                              | Decision recorded    | Secure Boot will remain disabled for the Fedora installation.                |
| Boot order       | USB first for installation, target disk after install | Verified             | Confirm again after installation so the Fedora target disk boots first.      |
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
- The system remains protected by VPN-only administrative access, SSH hardening, firewall controls, and physical network boundaries.

### Installation Media

Planned media:

- Fedora Server x86_64 installation image.
- Prefer the latest stable Fedora Server release available from the official Fedora Server download page on installation day.
- Record the selected Fedora Server version and checksum result before installation.

## Planned Hardware Upgrades

| Upgrade               | Timing                      | Status      | Notes                                            |
| --------------------- | --------------------------- | ----------- | ------------------------------------------------ |
| NVIDIA RTX 2080 Super | After base server stability | Deferred    | Install during the later GPU and local AI phase. |
| Additional storage    | After disk inventory        | Not planned | Add only when backup and disposition are clear.  |
| Additional RAM        | Future                      | Not planned | Current 32 GB is sufficient for Phase 1.         |

## Installation Readiness Checklist

### Hardware

- [x] Target installation disk is explicitly identified.
- [x] Existing disks are categorized as `keep`, `wipe`, or `reuse later`.
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
- [x] No uncertainty remains regarding which disk will be formatted.

## Recovery and Rollback Summary

Before installation:

1. Back up any data from disks marked `keep` or `reuse later`.
2. Export or photograph firmware settings if changing boot mode, virtualization, Secure Boot, or boot order.
3. Confirm active project repositories are pushed to Git remotes.
4. Confirm the target disk is the only disk marked `wipe`.
5. Prefer disconnecting non-target internal disks during installation if practical.

Rollback approach:

- If installation fails before disk formatting, power down and boot the previous system or recovery media.
- If installation fails after target disk formatting, reinstall Fedora Server on the same approved target disk.
- Do not attempt recovery writes to disks marked `keep`.
- Restore personal development data from Git remotes and documented backups after Fedora is stable.

## Change History

| Date       | Change                                                    | Author |
| ---------- | --------------------------------------------------------- | ------ |
| 2026-07-10 | Initial Forge hardware preparation document for story #9. | Codex  |
