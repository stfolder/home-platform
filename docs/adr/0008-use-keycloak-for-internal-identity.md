# ADR-0008: Use Keycloak for Internal Identity and Access Management

## Status

Accepted, deferred until after the Kubernetes phase.

## Context

Project Forge will host multiple personal applications and internal services. Managing separate users, passwords, roles, and service credentials for each application would create duplicated security logic and inconsistent access controls.

The platform needs a future internal identity provider that supports:

- OpenID Connect authentication.
- OAuth 2.0 authorization for APIs.
- Users, groups, roles, and service accounts.
- Multi-factor authentication.
- Integration with Java, Spring Security, Angular, and internal platform services.
- Optional single sign-on for Synology DSM where supported.

This capability is not an immediate priority. Once applications depend on centralized identity, the provider becomes a shared platform dependency and requires a stable runtime, database, backups, and recovery path.

## Decision

Use Keycloak as the internal identity provider.

- Deploy Keycloak on Kubernetes after the Kubernetes platform is stable.
- Run Keycloak's PostgreSQL database in a dedicated container on Synology.
- Use a dedicated database and least-privilege database role.
- Integrate the Finance App first, followed by selected internal services.
- Evaluate Synology DSM single sign-on only after local recovery access is verified.
- Retain local break-glass administrator accounts for Synology and critical infrastructure.
- Keep WireGuard, pfSense, Kubernetes recovery, database recovery, and backup restoration independent from Keycloak.

Hermes AI is introduced earlier on Forge because it can remain isolated and non-critical. Keycloak is introduced later because it becomes infrastructure that other services rely on.

## Rationale

Keycloak provides a free and open-source identity platform with strong support for OpenID Connect, OAuth 2.0, SAML, roles, groups, service accounts, MFA, and enterprise application integration.

It also has strong learning value for the Java and Spring ecosystem while solving a real platform need.

Kubernetes is the preferred runtime because Keycloak is a long-running shared service rather than a development tool. Synology is the preferred initial database host because it already provides persistent storage and backup capabilities independent from the Kubernetes node lifecycle.

## Consequences

### Positive

- Applications can use a shared authentication and authorization model.
- Finance App and future services avoid implementing independent user stores.
- Keycloak compute can be rebuilt independently from its database.
- PostgreSQL persistence and backups remain on Synology.
- The platform gains practical OIDC, OAuth 2.0, RBAC, MFA, and service-account experience.
- Critical recovery paths remain available when Keycloak is unavailable.

### Negative

- Keycloak, Kubernetes, Synology, PostgreSQL, and the home network become part of the login dependency chain.
- Synology-hosted PostgreSQL is still a single point of failure.
- Database and realm upgrades require explicit backup and restore testing.
- Some Synology protocols and third-party services may not support OIDC and will keep separate credentials.
- Centralized identity adds operational complexity and should not be introduced before enough services justify it.

## Alternatives Considered

### Run Keycloak on Forge before Kubernetes

Rejected as the primary plan because it would create a temporary production-like dependency on the development workstation and require an avoidable migration.

### Run PostgreSQL inside Kubernetes

Deferred because persistent storage and database recovery would become coupled to the initial Kubernetes platform. This can be reconsidered after the cluster and storage design mature.

### Use Authentik or Authelia

Both are viable home-lab identity tools. Keycloak was selected for its broader enterprise adoption, Java ecosystem alignment, authorization capabilities, and professional learning value.

### Keep independent application accounts

Acceptable during early phases but rejected as the long-term model because it duplicates security logic and fragments access management.

## Future Review

Review this decision if:

- Synology proves unsuitable for reliable PostgreSQL hosting.
- Kubernetes is delayed substantially while centralized identity becomes urgent.
- Keycloak becomes too operationally heavy for the number of services using it.
- A different identity provider offers materially better support for required applications.
- High availability becomes necessary.

## Related ADRs

- ADR-0003: Use Terraform for Cloud Infrastructure.
- ADR-0004: Use WireGuard via pfSense for Remote Access.
- ADR-0005: Separate Development, Deployment, and Platform Responsibilities.
- ADR-0007: Isolate Autonomous AI Agents.
