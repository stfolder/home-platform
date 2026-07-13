# Forge Development Toolchain Runbook

## Status

Review-ready for story #14.

Related stories:

- [#13 Configure host firewall and update policy](https://github.com/stfolder/home-platform/issues/13)
- [#14 Install core development toolchain](https://github.com/stfolder/home-platform/issues/14)
- [#15 Validate VS Code Remote SSH workflow](https://github.com/stfolder/home-platform/issues/15)

## Course Frame

Forge is being promoted from secure Fedora server to development workbench. The shape of the system is deliberately layered:

1. **Host-owned baseline:** shell tools, Git, diagnostics, Java host default, Node.js LTS, Python baseline, Docker Engine, and Docker Compose.
2. **Repository-owned development environment:** wrappers, lockfiles, project-local Python environments, local TypeScript, toolchains, and optional dev containers.
3. **Compose-owned project services:** PostgreSQL, Redis, Kafka, LocalStack, Keycloak, and similar dependencies.

The host should feel like a well-tuned sport bike: fast, direct, and not held together by mystery zip ties from a previous owner.

Tone for this runbook: useful first, fun second. The commands should be boringly reliable; the commentary can have a little Mass Effect engineering bay, garage wrenching, and RPG inventory-screen energy. If a joke ever makes the command less clear, the joke loses initiative.

## Baseline Capture

Captured on Forge during story #14.

### Host

| Check | Value |
|---|---|
| Hostname | `forge` |
| User | `serge` |
| Working directory | `/home/serge/forge-smoke/story-14` |
| Capture time | `2026-07-13T11:09:04-07:00` |
| Failed systemd units | none |

### Existing Tools Before Story #14 Changes

| Tool | Baseline result | Initial owner / notes |
|---|---|---|
| Git | missing | Install as host-owned Fedora package. |
| curl | `8.18.0` | Fedora package. |
| wget | command exists as Wget2, but `wget` RPM is not installed | Review package ownership during common tools install. |
| jq | `1.8.1` | Fedora package. |
| ripgrep | missing | Install as host-owned Fedora package. |
| fd | missing | Install as host-owned Fedora package. |
| Neovim | missing | Install as host-owned Fedora package. |
| tmux | missing | Install as host-owned Fedora package. |
| Java | missing | Install Java 25 LTS host default if available. |
| javac | missing | Installed with Java development package. |
| Maven | missing | Install Maven or validate Maven Wrapper ownership. |
| Node.js | missing | Install one supported LTS baseline. |
| npm | missing | Installed with Node.js strategy. |
| Corepack | missing | Validate or explicitly defer after Node install. |
| Python | `3.14.6` | Fedora system Python; do not use for global project dependencies. |
| uv | missing | Install for project-local Python environments. |
| Docker Engine | missing | Install Docker Engine and Compose plugin from documented Docker source. |
| Docker Compose | missing | Install as Docker Compose plugin. |
| Podman | `5.8.4` | Present from Fedora; not selected as the baseline runtime for #14. |
| dnf automatic updates | `dnf5-plugin-automatic-5.4.2.1-1.fc44` | Already configured by #13. |

### User And Security Baseline

| Check | Value |
|---|---|
| User groups | `serge wheel` |
| Docker group membership | absent before Docker installation |
| firewalld zone | `FedoraServer` |
| firewalld interface | `eno2` |
| firewalld services | none |
| firewalld forwarding | `no` |
| allowed inbound admin services | LAN-scoped SSH and Cockpit rich rules only |

### Listening Sockets Before Toolchain Installation

| Socket | Process | Notes |
|---|---|---|
| TCP `:22` | `sshd` | Approved by #13, LAN-scoped in firewalld. |
| TCP `:9090` | systemd socket for Cockpit | Approved by #13, LAN-scoped in firewalld. |
| TCP/UDP loopback `:53` | `systemd-resolved` | Local resolver stub only. |
| UDP loopback `:323` | `chronyd` | Local time service socket only. |

No LLMNR listener on `:5355` was present. This confirms the #13 baseline survived into story #14.

## Implementation Notes

- Docker is selected over Podman for the story #14 baseline because it provides the lowest-friction compatibility with Docker Compose, Dev Containers, IntelliJ, VS Code Remote SSH, common project repositories, and future browser or iPad workflows.
- Existing Podman is recorded, but #14 does not create a dual-runtime development model.
- Docker group membership must be documented as root-equivalent capability.
- Framework CLIs such as Angular CLI and TypeScript must remain repository-local unless a later story approves a global install.
- PostgreSQL and similar services must run through Docker Compose for smoke tests; they must not be installed permanently on Forge in this story.

## Update Policy

Fedora-owned and vendor-repository-owned tools are updated through `dnf` and the security-update policy documented in [Forge Host Baseline Runbook](host-baseline.md).

| Tool category | Update owner | Policy |
|---|---|---|
| Fedora packages | Fedora / `dnf` | Security updates apply through `dnf5-automatic`; broader updates are reviewed manually. |
| Docker Engine and Compose | Docker CE repository / `dnf` | Updated through `dnf` from the documented Docker repository. |
| `nvm` | user-level tool manager | Review manually; do not auto-update unattended. |
| SDKMAN | user-level tool manager | Review manually; do not auto-update unattended. |
| `uv` | user-level tool manager | Review manually; do not auto-update unattended. |
| Project Node.js versions | repository metadata | Declare with `.nvmrc` or package metadata. |
| Project Java versions | repository metadata | Declare with Maven/Gradle toolchains, `.sdkmanrc`, wrappers, or containers. |
| Project Python versions | repository metadata | Declare with `uv` metadata such as `.python-version` and `pyproject.toml`. |

Review user-level tool-manager updates monthly, or before starting a major project/toolchain upgrade. Record meaningful upgrades in this runbook or in future user-bootstrap automation.

Check `nvm`:

```bash
nvm --version
cd ~/.nvm
git fetch --tags
git tag --sort=-v:refname | head
```

Update `nvm` only after choosing the target release. Example using the currently installed release:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.5/install.sh | bash
```

Check and update SDKMAN:

```bash
sdk version
sdk selfupdate
```

Check and update `uv`:

```bash
uv --version
uv self update
```

Do not configure unattended updates for `nvm`, SDKMAN, or `uv`. These tools affect interactive project environments and should not shapeshift overnight like a suspicious sci-fi artifact found under a tarp in the garage.

## Results

Validation results are recorded incrementally as each lab checkpoint is completed.

### Part 2: Common Host Tools

Status: PASS.

This step installed the starter kit: Git for time travel, ripgrep for detective work, tmux for cockpit panels, Neovim for “I swear I only need to edit one line,” and native compilers for when the quest giver says “just build it from source.” Somewhere, a sci-fi mechanic in oil-stained boots nodded approvingly.

Installed through Fedora packages:

| Tool group | Validated versions / notes |
|---|---|
| Git | `git version 2.55.0`; disposable local repository commit succeeded. |
| curl | `curl 8.18.0`; already present before install. |
| wget command | `GNU Wget2 2.2.1`; command is provided by Fedora's `wget2` / `wget2-wget` packaging rather than a package named exactly `wget`. |
| jq | `jq-1.8.1`; already present before install. |
| ripgrep | `ripgrep 14.1.1`. |
| fd | `fd 10.4.2`. |
| tmux | `tmux 3.7b`. |
| Neovim | `NVIM v0.12.4`. |
| native build tools | `gcc 16.1.1`, `g++ 16.1.1`, `GNU Make 4.4.1`, `openssl-devel`. |
| tree | `tree v2.2.1`; already present before install. |
| Node.js side effect | `nodejs22-22.22.2`, `nodejs22-npm-10.9.7`; pulled in by Fedora package dependencies during common tool installation. |

The common-tools install pulled in Node.js 22 packages as weak dependencies through the Fedora package graph. The ship's AI woke up early, but it does not get a command chair yet: the Node checkpoint still owns the final Node/npm/Corepack decision and repository-local TypeScript smoke test.

Follow-up ownership checks:

| Package | Result |
|---|---|
| `wget` | not installed |
| `wget2` | `2.2.1-2.fc44` |
| `wget2-wget` | `2.2.1-2.fc44` |
| `nodejs22` | `22.22.2-3.fc44` |
| `nodejs22-npm` | `10.9.7-1.22.22.2.3.fc44` |
| `nodejs` / `npm` package names | not installed as generic package names |

Runtime follow-up:

| Command | Result |
|---|---|
| `node --version` | `v22.22.2` |
| `npm --version` | `10.9.7` |
| `corepack --version` | missing |

The Git smoke test used a disposable repository under `/home/serge/forge-smoke/story-14`, committed one file, and removed the repository afterward. Starter kit acquired: sword, medpack, flashlight, and a tasteful amount of sci-fi engineering tape.

### Part 3: Docker Engine And Compose

Status: PASS.

Docker is installed from Docker's Fedora repository, not from Fedora's Podman stack and not from an ad hoc shell script. This is the cargo bay: powerful, practical, and absolutely not something to leave open to space pirates.

| Component | Version / evidence | Owner |
|---|---|---|
| Docker Engine | `29.6.1` | Docker CE repository |
| Docker CLI | `29.6.1` | Docker CE repository |
| Docker Compose plugin | `v5.3.1` | Docker CE repository |
| containerd | `v2.2.6` | Docker CE repository |
| runc | `1.3.6` | Docker CE dependency |
| Buildx plugin | `0.35.0` | Docker CE repository |

Validated:

- `docker.service` is enabled and active.
- `sudo docker run --rm hello-world` succeeded.
- After a new SSH session, `serge` belongs to the `docker` group.
- Non-root `docker version`, `docker compose version`, and `docker run --rm hello-world` succeeded.
- `sudo ss -lntup | grep -i docker` returned no TCP listener.

Security note: membership in the `docker` group is effectively root-equivalent on Forge. Treat it like the key to a superbike: convenient, thrilling, and not something to hand to a random NPC who says they are "just going around the block."

### Part 4: Docker Compose Smoke Test

Status: PASS.

A disposable Compose stack under `/home/serge/forge-smoke/story-14/compose-hello` pulled `alpine:latest`, created a temporary network, ran one service, printed `compose-ok`, and exited with code `0`.

Cleanup:

- `docker compose down --volumes --remove-orphans` removed the container and network.
- `docker ps -a --filter "name=compose-hello"` returned no remaining containers.
- The temporary workspace was removed.

Compose is validated for the basic "summon companion, complete quest, dismiss companion" workflow. No camp clutter remained.

### Part 5: Java 25 Availability

Status: PASS.

Fedora 44 offers Java 25 directly:

| Package | Version | Repository | Decision |
|---|---|---|---|
| `java-25-openjdk` | `25.0.3.0.9-2.fc44` | Fedora updates | Install as host Java runtime. |
| `java-25-openjdk-devel` | `25.0.3.0.9-2.fc44` | Fedora updates | Install as host Java development kit. |
| `maven` | `3.9.11-11.fc44` | Fedora | Install as host baseline Maven. |

`java-latest-openjdk` currently points to OpenJDK 26, so it is intentionally not used as the host default. The story wants Java 25 LTS, not "whatever hyperspace lane Fedora latest points to today."

`java-21-openjdk` and `java-21-openjdk-devel` were not available by those package names during the check. Alternate-JDK guidance remains repository-owned: use Maven/Gradle toolchains, containers, or a future documented JDK source when a project genuinely requires Java 21 or another release.

### Part 6: Java 25 And Maven Smoke Test

Status: PASS.

Java 25 and Maven were installed from Fedora packages and Java 25 is the active host default.

| Component | Version / evidence | Owner |
|---|---|---|
| Java runtime | OpenJDK `25.0.3` / `25.0.3+9` | Fedora `java-25-openjdk` |
| Java compiler | `javac 25.0.3` | Fedora `java-25-openjdk-devel` |
| Maven | Apache Maven `3.9.11` | Fedora `maven` |
| Java alternative | `/usr/lib/jvm/java-25-openjdk/bin/java` | Fedora alternatives |
| javac alternative | `/usr/lib/jvm/java-25-openjdk/bin/javac` | Fedora alternatives |

Smoke project:

- Created a disposable Maven project under `/home/serge/forge-smoke/story-14/java-smoke`.
- Set `maven.compiler.release` to `25`.
- Compiled with `mvn -q package`.
- Ran `local.forge.App` successfully.
- Output included `Java smoke online` and `Runtime: 25.0.3+9`.
- Removed the disposable project afterward.

Maven emitted a Guice-related `sun.misc.Unsafe` deprecation warning while running on Java 25. The build still succeeded. Treat this as upstream dependency noise from Fedora Maven's dependency stack, not a Forge misconfiguration. The lightsaber ignited; it just made one dramatic prequel sound effect.

### Part 6A: Java Version Management With SDKMAN

Status: PASS.

SDKMAN is installed for the `serge` development user to support future per-project Java version switching. Fedora Java 25 remains the host-owned default for story #14.

| Component | Version / evidence | Owner |
|---|---|---|
| SDKMAN script | `5.23.0` | user-level installer under `/home/serge/.sdkman` |
| SDKMAN native | `0.7.34` | user-level installer under `/home/serge/.sdkman` |
| shell initialization | Added to `/home/serge/.bashrc` | user-level shell config |
| host Java default | OpenJDK `25.0.3` | Fedora packages and alternatives |

Validation:

- `source "$HOME/.sdkman/bin/sdkman-init.sh"` makes `sdk` available in the current shell.
- `sdk version` reports script `5.23.0` and native `0.7.34`.
- `java -version` remains OpenJDK `25.0.3`.
- `javac -version` remains `25.0.3`.
- `which java` and `which javac` resolve to `/usr/bin/java` and `/usr/bin/javac`, so SDKMAN did not hijack the active host default.

Policy:

- Fedora Java 25 remains the host default.
- SDKMAN is available for project-specific JDK selection.
- Projects should declare Java requirements through Maven/Gradle toolchains, wrappers, `.sdkmanrc`, or containers.
- Do not silently change the host default for one project.

The model is now nicely layered: Fedora Java 25 is the factory engine in Forge, SDKMAN is the rack of swappable engines in the hangar, and Maven or Gradle toolchains are the mission checklist that says which engine this particular starfighter needs.

### Part 7: Node Version Management With nvm

Status: PASS.

Node.js is managed for the `serge` development user with `nvm`, while Fedora's `nodejs22` packages remain documented as incidental packages pulled in during earlier host-tool installation.

| Component | Version / evidence | Owner |
|---|---|---|
| `nvm` | `0.40.5` | user-level installer under `/home/serge/.nvm` |
| default Node.js | `v24.18.0` | `nvm`, alias `default -> lts/*` |
| npm | `11.16.0` | bundled with nvm-managed Node.js |
| Fedora Node.js side effect | `nodejs22-22.22.2`, `nodejs22-npm-10.9.7` | Fedora packages, not the selected development default |

Validation:

- The `nvm` installer added source lines to `/home/serge/.bashrc`.
- A repeated install command detected the existing checkout and updated/reused it safely.
- `nvm install --lts` installed Node.js `v24.18.0`.
- `nvm alias default 'lts/*'` set the default to the current LTS line.
- `which node` and `which npm` point under `/home/serge/.nvm/versions/node/v24.18.0/bin/`.
- A disposable `.nvmrc` containing `lts/*` selected Node.js `v24.18.0`.

Policy:

- Use `nvm` for interactive development shells owned by `serge`.
- Repositories that require a specific Node.js version should include `.nvmrc` or equivalent metadata.
- Do not install global Angular, TypeScript, or project-framework CLIs for this baseline.
- Services, cron jobs, and non-interactive automation must not assume `nvm` is loaded unless that is explicitly configured and documented.

This is the starship loadout model: Fedora left an older fighter in the hangar, but `nvm` lets the pilot choose the right hyperdrive for each mission.

### Part 8: Repository-Local TypeScript Smoke Test

Status: PASS.

The TypeScript smoke test proved the intended Node ownership model:

- Node.js is provided by user-level `nvm`.
- The repository chooses its Node line with `.nvmrc`.
- TypeScript is installed locally in the project, not globally on Forge.

Validation evidence:

| Check | Result |
|---|---|
| `nvm use default` | selected Node.js `v24.18.0` with npm `11.16.0` |
| `.nvmrc` | `lts/*`, resolved to Node.js `v24.18.0` |
| `npm install --save-dev typescript` | succeeded; 0 vulnerabilities reported |
| TypeScript compiler | `typescript@7.0.2`, invoked through `npx tsc` |
| Lockfile | `package-lock.json` created in the disposable repository |
| Compile | `npx tsc` succeeded |
| Runtime | `node dist/index.js` printed `Ada says TypeScript is green across the board.` |
| Global framework packages | none installed for this baseline |

Ada's motorcycle cleared the TypeScript launch bay. The compiler stayed repository-local, which is exactly the point: no global Angular CLI, no global TypeScript compiler, no "this only builds on my workstation because I installed things during a late-night side quest."

### Part 9: Python And uv

Status: PASS.

Forge already had Fedora Python `3.14.6`, which matched the latest stable Python release at the time of validation. The system Python remains Fedora-owned. Project Python environments are owned by `uv`.

| Component | Version / evidence | Owner |
|---|---|---|
| System Python | `Python 3.14.6`, `/usr/bin/python3`, `python3-3.14.6-1.fc44` | Fedora |
| uv | `0.11.28`, `/home/serge/.local/bin/uv` | user-level installer |
| Project Python | `Python 3.14.6` inside `.venv` | `uv` project environment |
| Ruff | `0.15.21` | project dev dependency |
| pytest | `9.1.1` | project dev dependency |

Validation:

- `uv init python-smoke` created a disposable project.
- `uv add --dev pytest ruff` created `.venv`, `.python-version`, `pyproject.toml`, and `uv.lock`.
- `uv run ruff check .` passed.
- `uv run pytest` collected one test and passed.
- `uv tree` showed only project-local dev dependencies.
- The disposable project was removed afterward.

Python gets a clean lab bench: Fedora owns the ship's life-support Python, and `uv` gives each away team the sealed environment suit it needs. No global `pip install` spice spill, no sandworm.

### Part 10: Repository-Defined Container Environment

Status: PASS.

A disposable repository-owned container definition was built and run successfully.

Validation evidence:

| Check | Result |
|---|---|
| Build context | Disposable `/home/serge/forge-smoke/story-14/container-smoke` workspace |
| Base image | `alpine:latest` |
| Repository-owned files | `Dockerfile`, `smoke.sh` |
| Build command | `docker build -t forge-container-smoke:story-14 .` succeeded |
| Run command | `docker run --rm forge-container-smoke:story-14` succeeded |
| Runtime output | `Container smoke online. Red Five standing by.` |
| Container shell | GNU bash `5.3.9` from Alpine package install |
| Cleanup | Image removed with `docker image rm`; filtered image list empty afterward |

This proves future repositories can bring their own container definition without undocumented host customization. The away team carried its own gear locker, completed the mission, and did not leave cargo crates in the hallway.

### Part 11: PostgreSQL Compose Service Boundary

Status: PASS.

PostgreSQL was validated as a temporary Compose-owned project service, not a permanent Forge host service.

Validation evidence:

| Check | Result |
|---|---|
| Host PostgreSQL server package | `postgresql-server` is not installed |
| Host `psql` command | not installed before smoke test |
| Compose image | `postgres:latest` |
| Compose container | `postgres-smoke-postgres-1` started and became `healthy` |
| Exposed host ports | Compose reported `5432/tcp` inside the container only; no host port mapping was declared |
| Connection check | `select current_database(), current_user;` returned `smoke`, `smoke` |
| Cleanup | `docker compose down --volumes --remove-orphans` removed the container and network |
| Post-cleanup containers | none matching `postgres-smoke` |
| Post-cleanup volumes | none matching `postgres-smoke` |

The database was a guest star, not a roommate. It appeared, answered one SQL question, and left before putting its name on the lease.

### Part 12: Post-Install Security Baseline Check

Status: PASS.

After installing the toolchain and running container/Compose smoke tests, the story #13 host baseline remained intact.

Forge-side validation:

| Check | Result |
|---|---|
| Failed systemd units | none |
| `firewalld`, `sshd`, `cockpit.socket`, `docker`, `dnf5-automatic.timer` | active and enabled |
| Listening sockets | SSH `:22`, Cockpit `:9090`, loopback DNS stubs, loopback chrony |
| firewalld runtime/permanent | both keep empty `services`, `forward: no`, and LAN-scoped SSH/Cockpit rich rules |
| Docker TCP listener | none |
| Containers | none remaining |
| Volumes | none remaining |
| Docker networks | only default `bridge`, `host`, and `none` networks |

MacBook LAN reachability:

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
| 2375 | refused |
| 2376 | refused |

No project service, database, application port, or Docker daemon TCP port remained exposed. The cargo bay doors are closed; the motorcycle is back on the stand; the station is not broadcasting "free loot" to the LAN.

### Part 13: Reboot Validation

Status: PASS.

Forge rebooted successfully after the toolchain installation and the workbench remained functional.

Post-reboot service state:

| Service | Result |
|---|---|
| failed systemd units | none |
| `firewalld` | active and enabled |
| `sshd` | active and enabled |
| `cockpit.socket` | active and enabled |
| `docker` | active and enabled |
| `dnf5-automatic.timer` | active and enabled |

Post-reboot tool validation:

| Tool | Result |
|---|---|
| Git | `2.55.0` |
| Docker Engine | `29.6.1` |
| Docker Compose | `v5.3.1` |
| Java | OpenJDK `25.0.3` |
| javac | `25.0.3` |
| Maven | `3.9.11` |
| SDKMAN | script `5.23.0`, native `0.7.34` |
| Node.js | `v24.18.0` through `nvm` |
| npm | `11.16.0` |
| nvm | `0.40.5` |
| Python | `3.14.6` |
| uv | `0.11.28` |

Post-reboot security validation:

- Listening sockets remained limited to SSH `:22`, Cockpit `:9090`, loopback DNS, and loopback chrony.
- Runtime and permanent firewalld state still matched.
- SSH and Cockpit remained LAN-scoped through firewalld rich rules.
- Docker exposed no TCP listener.
- MacBook LAN checks showed TCP `22` and `9090` reachable.
- MacBook LAN checks showed TCP `80`, `443`, `3000`, `5432`, `8080`, `8443`, `2375`, and `2376` refused.

The save point worked. Forge came back with the toolchain intact, the firewall still wearing armor, and no unexpected boss music from exposed ports.

### Part 14: Final Cleanup

Status: PASS.

Smoke-test artifacts were removed after validation.

| Artifact type | Result |
|---|---|
| Containers | none remaining |
| Images | `alpine:latest`, `hello-world:latest`, and `postgres:latest` removed; image list empty |
| Volumes | none remaining |
| Networks | only Docker defaults remain: `bridge`, `host`, `none` |
| Build cache prune | completed; reclaimed `0B` |
| Smoke workspace | `/home/serge/forge-smoke/story-14` removed |

The lab bench is clean. No loot crates, no orphaned database volumes, no suspicious glowing artifact humming in the corner.
