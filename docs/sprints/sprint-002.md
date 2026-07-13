# Sprint 2: Forge Workbench

## Status

Planned.

## Duration

Two weeks.

## Sprint goal

Turn Forge into a secure, practical remote development workstation with a layered host-and-project environment model, first-class IntelliJ support from the MacBook, flexible VS Code workflows, and a useful iPad access path.

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

## Stretch candidate

| Issue | Story | Points | Pull condition | Dependency |
|---|---|---:|---|---|
| #15 | Validate IntelliJ, VS Code, and iPad remote development workflows | 5 | Pull only when #13 and #14 are complete or clearly on track | #12, #14 |

If #15 is pulled and completed, total delivered scope will be **13 points**.

The expanded estimate reflects first-class IntelliJ remote Java development, IntelliJ database tooling, VS Code Remote SSH, one dev-container workflow, and a practical iPad path.

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

## Execution order

1. Complete and accept #13.
2. Promote #14 to Ready after #13 is accepted.
3. Complete and accept #14 with documented native, Compose, and dev-container-compatible smoke tests.
4. Pull #15 only when committed work is complete or clearly on track.
5. Preserve the one-primary-story work-in-progress limit.

## Sprint success criteria

Sprint 2 succeeds when:

- Forge exposes only intentionally approved host services on the home LAN.
- Firewall and update behavior survive reboot.
- Active listening ports and update procedures are documented.
- Core Java, Node.js/TypeScript, Python, Git, shell, container, Compose, and minimal dev-container-compatible tooling pass documented smoke tests.
- The host-versus-project ownership boundary is documented clearly.
- Tool installation avoids unnecessary global state and privileges.
- No secret, private key, token, database password, or unique machine identifier is committed.

Pulled stretch success for #15 additionally requires:

- IntelliJ remote development supports a representative Java project through build, test, run, and debug.
- IntelliJ Database Tools can query a representative Compose-hosted PostgreSQL service without public exposure.
- VS Code connects through the documented `forge` SSH alias and completes a representative remote development loop.
- One repository-defined dev-container workflow succeeds on Forge.
- A useful iPad development path reaches the same Forge-hosted project environment and its limitations are documented honestly.

## Review evidence

The sprint review should include:

- effective firewalld configuration and active listening-port review
- update-policy behavior and manual update/reboot procedure
- version and smoke-test evidence for the installed host toolchains
- representative Docker Compose validation
- proof of the host-versus-project environment boundary
- if #15 is pulled, IntelliJ Java and database workflows, VS Code Remote SSH, a dev container, and an iPad development action
- deviations, follow-up issues, and architecture decisions

## Documentation strategy

Each document created or updated in this sprint must answer a concrete question:

- `host-baseline.md`: What is Forge allowed to expose, and how is it kept current?
- `development-toolchain.md`: Which tools belong on Forge, which belong to repositories, and how are they validated?
- `remote-development.md`: How do the MacBook and iPad use Forge safely and effectively?

Avoid command transcripts, duplicated inventories, and machine-specific cache documentation unless they are needed for recovery or reproducibility.

## Risks

- Firewall changes could interrupt SSH access if applied without an existing recovery path.
- #14 may grow through version-manager, repository, Docker, or permissions decisions.
- Installing many tools globally can create an unreproducible workstation.
- Making every workflow dev-container-dependent can create unnecessary friction.
- IntelliJ indexing, language servers, browser IDEs, and remote backends may expose resource or compatibility limits.
- The iPad browser workflow may require a separate secure-access story before use away from the home LAN.

## Out of scope

- Ansible-managed workstation configuration.
- Terraform.
- Kubernetes.
- Hermes or other AI runtime deployment.
- Public browser-IDE or SSH exposure.
- VPN-based remote access unless captured and approved separately.
- Centralized identity or SSH certificates.
- Corporate laptop integration.

## Retrospective questions

- Was the two-week sprint cadence better aligned with normal availability?
- Was eight committed points appropriately conservative?
- Which toolchain or ownership decision created the most friction?
- Did the reduced documentation rule keep the docs easier to navigate?
- Did the hybrid host-plus-project model feel convenient rather than ceremonial?
- Was #15 small enough as one story, or should future IDE and mobile workflows be split?
- Is Forge ready to become the source host for configuration automation?
