# Forge Remote Development Runbook

## Status

LAN validation complete for story #15. Outside-LAN validation is deferred until #18 provides VPN-only access.

Related stories:

- [#14 Install core development toolchain](https://github.com/stfolder/home-platform/issues/14)
- [#15 Validate IntelliJ, VS Code, and iPad remote development workflows](https://github.com/stfolder/home-platform/issues/15)
- [#18 Enable VPN-only remote access](https://github.com/stfolder/home-platform/issues/18)

## Course Frame

Story #15 proves that Forge is not just a server sitting in the corner looking serious. It is the development engine room: Java, Docker, Git, database services, terminals, and builds run on Forge while the MacBook and iPad act as control surfaces.

The lab is intentionally checkpoint-based. Each checkpoint captures a useful operating fact, validates one workflow, and avoids sneaking credentials or machine caches into Git like a cursed side quest item.

## Architecture Position

Remote development uses the layers established by story #14:

1. **Forge host tooling:** Java 25, Maven, Git, Docker, Compose, Python, `uv`, Node.js through `nvm`, and shell tools.
2. **Repository-owned configuration:** build files, wrappers, lockfiles, `.nvmrc`, `.python-version`, `.devcontainer`, Compose definitions, and test fixtures.
3. **Client tools:** IntelliJ IDEA, VS Code, terminal clients, and iPad clients. Clients should connect to Forge; they should not duplicate the full toolchain locally.

Docker remains Unix-socket-only. No IDE convenience feature is allowed to turn the Docker daemon into a LAN piñata.

## Baseline Capture

### Part 1: MacBook Client And SSH Baseline

Status: PASS.

Captured from the MacBook before IDE-specific changes.

| Check | Result |
|---|---|
| Mac hostname | `Sergeys-MacBook-Pro.local` |
| User | `serge` |
| macOS | `15.6.1` / build `24G90` |
| Kernel | Darwin `24.6.0`, `arm64` |
| Local project checkout | `/Users/serge/Projects/home-platform` |
| Git branch | `main` |
| Git remote | `https://github.com/stfolder/home-platform.git` |
| Local working tree | clean at capture time |

SSH alias behavior:

| SSH setting | Result |
|---|---|
| Alias | `forge` |
| Hostname | `forge.home.arpa` |
| User | `serge` |
| Port | `22` |
| Identities only | `yes` |
| Agent forwarding | `no` |
| Public-key authentication | enabled |
| Password authentication | still available in local client config |
| Keyboard-interactive authentication | still available in local client config |
| Identity file | dedicated Forge SSH identity configured locally; private path intentionally not recorded in this public runbook |

SSH smoke test:

| Check | Result |
|---|---|
| Remote host | `forge` |
| Remote user | `serge` |
| Remote working directory | `/home/serge` |
| Remote time | `2026-07-13T17:02:02-07:00` |

Client tool baseline:

| Tool | Result |
|---|---|
| VS Code | `1.126.0`, `arm64` |
| VS Code Remote SSH | `ms-vscode-remote.remote-ssh@0.124.0` |
| VS Code Remote Containers / Dev Containers | `ms-vscode-remote.remote-containers@0.463.0` |
| VS Code Docker tooling | `docker.docker@0.18.0`, `ms-azuretools.vscode-docker@2.0.0`, `ms-azuretools.vscode-containers@2.4.5` |
| VS Code Java tooling | Extension Pack for Java, Java debugger, Maven, Gradle, test, dependency, upgrade, and Red Hat Java extensions present |
| VS Code Python tooling | Python, Pylance, debugpy, Black, flake8, pylint, Python envs present |
| VS Code infrastructure tooling | Terraform, YAML, Kubernetes, TOML tooling present |
| IntelliJ IDEA | installed in `/Applications` |
| JetBrains Toolbox / Fleet | Fleet visible under the user Applications tree |
| SSH agent | no identities loaded |

Notes:

- The Mac `date -Is` command failed because BSD/macOS `date` does not support GNU `date -I` syntax. Use `date "+%Y-%m-%dT%H:%M:%S%z"` on macOS when an ISO-like timestamp is needed.
- The local VS Code extension list is useful for baseline context, but the final runbook records only relevant families and versions rather than becoming a scroll of every cockpit sticker on the dashboard.
- Local SSH client config still allows password and keyboard-interactive authentication from the client side. The server-side Forge `sshd -T` result from story #12/#13 is the authority for what Forge actually accepts.

## Remaining Validation

- Outside-LAN MacBook workflow after #18 provides VPN-only access.
- Outside-LAN iPad workflow after #18 provides VPN-only access.
- Final story bookkeeping and issue update.

## IntelliJ Remote Development

### Part 4: Initial Connection Attempt

Status: PASS after Forge root filesystem expansion.

Initial IntelliJ Remote Development attempt:

| Check | Result |
|---|---|
| Client path | IntelliJ IDEA Remote Development flow |
| SSH target | `forge` |
| Project path | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| Result | failed after step 5 |
| Reported error | `There was an error in the connection provider` |

The JetBrains error was generic, so the first triage pass checked whether Forge itself looked compatible with a remote backend.

### Part 4A: Connection Provider Triage

Status: PASS for Forge-side prerequisites; client/backend failure still pending.

SSH sanity:

| Check | Result |
|---|---|
| Clean SSH stdout with `ssh forge 'true'` | `0` bytes |
| Simple SSH command | `jetbrains-ssh-ok` |

Forge platform:

| Check | Result |
|---|---|
| Host | `forge` |
| User | `serge` |
| Shell | `/bin/bash` |
| Home | `/home/serge` |
| OS | Fedora Linux 44 Server Edition |
| Kernel | `7.1.3-200.fc44.x86_64` |
| Architecture | `x86_64`, 64-bit |

Required remote basics:

| Tool | Result |
|---|---|
| `bash` | `/usr/bin/bash` |
| `sh` | `/usr/bin/sh` |
| `tar` | `/usr/bin/tar` |
| `gzip` | `/usr/bin/gzip` |
| `curl` | `/usr/bin/curl` |
| `wget` | `/usr/bin/wget` |
| `java` | `/usr/bin/java` |
| `git` | `/usr/bin/git` |

Storage and permissions:

| Check | Result |
|---|---|
| Root filesystem | 15G total, 9.3G used, 5.7G available |
| `/tmp` | 16G available |
| Home writable | yes |
| `/tmp` writable | yes |

JetBrains remote footprint:

| Path | Result |
|---|---|
| `/home/serge/.cache/JetBrains` | present |
| `/home/serge/.cache/JetBrains/RemoteDev` | present |
| `/home/serge/.cache/JetBrains/RemoteDev/dist` | present |
| `/home/serge/.cache/JetBrains/RemoteDev/remote-dev-worker` | present |
| `/home/serge/.config/JetBrains` | present |

No JetBrains remote-backend process was running after the failed attempt.

Conclusion: Forge satisfies the obvious SSH, OS, tool, disk, and permission prerequisites. The failure is now likely in the JetBrains client/backend bootstrap layer or in a specific remote-dev worker log. The next evidence should come from JetBrains client logs and remote JetBrains cache/log files.

### Part 4B: JetBrains Backend Disk Requirement

Status: BLOCKED.

JetBrains deployment logs identify the failure clearly:

| Check | Result |
|---|---|
| Backend target | `/home/serge/.cache/JetBrains/RemoteDev/dist/549d1c2fb22f6_idea-2026.1.4` |
| Backend archive URI | `https://download.jetbrains.com/idea/idea-2026.1.4.tar.gz` |
| Archive content length | `1,577,586,467` bytes |
| Available space in target path | `6,111,260,672` bytes |
| Required space | `6,310,345,868` bytes |
| Failure | `Not enough space to download IDE backend` |

The shortage is on Forge, not on the MacBook. JetBrains needs roughly 6.31 GB available in the Forge user cache path and found roughly 6.11 GB. The immediate shortfall is only about 190 MiB, but the practical fix should create several gigabytes of headroom because remote IDE backends, Maven caches, Docker images, and logs all like to expand like they discovered a cheat code.

Decision: free or expand Forge root/home storage before retrying IntelliJ Remote Development. Do not work around this by exposing services or moving credentials.

### Part 4C: Forge Storage Layout Investigation

Status: ROOT CAUSE FOUND.

Storage layout:

| Device / volume | Size | Role |
|---|---:|---|
| `/dev/nvme0n1` | 465.8G | Samsung SSD 970 EVO Plus 500GB; Fedora install disk |
| `/dev/nvme0n1p1` | 600M | EFI system partition mounted at `/boot/efi` |
| `/dev/nvme0n1p2` | 2G | XFS `/boot` |
| `/dev/nvme0n1p3` | 463.2G | LVM physical volume |
| `fedora/root` | 15G | XFS `/` |
| LVM free space in `fedora` VG | 448.17G | available but not allocated |
| `/dev/nvme1n1` | 953.9G | separate Intel NVMe with existing NTFS partitions |
| `/dev/sda` | 1.8T | HDD with existing NTFS partition; investigate before use per earlier disk health notes |

Root cause:

Forge does have the Fedora install disk available through LVM, but only 15G was allocated to the root filesystem. `/home/serge` is currently on `/`, so JetBrains Remote Development is trying to install its backend into a small root filesystem even though the volume group has hundreds of gigabytes free.

Recommended fix:

- Extend `fedora/root` using free space from the existing `fedora` volume group.
- Grow the XFS filesystem online.
- Re-check free space before retrying IntelliJ Remote Development.

This is not a hardware-capacity problem. It is an allocation problem: the mansion exists, but Forge is currently living in the broom closet.

### Part 4D: Root Filesystem Expansion

Status: PASS.

The root logical volume was expanded using Option A: allocate most of the Fedora NVMe to `/` while leaving a small LVM reserve.

| Check | Before | After |
|---|---:|---:|
| Root filesystem size | 15G | 450G |
| Root filesystem used | 9.3G during investigation | 18G after resize and current cache state |
| Root filesystem available | 5.7G during investigation | 433G |
| Root filesystem use | 62% | 4% |
| LVM free space | 448.17G | 13.17G |

Commands used:

```bash
sudo lvextend -L 450G /dev/fedora/root
sudo xfs_growfs /
```

JetBrains target path now has ample space:

| Path | Available |
|---|---:|
| `/home/serge/.cache/JetBrains/RemoteDev/dist` | 433G |

A failed partial backend directory remains and should be removed before retrying IntelliJ:

```text
/home/serge/.cache/JetBrains/RemoteDev/dist/549d1c2fb22f6_idea-2026.1.4
```

The failed partial backend directory was removed and IntelliJ Remote Development was retried successfully.

### Part 4E: IntelliJ Java Workflow

Status: PASS.

Client and backend:

| Check | Result |
|---|---|
| IntelliJ version | `2026.1.4` |
| Remote backend path | `/home/serge/.cache/JetBrains/RemoteDev/dist/549d1c2fb22f6_idea-2026.1.4` |
| Project path | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| Backend process | `remote-dev-server` running on Forge |
| JetBrains daemon | `jetbrainsd` running on Forge |
| Maven remote server | running under the JetBrains backend |
| Java build process | running under the JetBrains backend during validation |

IDE validation:

| Check | Result |
|---|---|
| Project opened remotely | PASS |
| Indexing | completed |
| Project JDK | Java 25 selected |
| Code navigation | PASS |
| Unit test from IntelliJ | PASS |
| Run `App.main` from IntelliJ | PASS |
| Breakpoint in `MissionBriefing.summary()` | PASS |
| Debug `App.main` | breakpoint hit as expected |

Terminal confirmation while IntelliJ was open:

| Check | Result |
|---|---|
| Git status | clean |
| Java | OpenJDK `25.0.3` |
| javac | `25.0.3` |
| Maven | `3.9.11`, using Java `25.0.3` |
| `mvn test -q` | exit status `0` |

Conclusion: IntelliJ Remote Development can open, index, build, test, run, and debug the representative Java 25 project on Forge. Java selection remains aligned with the Fedora-owned Java 25 host default.

### Part 5: IntelliJ Database Tools

Status: PASS.

IntelliJ database validation:

| Check | Result |
|---|---|
| PostgreSQL data source | configured against `127.0.0.1:55432` |
| Database | `forge_remote_dev` |
| User | `forge_dev` |
| Connection test | PASS |
| Schema/table browsing | PASS |
| SQL query | PASS |
| Query result | `1, Forge database link online. Keep the shiny side up.` |

Forge-side confirmation:

| Check | Result |
|---|---|
| Compose service | `remote-dev-validation-postgres-1` |
| Health | `healthy` after 58 minutes |
| Port mapping | `127.0.0.1:55432->5432/tcp` |
| Socket bind | `LISTEN 127.0.0.1:55432` |
| Container-side row count | `mission_log_rows = 1` |

Conclusion: IntelliJ can query the Compose-hosted PostgreSQL service through the approved private development path. The database is not exposed on a LAN interface.

The final Git status line was not captured in the database checkpoint output, but the next VS Code baseline opened the same repository and showed no pending changes before the VS Code edit.

## VS Code Remote SSH

### Part 6: Native Remote SSH Workflow

Status: PASS.

VS Code connection:

| Check | Result |
|---|---|
| Remote SSH connection to `forge` | PASS |
| Folder opened | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| Java language tooling | alive |
| Dev Container prompt | offered by VS Code; intentionally deferred to the Dev Container checkpoint |

VS Code integrated terminal baseline:

| Check | Result |
|---|---|
| Terminal host | `forge` |
| Terminal working directory | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| Java | OpenJDK `25.0.3` |
| Maven | `3.9.11`, using Java `25.0.3` |
| Compose service | PostgreSQL still `healthy` |
| PostgreSQL port | `127.0.0.1:55432->5432/tcp` |

Bounded edit:

| Check | Result |
|---|---|
| Edited file | `src/main/java/dev/forge/remote/MissionBriefing.java` |
| Change | added `VS Code thrusters: armed.` to the summary string |
| Diff review | PASS |
| `mvn test` | PASS |
| Test result | `Tests run: 2, Failures: 0, Errors: 0, Skipped: 0` |
| App run | printed updated summary |
| Local commit | `f57deef Validate VS Code remote edit workflow` |
| Git status after commit | clean |

Conclusion: VS Code Remote SSH can open the Forge-hosted repository, run its terminal on Forge, use Java tooling, inspect Docker Compose state, edit source, review diffs, run Maven tests, run the application, and commit locally without copying the toolchain to the MacBook.

## Docker Dev Container

### Part 7: Initial Dev Container Attempt

Status: NEEDS REPOSITORY CONFIG FIX.

VS Code detected the repository-owned `.devcontainer/devcontainer.json` and offered to reopen the project in a Dev Container. The offer was deferred during the native VS Code Remote SSH checkpoint, then accepted for the Dev Container checkpoint.

Container identity:

| Check | Result |
|---|---|
| Container hostname | `472b7ecf9d57` |
| User | `vscode` |
| Working directory | `/workspaces/remote-dev-validation` |
| UID/GID | `1000:1000` |
| Groups | `vscode`, `nvm`, `sdkman` |

Container tool validation:

| Tool | Result |
|---|---|
| Java | OpenJDK `25.0.3` LTS, Microsoft build |
| javac | `25.0.3` |
| Maven | missing: `bash: mvn: command not found` |

Project visibility:

| Check | Result |
|---|---|
| Source tree mounted | PASS |
| Git metadata visible | PASS |
| IntelliJ `.idea` metadata present | ignored by `.gitignore`; not committed |
| Existing Maven target output visible | PASS |

Build/test:

| Check | Result |
|---|---|
| `mvn test` inside container | FAIL because Maven is not installed |

Host ownership after leaving container:

| Check | Result |
|---|---|
| Source file owner from container | `1000:1000` |
| Source file owner from host | `1000:1000` |
| Host Git status | clean |

Docker resources:

| Resource | Result |
|---|---|
| PostgreSQL Compose container | still healthy |
| Dev Container image | `vsc-remote-dev-validation-...`, about `2.25GB` |
| Dev Container volume | `vscode` local volume |

Conclusion: the Dev Container workflow can open the repository and preserve file ownership, but the repository-owned configuration is incomplete because it does not provide Maven. This is exactly why the Dev Container contract belongs in the repository: the container should carry its own build tools instead of hoping the base image is feeling generous.

### Part 7B: Dev Container Maven Fix And Rebuild

Status: PASS.

The repository-owned Dev Container configuration was updated to add Maven through the Dev Containers Java feature, then the container was rebuilt.

Rebuilt container tools:

| Tool | Result |
|---|---|
| Java | OpenJDK `25.0.3` LTS, Microsoft build |
| javac | `25.0.3` |
| Maven | Apache Maven `3.9.16` |
| Maven home | `/usr/local/sdkman/candidates/maven/current` |
| Maven Java runtime | Microsoft OpenJDK `25.0.3` |

Build/test inside rebuilt container:

| Check | Result |
|---|---|
| `mvn test` | PASS |
| Test result | `Tests run: 2, Failures: 0, Errors: 0, Skipped: 0` |
| Build result | `BUILD SUCCESS` |

Ownership check:

| Check | Result |
|---|---|
| Source file owner inside rebuilt container | `1000:1000` |

Conclusion: the Dev Container can now build and test the repository using repository-owned configuration. The build tools are inside the container instead of depending on undocumented host customization, which is the whole point of putting this thing in a container rather than just painting racing stripes on native Forge.

Host-side tail checks:

| Check | Result |
|---|---|
| Dev Container config fix commit | `191c662 Add Maven to dev container validation` |
| Git status after rebuild | untracked `.devcontainer/devcontainer-lock.json` |
| Source file ownership from host | `1000:1000` |
| `.devcontainer/devcontainer.json` ownership | `1000:1000` |
| `.devcontainer/devcontainer-lock.json` ownership | `1000:1000` |
| PostgreSQL Compose container | still healthy |
| Dev Container images | rebuilt feature images present, about `2.27GB`; previous image still present, about `2.25GB` |
| Dev Container volume | `vscode` local volume present |

The generated lockfile is not a secret. It was committed as part of the repository-owned Dev Container contract because it records the resolved Dev Container feature state.

Lockfile commit:

| Check | Result |
|---|---|
| Lockfile | `.devcontainer/devcontainer-lock.json` |
| Feature version | `ghcr.io/devcontainers/features/java:1`, resolved version `1.8.0` |
| Feature digest | `sha256:9663ce0219ff85786e87901ce5f0a59f488edd5f99b46015192cda48468b233a` |
| Commit | `e1d8a44 Record dev container feature lockfile` |
| Git status after commit | clean |

## iPad Thin-Client Workflow

### Part 8: SSH And tmux Java Workflow

Status: PASS.

Client decision:

| Check | Result |
|---|---|
| Primary iPad workflow | SSH terminal plus `tmux` |
| iPad SSH client | validated by successful SSH connection |
| Keyboard/trackpad usability | acceptable |
| Local iPad Java/Docker/Maven install | not required |
| Persistent terminal | `tmux` detach/reconnect validated |

Workflow evidence:

| Check | Result |
|---|---|
| Repository path | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| Latest commit visible | `e1d8a44 Record dev container feature lockfile` |
| Java action | `mvn test -q` |
| Maven test status | `0` |
| App run | `mvn -q exec:java -Dexec.mainClass=dev.forge.remote.App` |
| App output | `Forge remote development is online. Debugger helmet: polished. VS Code thrusters: armed.` |
| tmux marker | `/tmp/story15-ipad-marker.txt` |
| Marker timestamp | `2026-07-13T22:15:35-07:00` |

Limitations:

- The iPad workflow is useful for terminal-first Java actions: inspect, edit with terminal editor if desired, run Maven, review Git, keep long-running sessions alive with `tmux`.
- It is not parity with IntelliJ Remote Development. Debugging, refactoring, and database browsing remain better on the MacBook IDEs.
- No browser IDE, public endpoint, Docker TCP listener, or database exposure was introduced for iPad convenience.

Conclusion: the iPad can perform useful Java development actions as a thin client while Forge owns the toolchain. The iPad gets to be the command deck, not the engine room.

## Reconnect And Reboot Validation

### Part 9: Pre-Reboot Snapshot

Status: PASS with note.

Repository state:

| Check | Result |
|---|---|
| Latest commit | `e1d8a44 Record dev container feature lockfile` |
| Recent history | `191c662`, `f57deef`, `65dea04` also present |
| Git status | clean; no status output before log |

Compose and Docker:

| Check | Result |
|---|---|
| PostgreSQL Compose service | `remote-dev-validation-postgres-1` healthy for about 5 hours |
| PostgreSQL port | `127.0.0.1:55432->5432/tcp` |
| Dev Container images | present |
| PostgreSQL image | present |
| Dev Container volume | `vscode` local volume present |

Note: a container named `goofy_kilby` was running before reboot. This is not part of the story #15 validation set and should be identified or cleaned up before final story wrap-up.

Service health:

| Check | Result |
|---|---|
| `firewalld`, `sshd`, `cockpit.socket`, `docker`, `dnf5-automatic.timer` | active |
| Failed units | `fwupd-refresh.service` failed before reboot |

The `fwupd-refresh.service` failure cleared after reboot and did not block the remote-development validation.

### Part 9B: Post-Reboot Validation

Status: PASS.

Forge service and tool state:

| Check | Result |
|---|---|
| Failed systemd units | none |
| `firewalld`, `sshd`, `cockpit.socket`, `docker`, `dnf5-automatic.timer` | active and enabled |
| Java | OpenJDK `25.0.3` |
| Maven | `3.9.11`, using Java `25.0.3` |
| Git | `2.55.0` |
| Docker | `29.6.1` |
| Docker Compose | `v5.3.1` |

Repository and Java action:

| Check | Result |
|---|---|
| Latest commit after reboot | `e1d8a44 Record dev container feature lockfile` |
| Compose state after reboot | no project containers running |
| `mvn test -q` on Forge | PASS, exit status `0` |

Project Compose services are not configured to auto-start, and that is acceptable. They are development services owned by the repository and should be started deliberately when needed.

Client reconnect:

| Client | Result |
|---|---|
| IntelliJ Remote Development | reconnected successfully |
| VS Code Remote SSH | reconnected successfully |
| iPad SSH | reconnected successfully |

iPad post-reboot validation:

| Check | Result |
|---|---|
| Previous tmux session | not present after reboot |
| Repository path | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| `mvn test -q` from iPad SSH | PASS |
| Exit status | `ipad-post-reboot-mvn-status=0` |

The tmux session did not survive reboot, which is expected for a normal user tmux session under `/tmp/tmux-1000`. The iPad workflow still passes because the iPad can reconnect and run the Java action after reboot. If persistent sessions across reboot become important, that should be a separate systemd user service or terminal-persistence story, not an accidental requirement hidden in #15.

### Part 9C: Post-Reboot Exposure And Cleanup Review

Status: PASS.

Listening sockets after reconnecting IDE clients:

| Socket | Process | Result |
|---|---|---|
| TCP `:22` | `sshd` | expected LAN-scoped SSH listener |
| TCP `:9090` | systemd/Cockpit | expected LAN-scoped Cockpit listener |
| TCP/UDP loopback `:53` | `systemd-resolved` | expected local resolver |
| UDP loopback `:323` | `chronyd` | expected local time socket |
| TCP `127.0.0.1:55432` | `docker-proxy` | expected PostgreSQL Compose loopback bind |
| TCP `127.0.0.1:*` and `[::ffff:127.0.0.1]:*` | VS Code / IntelliJ remote backend processes | expected local-only remote IDE listeners |

Firewall state:

| Check | Result |
|---|---|
| Runtime zone | `FedoraServer`, active on `eno2` |
| Runtime services/ports | none |
| Runtime forwarding | `no` |
| Runtime rich rules | LAN-scoped SSH and Cockpit only |
| Permanent services/ports | none |
| Permanent forwarding | `no` |
| Permanent rich rules | LAN-scoped SSH and Cockpit only |

Docker cleanup review:

| Resource | Result |
|---|---|
| PostgreSQL container | `remote-dev-validation-postgres-1`, healthy, loopback-only port |
| Exited Dev Container helper container | `goofy_kilby`, exited `0`; image name matches VS Code Dev Container feature image |
| Dev Container images | feature/current images present, about `2.25GB` to `2.27GB` each |
| PostgreSQL image | present and in use |
| `hello-world` image | leftover from smoke checks |
| Volumes | one hashed local volume and `vscode` local volume |
| Networks | Docker defaults plus `remote-dev-validation_default` |

Conclusion: no new LAN exposure was introduced. Remaining Docker resources are expected validation artifacts or cleanup candidates. The exited `goofy_kilby` container is not a mystery workload; it is a Dev Container build/helper artifact with Docker's auto-generated name.

Cleanup should stop/remove the PostgreSQL service when database validation is complete, remove the exited Dev Container helper container, and remove disposable smoke images. Dev Container images and the `vscode` volume may be retained if fast reconnect is desired, or removed if the goal is a pristine cleanup.

### Part 9D: Final Docker Cleanup

Status: PASS.

Cleanup actions:

| Action | Result |
|---|---|
| `docker compose down --volumes --remove-orphans` | removed `remote-dev-validation-postgres-1` and `remote-dev-validation_default` |
| Remove exited Dev Container helper | removed `goofy_kilby` |
| Remove `hello-world:latest` | removed |

Final Docker state:

| Resource | Result |
|---|---|
| Containers | none |
| Networks | only Docker defaults: `bridge`, `host`, `none` |
| Volumes | retained `vscode` local volume |
| Images retained | `postgres:latest` and Dev Container images |

The retained images and `vscode` volume are intentional for quick follow-up validation. They can be pruned later if a pristine Docker state is preferred.

## Forge Regression Checks

### Part 2: Forge Toolchain Regression

Status: PASS.

The first all-in-one SSH command stopped after the user-manager load banner, so the check was rerun through `ssh forge 'bash -s' <<'EOF' ... EOF`. That form is preferred for longer remote lab scripts because quoting stays sane and the command does not become a suspiciously overloaded backpack.

Manager load diagnostics:

| Check | Result |
|---|---|
| Remote shell | `/bin/bash` |
| Bash version | `5.3.9(1)-release` |
| `nvm` load | succeeded |
| `nvm` version | `0.40.5` |
| SDKMAN load | succeeded through a 10-second timeout wrapper |
| SDKMAN version | script `5.23.0`, native `0.7.34` |
| `uv` path | loaded through `$HOME/.local/bin` |
| `uv` version | `0.11.28` |

Core tool versions:

| Tool | Result |
|---|---|
| Git | `2.55.0` |
| Java | OpenJDK `25.0.3` |
| javac | `25.0.3` |
| Maven | `3.9.11` |
| Node.js | `v24.18.0` |
| npm | `11.16.0` |
| Python | `3.14.6` |
| Docker | `29.6.1` |
| Docker Compose | `v5.3.1` |

Service health had already been captured immediately before the manager diagnostics:

| Check | Result |
|---|---|
| Failed systemd units | none |
| `firewalld` | active and enabled |
| `sshd` | active and enabled |
| `cockpit.socket` | active and enabled |
| `docker` | active and enabled |
| `dnf5-automatic.timer` | active and enabled |

Conclusion: the story #14 baseline is still alive and suitable for story #15 remote development validation.

### Part 2B: Docker Smoke And Exposure Check

Status: PASS.

Docker non-root smoke:

| Check | Result |
|---|---|
| `docker run --rm hello-world` | PASS |
| Image behavior | `hello-world:latest` was re-pulled because story #14 cleanup removed smoke images |
| Runtime result | container ran and exited successfully |

The first remote sudo attempt used a heredoc with `ssh -t`, but SSH did not allocate a pseudo-terminal because stdin was not a terminal. As a result, `sudo` could not read the password. The checks were rerun from an interactive SSH session.

Listening sockets:

| Socket | Process | Result |
|---|---|---|
| TCP `:22` | `sshd` | expected SSH listener |
| TCP `:9090` | systemd/Cockpit socket | expected Cockpit listener |
| TCP/UDP loopback `:53` | `systemd-resolved` | expected local resolver |
| UDP loopback `:323` | `chronyd` | expected local time socket |

Docker exposure:

| Check | Result |
|---|---|
| Docker TCP listener search | no output; no Docker TCP listener present |

Firewall state:

| Check | Runtime | Permanent |
|---|---|---|
| Zone | `FedoraServer` | `FedoraServer` |
| Interface | `eno2` | none listed in permanent view |
| Services | none | none |
| Ports | none | none |
| Forward | `no` | `no` |
| Rich rule: LAN SSH | present for `10.42.42.0/24` | present for `10.42.42.0/24` |
| Rich rule: LAN Cockpit | present for `10.42.42.0/24` | present for `10.42.42.0/24` |

Conclusion: Forge remains aligned with the #13/#14 network baseline. The IDE story can proceed without opening any surprise ports or giving Docker a megaphone.

## Representative Validation Repository

### Part 3: Repository Creation

Status: PASS.

Repository selected for story #15:

| Check | Result |
|---|---|
| Location | `/home/serge/forge-labs/story-15/remote-dev-validation` |
| Ownership | disposable Forge-hosted validation repository for story #15 |
| Git repository | initialized locally |
| Initial branch | `master` from Git default; renamed to `main` in Part 3B |
| Initial commit | `65dea04 Create remote development validation project` |

The repository is intentionally small but not useless. It contains enough structure to make IntelliJ, VS Code, Docker, Compose, Java, Maven, PostgreSQL, Git, and later iPad workflows do actual work instead of posing for a brochure.

Repository contents:

| Path | Purpose |
|---|---|
| `pom.xml` | Java 25 Maven project with JUnit 5 and Surefire |
| `src/main/java/dev/forge/remote/App.java` | runnable application entry point |
| `src/main/java/dev/forge/remote/MissionBriefing.java` | source class for edit, navigation, completion, refactor, and debug validation |
| `src/test/java/dev/forge/remote/MissionBriefingTest.java` | unit tests for build/test validation |
| `compose.yaml` | Docker Compose PostgreSQL service |
| `db/init/001_schema.sql` | disposable schema and seed data |
| `.devcontainer/devcontainer.json` | repository-owned Dev Container definition |
| `.gitignore` | excludes IDE/project build noise |
| `README.md` | validation-project notes |

Maven validation:

| Check | Result |
|---|---|
| Java release | compiled with `javac` release `25` |
| Test command | `mvn test` |
| Test result | `Tests run: 2, Failures: 0, Errors: 0, Skipped: 0` |
| Build result | `BUILD SUCCESS` |
| Runtime command | `mvn -q exec:java -Dexec.mainClass=dev.forge.remote.App` |
| Runtime output | `Forge remote development is online. Debugger helmet: polished.` |

Maven emitted the same Guice / `sun.misc.Unsafe` deprecation warning previously seen in story #14. The build and runtime succeeded, so this remains upstream Maven dependency noise rather than a Forge configuration failure.

Compose service boundary:

| Check | Result |
|---|---|
| PostgreSQL image | `postgres:latest` |
| Host bind | `127.0.0.1:55432:5432` |
| Database | `forge_remote_dev` |
| Disposable user | `forge_dev` |
| Seed table | `mission_log` |

The PostgreSQL port is deliberately bound to Forge loopback, not all LAN interfaces. The database should be reached through approved private paths such as an SSH tunnel or remote IDE backend, not by shouting SQL across the network like a reckless spaceport announcer.

### Part 3B: Branch Normalization And PostgreSQL Compose Check

Status: PASS.

Repository normalization:

| Check | Result |
|---|---|
| Branch rename | `master` renamed to `main` |
| Current branch | `main` |
| Latest commit | `65dea04 Create remote development validation project` |
| Expected source files | present under `src`, `db`, and `.devcontainer` |

PostgreSQL Compose validation:

| Check | Result |
|---|---|
| `docker compose up -d postgres` | PASS |
| Image behavior | `postgres:latest` was pulled because story #14 cleanup removed smoke images |
| Container | `remote-dev-validation-postgres-1` |
| Health | reached `healthy` |
| Compose port display | `127.0.0.1:55432->5432/tcp` |
| SQL query | PASS |
| Query result | `Forge database link online. Keep the shiny side up.` |

The Compose status shows the intended loopback-only host bind. This is exactly the kind of database exposure we want for an IDE validation lab: useful to the developer, boring to the LAN, invisible to anyone hoping the database left the garage door open.

Tail checks:

| Check | Result |
|---|---|
| Host bind | `LISTEN 127.0.0.1:55432`; loopback-only |
| Git status after Compose start | clean |
| Compose status after tail check | still `healthy`, still `127.0.0.1:55432->5432/tcp` |
