# ADR-0002: Use Ansible for Machine Configuration

## Status

Accepted

## Context

The platform will include a Fedora development server and, later, Kubernetes nodes. Their operating-system configuration should be reproducible, reviewable, and recoverable without relying on undocumented manual steps.

The configuration-management tool should also provide useful professional experience for enterprise and platform-engineering work.

## Decision

Use Ansible to manage host-level configuration for the Fedora development server and future Kubernetes nodes.

Ansible owns:

- Package installation.
- User and group configuration.
- SSH hardening.
- firewalld configuration.
- Docker installation.
- Developer tooling.
- Future NVIDIA host support.
- Kubernetes-node prerequisites.

Ansible does not own cloud infrastructure, pfSense configuration, application deployment logic, or secrets in plaintext.

## Rationale

- Ansible is widely used in enterprise and hybrid infrastructure environments.
- Playbooks provide executable documentation of machine state.
- Idempotent automation supports recovery and repeatable provisioning.
- It complements Terraform rather than competing with it.
- It provides strong career value without requiring a new operating-system model.

## Consequences

### Positive

- Hosts can be rebuilt consistently.
- Configuration changes become reviewable in Git.
- Manual setup drift is reduced.
- The same roles can support future additional nodes.

### Negative

- Ansible introduces its own YAML conventions, module ecosystem, and debugging model.
- Initial automation takes longer than purely manual setup.
- Some bootstrap work must still occur before Ansible can connect safely.

## Alternatives Considered

- Manual documentation only: simpler initially, but fragile and difficult to reproduce.
- Bash provisioning scripts: useful for bootstrap tasks, but less declarative and harder to keep idempotent at scale.
- Nix/NixOS: stronger declarative guarantees, but much steeper learning and migration cost.
- Terraform provisioners: rejected because Terraform should not own operating-system configuration.

## Related ADRs

- ADR-0001: Use Fedora Server as the Development Platform
- ADR-0003: Use Terraform for Cloud Infrastructure
- ADR-0005: Separate Development, Deployment, and Platform Responsibilities
