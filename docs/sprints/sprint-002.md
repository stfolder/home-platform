# Sprint 2: Forge Workbench

## Status

In progress.

## Duration

Two weeks.

## Sprint goal

Turn Forge into a secure, practical remote development workstation with a layered host-and-project environment model, AI-assisted terminal workflows, first-class IntelliJ support from the MacBook, flexible VS Code workflows, secure VPN-only access from outside the LAN, and a useful iPad access path.

## Capacity

Planned capacity: **10 story points**.

Planning assumptions:

- Four to five focused sessions per week when normal life permits.
- One primary story in progress at a time.
- Sprint 1 delivered 13 points in an unusually concentrated weekend; that is not yet a stable velocity signal.
- Documentation should answer concrete operational or architectural questions and avoid duplicating transient implementation detail.

## Original commitment

| Issue | Story | Points | Initial board status | Dependency |
|---|---|---:|---|---|
| #13 | Configure host firewall and update policy | 3 | Ready | #12 |
| #14 | Install core development toolchain | 5 | Backlog | #13 |

Original commitment: **8 story points**.

Both committed stories are complete. Forge now has the secure host baseline and layered development toolchain required for additional workstation capabilities.

## Pulled stretch work

| Issue | Story | Points | Pull status | Dependency |
|---|---|---:|---|---|
| #18 | Establish VPN-only remote access to Forge | 3 | Pulled into Sprint 2 | #13 |

Issue #18 is intentionally recorded separately from the original commitment. It enables secure MacBook and iPad access from outside the home LAN without exposing Forge services directly to the Internet.

Current planned scope after the pull: **11 story points**, consisting of **8 committed points** plus **3 pulled stretch points**.

## Additional stretch candidates

| Issue | Story | Points | Pull condition | Dependency |
|---|---|---:|---|---|
| #19 | Install and validate AI coding harnesses on Forge | 3 | Pull after #14; keep one primary story in progress | #14 |
| #15 | Validate IntelliJ, VS Code, and iPad remote development workflows | 5 | Pull only when #14 and #18 are complete or clearly on track | #14, #18 |

If #19 is pulled, delivered scope can reach **14 points**. If both #19 and #15 are pulled and completed, maximum Sprint 2 scope becomes **19 points**.

#19 establishes safe terminal-based AI coding through Codex, OpenCode, or another approved harness. #15 remains independent so normal IDE and remote-development workflows do not depend on an AI provider or agent being available.

## Development architecture

Sprint 2 establishes a layered model:

1. **Forge host baseline**
   - administration and shell tools
   - Git and diagnostics
   - Docker Engine and Docker Compose
   - Java 25 LTS, Node.js LTS, and Python baselines

2. **Repository-owned environments**
   - Maven or Gradle project requirements and toolchains
   - project-local Python environments
   - package-manager metadata and lockfiles
   - dev containers where they materially improve reproducibility

3. **Compose-hosted project services**
   - PostgreSQL, Redis, Kafka, LocalStack, Keycloak, and similar dependencies remain separate from the host unless another story explicitly approves host installation

4. **AI coding harnesses**
   - run as user-level interactive tools under the approved development account
   - orchestrate the existing Forge toolchain rather than replacing it
   - keep provider credentials, sessions, caches, and private configuration outside Git
   - expose no public or LAN-facing control service
   - require deliberate human intent for destructive commands, broad changes, commits, pushes, deployments, and infrastructure mutation

5. **Thin clients**
   - MacBook IntelliJ is the primary Java and Spring cockpit
   - MacBook VS Code is the flexible polyglot and infrastructure client
   - iPad reaches the same Forge-hosted repositories through a browser IDE, SSH terminal, or both

6. **Remote-access boundary**
   - pfSense owns the VPN perimeter
   - approved MacBook and iPad clients reach Forge through VPN-only private paths
   - SSH, IDE backends, browser IDEs, databases, AI harness controls, and application services are not exposed directly to the Internet

## Execution order

1. Complete and accept #13.
2. Complete and accept #14.
3. Pull #19 when AI-assisted terminal development is the next priority, or proceed with #18 when outside-LAN access is the next priority.
4. Complete #18 before requiring outside-LAN validation in #15.
5. Pull #15 only when #14 and #18 are complete or clearly on track.
6. Preserve the one-primary-story work-in-progress limit.

#19 and #18 are independent after #14 and may be executed in either order. #15 does not depend on #19, but it should record how the selected harness behaves through IntelliJ, VS Code, browser, or iPad workflows when useful.

## Sprint success criteria

Sprint 2 succeeds on its original commitment when:

