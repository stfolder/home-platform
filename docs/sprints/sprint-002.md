# Sprint 2: Forge Workbench

## Status

Planned.

## Duration

Two weeks.

## Sprint goal

Turn Forge into a secure, practical remote development workstation with a layered host-and-project environment model, first-class IntelliJ support from the MacBook, flexible VS Code workflows, secure VPN-only access from outside the LAN, and a useful iPad access path.

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

The sprint intentionally leaves two points of capacity uncommitted because #14 spans container, Java, Node.js, Python, package-ownership, and reproducibility decisions.

## Pulled stretch work

| Issue | Story | Points | Pull status | Dependency |
|---|---|---:|---|---|
| #18 | Establish VPN-only remote access to Forge | 3 | Pulled into Sprint 2 | #13 |

Issue #18 is intentionally recorded separately from the original commitment. It enables secure MacBook and iPad access from outside the home LAN without exposing Forge services directly to the Internet.

Current planned scope after the pull: **11 story points**, consisting of **8 committed points** plus **3 pulled stretch points**.

## Additional stretch candidate

| Issue | Story | Points | Pull condition | Dependency |
|---|---|---:|---|---|
| #15 | Validate IntelliJ, VS Code, and iPad remote development workflows | 5 | Pull only when #13, #14, and #18 are complete or clearly on track | #14, #18 |

If #15 is also pulled and completed, total delivered scope will be **16 points**.

The expanded estimate reflects first-class IntelliJ remote Java development, IntelliJ database tooling, VS Code Remote SSH, one dev-container workflow, and a practical iPad path through the VPN when outside the LAN.

## Development architecture

Sprint 2 establishes a layered model:

1. **Forge host baseline**
   - administration and shell tools
   - Git and diagnostics
   - container runtime and Compose
   - modest native Java, Node.js, and Python baselines

2. **Repository-owned environments**
   - Maven or Gradle project requirements
   - project-local Python environments
   - package-manager metadata
   - dev containers where they materially improve reproducibility

3. **Compose-hosted project services**
   - PostgreSQL, Redis, Kafka, LocalStack, Keycloak, and similar dependencies remain separate from the host unless another story explicitly approves host installation

4. **Thin clients**
   - MacBook IntelliJ is the primary Java and Spring cockpit
   - MacBook VS Code is the flexible polyglot and infrastructure client
   - iPad reaches the same Forge-hosted repositories through a browser IDE, SSH terminal, or both

5. **Remote-access boundary**
   - pfSense owns the VPN perimeter
   - approved MacBook and iPad clients reach Forge through VPN-only private paths
   - SSH, IDE backends, browser IDEs, databases, and application services are not exposed directly to the Internet

## Execution order

1. Complete and accept #13.
2. Start #18 after #13 establishes the firewall baseline; validate VPN-only SSH from the MacBook and iPad.
3. Promote #14 to Ready after #13 is accepted; #14 may proceed before or after #18 while preserving one primary story in progress.
4. Complete and accept #14 with documented native, Compose, and dev-container-compatible smoke tests.
5. Pull #15 only when #14 and #18 are complete or clearly on track.
6. Preserve the one-primary-story work-in-progress limit.

## Sprint success criteria

Sprint 2 succeeds on its original commitment when:

- Forge exposes only intentionally approved host services on the home LAN.
- Firewall and update behavior survive reboot.
- Active listening ports and update procedures are documented.
- Core Java, Node.js/TypeScript, Python, Git, shell, container, Compose, and minimal dev-container-compatible tooling pass documented smoke tests.
- The host-versus-project ownership boundary is documented clearly.
- Tool installation avoids unnecessary global state and privileges.
- No secret, private key, token, database password, or unique machine identifier is committed.

Pulled stretch success for #18 additionally requires:

- MacBook and iPad establish the approved VPN from outside the home LAN.
- `ssh forge` or the approved private iPad path works only while the VPN is connected.
- Private DNS or an approved equivalent resolves Forge over the VPN.
- No Forge service is exposed directly to the Internet.
- Client enrollment, revocation, routing, DNS, and recovery are documented without committing VPN secrets.

Additional stretch success for #15 requires:

- IntelliJ remote development supports a representative Java project through build, test, run, and debug.
- IntelliJ Database Tools can query a representative Compose-hosted PostgreSQL service without public exposure.
- VS Code connects through the documented `forge` SSH alias and completes a representative remote development loop.
- One repository-defined dev-container workflow succeeds on Forge.
- A useful iPad development path reaches the same Forge-hosted project environment locally and, through #18, from outside the LAN.

## Review evidence

The sprint review should include:

- effective firewalld configuration and active listening-port review
- update-policy behavior and manual update/reboot procedure
- version and smoke-test evidence for the installed host toolchains
- representative Docker Compose validation
- proof of the host-versus-project environment boundary
- for #18, outside-LAN MacBook and iPad VPN validation plus proof that remote Forge access disappears after VPN disconnect
- if #15 is pulled, IntelliJ Java and database workflows, VS Code Remote SSH, a dev container, and an iPad development action
- deviations, follow-up issues, and architecture decisions

## Documentation strategy

Each document created or updated in this sprint must answer a concrete question:

- `host-baseline.md`: What is Forge allowed to expose, and how is it kept current?
- `development-toolchain.md`: Which tools belong on Forge, which belong to repositories, and how are they validated?
- `vpn-remote-access.md`: How do approved devices reach Forge securely from outside the LAN?
- `remote-development.md`: How do the MacBook and iPad use Forge safely and effectively?

Avoid command transcripts, duplicated inventories, machine-specific caches, complete VPN profiles, and security-sensitive exports unless they are needed for recovery or reproducibility and safe for the public repository.

## Risks

- Firewall changes could interrupt SSH access if applied without an existing recovery path.
- #14 may grow through version-manager, repository, Docker, or permissions decisions.
- Installing many tools globally can create an unreproducible workstation.
- Making every workflow dev-container-dependent can create unnecessary friction.
- Broad VPN rules can expose more of the home LAN than required.
- VPN DNS or routing may appear connected while Forge remains unreachable.
- IntelliJ indexing, language servers, browser IDEs, and remote backends may expose resource or compatibility limits.
- Pulling both #18 and #15 raises total scope to 16 points, well above the original 8-point commitment.

## Out of scope

- Ansible-managed workstation configuration.
- Terraform.
- Kubernetes.
- Hermes or other AI runtime deployment.
- Public browser-IDE or SSH exposure.
- Direct router port forwarding to Forge.
- Centralized identity or SSH certificates.
- Corporate laptop integration.

## Retrospective questions

- Was the two-week sprint cadence better aligned with normal availability?
- Was eight committed points appropriately conservative?
- Was pulling #18 into Sprint 2 a healthy scope adjustment?
- Which toolchain or ownership decision created the most friction?
- Did the reduced documentation rule keep the docs easier to navigate?
- Did the hybrid host-plus-project model feel convenient rather than ceremonial?
- Did VPN-only access make the iPad workflow genuinely useful outside the LAN?
- Was #15 small enough as one story, or should future IDE and mobile workflows be split?
- Is Forge ready to become the source host for configuration automation?
