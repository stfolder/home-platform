# ADR-0005: Separate Development, Deployment, and Platform Responsibilities

## Status

Accepted

## Context

The home platform includes multiple machines and services with different lifecycle, security, and reliability needs.

Combining development, application hosting, infrastructure automation, and autonomous AI services on one machine would increase operational coupling and make experimentation riskier.

The platform currently includes:

- A Fedora development server named `forge`.
- A Raspberry Pi 4 running Ubuntu Server and hosting the Finance App.
- A future Kubernetes deployment platform.
- pfSense, a NAS, and a hardware KVM-over-IP appliance.

## Decision

Assign every machine and major platform component one primary responsibility.

Current responsibility boundaries:

- `forge`: development workstation, local development services, coding agents, and controlled AI experimentation.
- Raspberry Pi: current stable application deployment target.
- Kubernetes cluster: future primary platform for long-running deployed applications.
- pfSense: network routing, firewall policy, and VPN termination.
- NAS: backup and recovery storage.
- Hardware KVM: OS-independent access to the work laptop, subject to employer policy.

Development and deployment remain separate concerns. Kubernetes is not required before the development platform and Raspberry Pi delivery workflow are stable.

## Rationale

- Failures in a deployment target should not interrupt the development environment.
- Development experiments should not destabilize hosted applications.
- Clear ownership improves automation, documentation, backup, and security design.
- The separation mirrors modern professional platform architecture.
- Components can be replaced or evolved independently.

## Consequences

### Positive

- Reduced operational coupling.
- Safer experimentation on the development server.
- Clear migration path from Raspberry Pi deployment to Kubernetes.
- Easier recovery and troubleshooting.
- More realistic platform-engineering experience.

### Negative

- More machines and repositories must be maintained.
- Deployment workflows must explicitly move artifacts between environments.
- Some services may initially be duplicated between development and deployment targets.

## Alternatives Considered

- Run development and deployed applications on `forge`: simpler hardware footprint, but creates unacceptable coupling and weakens the learning architecture.
- Replace the Raspberry Pi immediately with Kubernetes: rejected because it introduces complexity before CI/CD and deployment fundamentals are automated.
- Use only cloud environments: rejected because the existing hardware provides lower-cost learning and local control.

## Related ADRs

- ADR-0001: Use Fedora Server as the Development Platform
- ADR-0002: Use Ansible for Machine Configuration
- ADR-0003: Use Terraform for Cloud Infrastructure
- ADR-0004: Use WireGuard via pfSense for Remote Access
- ADR-0006: Application Repositories Own Deployment Configuration
- ADR-0007: Isolate Autonomous AI Agents
