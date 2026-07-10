# ADR-0006: Application Repositories Own Deployment Configuration

## Status

Accepted.

## Context

The home engineering platform separates platform architecture, machine configuration, cloud infrastructure, and application development into distinct repositories.

The Finance App currently runs on a Raspberry Pi 4 with Ubuntu Server and is deployed manually through SSH. A future Kubernetes cluster will become the primary deployment platform for personal applications.

A separate deployment repository was considered for Finance App deployment configuration. For a single-owner personal platform, that separation would add coordination and versioning overhead without a clear operational benefit.

## Decision

Each application repository owns the artifacts required to build, release, deploy, and operate that application.

The `finance-app` repository may contain:

- GitHub Actions workflows.
- Container build definitions.
- Docker Compose configuration.
- Raspberry Pi deployment scripts.
- Health-check and rollback scripts.
- Kubernetes manifests.
- Helm charts.
- Application-specific release and operational documentation.

The platform repositories retain separate responsibilities:

- `home-platform`: architecture, decisions, roadmap, and operational planning.
- `home-ansible`: host and node configuration.
- `home-terraform`: cloud infrastructure.

Platform repositories must not contain Finance App-specific deployment logic.

No separate `deployment`, `finance-app-deployment`, or `finance-app-ops` repository will be created unless future scale or team ownership provides a concrete reason.

## Rationale

Deployment artifacts evolve with application code and should be versioned with compatible application changes.

Keeping deployment configuration in the application repository provides:

- Atomic changes to code and deployment configuration.
- Easier rollback to a known compatible release.
- Less cross-repository coordination.
- A simpler workflow for a single developer.
- A natural transition from Raspberry Pi deployment to Kubernetes.

## Consequences

### Positive

- One repository contains everything needed to build and deploy the application.
- CI/CD workflows can validate application and deployment changes together.
- Deployment targets may evolve without moving application-specific configuration between repositories.
- Platform repositories remain application-agnostic.

### Negative

- Application repositories become slightly broader in scope.
- Shared deployment patterns may be duplicated between applications until common platform tooling is justified.
- Repository permissions cannot independently separate application code from its deployment configuration.

## Follow-up

- Document the current manual Finance App deployment process.
- Add CI validation before automating delivery.
- Automate Raspberry Pi deployment with health verification and rollback.
- Revisit GitOps strategy before Kubernetes deployment automation is selected.