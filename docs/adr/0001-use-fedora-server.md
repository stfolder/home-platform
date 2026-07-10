# ADR-0001: Use Fedora Server as the Development Platform

## Status

Accepted

## Context

The platform needs a persistent Linux development workstation for Java, Angular, TypeScript, Python, Docker, AI tooling, and remote development from MacBook and iPad clients.

The operating system should provide a modern developer experience, current kernels and packages, strong container support, and good compatibility with future NVIDIA GPU experimentation.

## Decision

Use Fedora Server as the canonical operating system for the development server named `forge`.

## Rationale

- Fedora is already familiar and preferred as a desktop Linux environment.
- It provides modern kernels, compilers, Python, container tooling, and security features.
- SELinux and firewalld provide valuable enterprise-relevant experience.
- Fedora aligns well with the platform's goal of improving Linux and infrastructure skills.
- It supports the required Java, Node.js, Python, Docker, VS Code Remote SSH, and NVIDIA workflows.

## Consequences

### Positive

- Modern development toolchain and kernel support.
- Strong professional learning value.
- Good fit for containerized development and future GPU work.
- Consistent operating system choice across the development-server automation.

### Negative

- Fedora has a faster release cadence than Ubuntu LTS or Debian stable.
- Regular distribution upgrades are required.
- Some third-party documentation may target Ubuntu first.

## Alternatives Considered

- Ubuntu Server LTS: lower maintenance and broader third-party documentation, but less aligned with personal preference and the desired modern Fedora workflow.
- Debian stable: conservative and reliable, but slower-moving packages and more friction for some development and GPU tooling.
- NixOS: highly reproducible, but introduces significant additional complexity and learning overhead.

## Related ADRs

- ADR-0002: Use Ansible for Machine Configuration
- ADR-0005: Separate Development, Deployment, and Platform Responsibilities
