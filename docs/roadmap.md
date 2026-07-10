# Home Platform Implementation Roadmap

## Status

This document is the current implementation sequence for Project Forge. It refines the older roadmap embedded in `docs/architecture.md` and should be treated as the active phase ordering when the two differ.

## Guiding Rule

Build foundational capabilities before shared dependencies. Experimental services may appear earlier when they are isolated, reversible, and not required by critical applications.

## Phase 1: Fedora Development Server

- Install Fedora Server on Forge.
- Configure hostname, local DNS, SSH keys, host firewall, and automatic security updates.
- Install Git, Docker, Java, Node.js, Python, tmux, Neovim, and remote-development tooling.
- Validate Docker Compose and VPN-only access.

## Phase 2: Ansible Automation

- Create and maintain `home-ansible`.
- Reproduce Forge configuration, package installation, SSH hardening, Docker setup, and developer tooling.
- Document manual recovery procedures and verification commands.

## Phase 3: Application Development Workflow

- Establish the Finance App development workflow.
- Support Java, Angular, TypeScript, and Python tooling.
- Run local dependencies through Docker Compose.
- Configure GitHub CLI, OpenCode, Codex, and repository-level development settings.

## Phase 4: Raspberry Pi Deployment Automation

- Continue using the Raspberry Pi as the current deployment target.
- Replace manual SSH deployment with application-owned GitHub Actions and deployment scripts.
- Add health checks, rollback procedures, backups, and deployment documentation.

## Phase 5: Backup and Basic Monitoring

- Back up repositories, configuration, database dumps, application state, and recovery material to Synology.
- Monitor disk, memory, CPU, Docker, failed services, security updates, and deployment health.
- Test at least one recovery procedure.

## Phase 6: Hermes AI

- Run Hermes AI on Forge through an isolated Docker Compose runtime.
- Use an approved cloud model API initially.
- Limit the first workflows to low-risk documentation, reporting, and draft issue creation.
- Require human approval for repository writes, deployments, infrastructure changes, communications, and access to sensitive data.
- Keep the runtime portable so it can migrate to Kubernetes later.

Hermes is intentionally introduced before Kubernetes because it is experimental, isolated, and not a dependency for critical applications.

## Phase 7: Terraform and AWS Learning

- Create and maintain `home-terraform`.
- Keep Terraform separate from Ansible.
- Begin with small AWS networking, IAM, DNS, storage, and compute exercises.
- Document state and secret-management boundaries.

## Phase 8: Kubernetes Deployment Platform

- Introduce a single-node Kubernetes cluster.
- Deploy suitable personal applications and platform services.
- Add ingress, secrets, storage, GitOps, and multi-node expansion only when justified.
- Define backup and recovery procedures before moving critical shared services.

## Phase 9: Keycloak Identity Platform

- Deploy Keycloak on Kubernetes as the internal OpenID Connect and OAuth 2.0 identity provider.
- Run the Keycloak PostgreSQL database in a dedicated container on Synology.
- Use separate databases and roles for Keycloak and application workloads.
- Integrate the Finance App first, then selected internal services and Synology DSM where supported.
- Retain local emergency administrator accounts for Synology and other critical infrastructure.
- Do not make VPN, pfSense, recovery access, or break-glass administration depend on Keycloak.

Keycloak is deferred until after Kubernetes because it becomes a shared dependency once applications rely on centralized authentication.

## Phase 10: Broader Observability and Platform Services

- Add Prometheus, Grafana, centralized logging, and tracing where useful.
- Monitor Kubernetes workloads, Keycloak, Synology-hosted PostgreSQL, Hermes AI, and application deployments.
- Add alerting only after meaningful service-level signals are defined.

## Phase 11: Local AI Expansion

- Install and validate the RTX 2080 Super and NVIDIA container support.
- Evaluate local model gateways, embeddings, smaller quantized models, and vector databases.
- Optionally migrate Hermes AI to Kubernetes after its behavior, permissions, storage, and recovery process are stable.

## Cross-Cutting Requirements

Every phase must preserve:

- WireGuard and LAN-only administrative access.
- No direct public exposure of internal services.
- Least-privilege credentials and service identities.
- Secrets outside Git.
- Reproducible configuration.
- Documented backup, recovery, and rollback procedures.
