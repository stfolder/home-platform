# Sprint 2: Forge Workbench

## Status

Planned.

## Duration

Three weeks.

## Sprint goal

Turn Forge into a secure, practical remote development workstation that can build representative Java, TypeScript, and Python projects from the MacBook.

## Capacity

Planned capacity: **10 story points**.

Planning assumptions:

- Four to five focused sessions per week.
- Approximately five to seven engineering hours per week.
- One primary story in progress at a time.
- Sprint 1 delivered 13 points, but a single sprint is not enough evidence to raise planned capacity.

## Original commitment

| Issue | Story | Points | Initial board status | Dependency |
|---|---|---:|---|---|
| #13 | Configure host firewall and update policy | 3 | Ready | #12 |
| #14 | Install core development toolchain | 5 | Backlog | #13 |

Original commitment: **8 story points**.

The sprint intentionally leaves two points of capacity uncommitted because #14 spans several ecosystems and may contain integration friction.

## Stretch candidate

| Issue | Story | Points | Pull condition | Dependency |
|---|---|---:|---|---|
| #15 | Validate VS Code Remote SSH development workflow | 3 | Pull only when #13 and #14 are complete or clearly on track | #12, #14 |

If #15 is pulled and completed, total delivered scope will be **11 points**.

## Execution order

1. Complete and accept #13.
2. Promote #14 to Ready after #13 is accepted.
3. Complete and accept #14 with documented smoke tests.
4. Pull #15 only when committed work is complete or clearly on track.
5. Preserve the one-primary-story work-in-progress limit.

## Sprint success criteria

Sprint 2 succeeds when:

- Forge exposes only intentionally approved host services on the home LAN.
- Firewall and update behavior survive reboot.
- Active listening ports and update procedures are documented.
- Core Java, Node.js/TypeScript, Python, Git, shell, and container tooling pass documented smoke tests.
- Tool installation avoids unnecessary global state and privileges.
- No secret, private key, token, or unique machine identifier is committed.

Pulled stretch success for #15 additionally requires:

- VS Code connects through the documented `forge` SSH alias.
- A representative remote project can be edited, built, tested, committed, and re-opened after reconnecting.
- Git credentials are handled without copying private credentials into the project repository.

## Review evidence

The sprint review should include:

- Effective firewalld configuration and active listening-port review.
- Update-policy behavior and manual update/reboot procedure.
- Version and smoke-test evidence for the installed toolchains.
- A representative Docker Compose validation.
- If #15 is pulled, a complete VS Code Remote SSH development loop.
- Deviations, follow-up issues, and any architecture decisions.

## Risks

- Firewall changes could interrupt SSH access if applied without an existing recovery path.
- #14 may grow through version-manager, repository, Docker, or permissions decisions.
- Installing many tools globally can create an unreproducible workstation.
- Remote IDE behavior may expose file-watching or language-server performance problems.

## Out of scope

- Ansible-managed workstation configuration.
- Terraform.
- Kubernetes.
- Hermes or other AI runtime deployment.
- Public service exposure.
- VPN-based remote access.
- Centralized identity or SSH certificates.

## Retrospective questions

- Was eight committed points appropriately conservative?
- Which toolchain created the most friction?
- Did the security baseline make later installation safer or slower?
- Is Forge ready to become the source host for configuration automation?
- Should Sprint 3 prioritize Ansible, generated inventory, or application delivery?