- Forge exposes only intentionally approved host services on the home LAN.
- Firewall and update behavior survive reboot.
- Active listening ports and update procedures are documented.
- Core Java, Node.js/TypeScript, Python, Git, shell, Docker, Compose, and repository-defined container tooling pass documented smoke tests.
- The host-versus-project ownership boundary is documented clearly.
- Tool installation avoids unnecessary global state and privileges.
- No secret, private key, token, database password, or unique machine identifier is committed.

Pulled stretch success for #18 additionally requires:

- MacBook and iPad establish the approved VPN from outside the home LAN.
- `ssh forge` or the approved private iPad path works only while the VPN is connected.
- Private DNS or an approved equivalent resolves Forge over the VPN.
- No Forge service is exposed directly to the Internet.
- Client enrollment, revocation, routing, DNS, and recovery are documented without committing VPN secrets.

Additional stretch success for #19 requires:

- one primary AI coding harness is selected, installed, and usable in a fresh SSH session
- authentication and revocation are documented without exposing credentials
- a disposable repository completes inspect, bounded-edit, diff-review, and build/test workflows
- shell, repository-write, Docker, and destructive-action boundaries are documented plainly
- no AI harness daemon, callback listener, control port, or web UI remains exposed
- iPad, browser, and IDE limitations are handed to #15 rather than expanding #19

Additional stretch success for #15 requires:

- IntelliJ remote development supports a representative Java project through build, test, run, and debug.
- IntelliJ Database Tools can query a representative Docker Compose-hosted PostgreSQL service without public exposure.
- VS Code connects through the documented `forge` SSH alias and completes a representative remote development loop.
- One repository-defined Docker Dev Container workflow succeeds on Forge.
- A useful iPad development path reaches the same Forge-hosted project environment locally and, through #18, from outside the LAN.

## Review evidence

The sprint review should include:

- effective firewalld configuration and active listening-port review
- update-policy behavior and manual update/reboot procedure
- version and smoke-test evidence for the installed host toolchains
- representative Docker Compose validation
- proof of the host-versus-project environment boundary
- for #19, harness version, ownership, safe repository workflow, authentication boundary, process/listener review, and cleanup
- for #18, outside-LAN MacBook and iPad VPN validation plus proof that remote Forge access disappears after VPN disconnect
- if #15 is pulled, IntelliJ Java and database workflows, VS Code Remote SSH, a dev container, and an iPad development action
- deviations, follow-up issues, and architecture decisions

## Documentation strategy

Each document created or updated in this sprint must answer a concrete question:

- `host-baseline.md`: What is Forge allowed to expose, and how is it kept current?
- `development-toolchain.md`: Which tools belong on Forge, which belong to repositories, and how are they validated?
- `ai-coding-harnesses.md`: Which AI coding harness is primary, what access does it receive, and how are authentication and safe operation managed?
- `vpn-remote-access.md`: How do approved devices reach Forge securely from outside the LAN?
- `remote-development.md`: How do the MacBook and iPad use Forge safely and effectively?

Avoid command transcripts, duplicated inventories, machine-specific caches, raw AI conversation histories, complete VPN profiles, and security-sensitive exports unless they are needed for recovery or reproducibility and safe for the public repository.

## Risks

- Firewall changes could interrupt SSH access if applied without an existing recovery path.
- Installing many tools globally can create an unreproducible workstation.
- Docker-group membership grants root-equivalent capability.
- AI harnesses can execute destructive commands, access sensitive files, or retain repository content and credentials in local caches.
- Installing multiple harnesses without selecting a primary path can create update and workflow drift.
- Making every workflow dev-container-dependent can create unnecessary friction.
- Broad VPN rules can expose more of the home LAN than required.
- VPN DNS or routing may appear connected while Forge remains unreachable.
- IntelliJ indexing, language servers, browser IDEs, and remote backends may expose resource or compatibility limits.
- Pulling #18, #19, and #15 raises maximum scope to 19 points, well above the original 8-point commitment.

## Out of scope

- Ansible-managed workstation configuration.
- Terraform.
- Kubernetes.
- Hermes or other persistent AI runtime deployment.
- Autonomous unattended development agents.
- Public AI harness, browser-IDE, or SSH exposure.
- Direct router port forwarding to Forge.
- Centralized identity or SSH certificates.
- Corporate laptop integration.

## Retrospective questions

- Was the two-week sprint cadence better aligned with normal availability?
- Was eight committed points appropriately conservative?
- Was pulling #18 into Sprint 2 a healthy scope adjustment?
- Did #19 provide useful AI-assisted development without creating credential or permission anxiety?
- Which toolchain or ownership decision created the most friction?
- Did the reduced documentation rule keep the docs easier to navigate?
- Did the hybrid host-plus-project model feel convenient rather than ceremonial?
- Did VPN-only access make the iPad workflow genuinely useful outside the LAN?
- Was #15 small enough as one story, or should future IDE and mobile workflows be split?
- Is Forge ready to become the source host for configuration automation?
