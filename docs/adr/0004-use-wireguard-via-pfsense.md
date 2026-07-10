# ADR-0004: Use WireGuard via pfSense for Remote Access

## Status

Accepted

## Context

The development server, hardware KVM, NAS, and future internal platform services must be reachable securely from approved devices while away from home.

Administrative services such as SSH, KVM management, browser IDEs, databases, and AI endpoints must not be exposed directly to the public internet.

The existing network uses pfSense as the software router and firewall.

## Decision

Use WireGuard terminated on pfSense as the primary remote-access mechanism for the home platform.

The VPN is the only planned public administrative entry point.

Requirements:

- Per-device WireGuard keys.
- Ability to revoke individual clients.
- A dedicated VPN subnet.
- Firewall rules granting only required access to internal resources.
- No public SSH, KVM, database, IDE, AI, Docker, or Kubernetes administration ports.
- Tailscale remains a fallback only if WireGuard through pfSense proves impractical or unreliable.

## Rationale

- pfSense is already the network control plane.
- WireGuard is lightweight, fast, and supported on macOS and iPadOS.
- Central VPN termination simplifies firewall policy and auditing.
- Avoiding direct public exposure significantly reduces attack surface.
- Per-device keys provide straightforward revocation.

## Consequences

### Positive

- One controlled entry point for remote administration.
- Internal services behave as LAN resources after VPN connection.
- No need for public SSH or KVM port forwarding.
- Consistent access model across MacBook, iPad, and other approved clients.

### Negative

- Remote access depends on home internet, pfSense, and the VPN endpoint.
- WireGuard configuration and routing must be documented and backed up.
- Recovery may require local access if pfSense or the VPN configuration fails.

## Alternatives Considered

- Tailscale: easier NAT traversal and coordination, but introduces an external coordination dependency.
- Public SSH with hardened configuration: simpler, but increases attack surface and does not solve KVM or other private-service access cleanly.
- Individual reverse proxies and port forwarding: rejected due to complexity and unnecessary public exposure.

## Related ADRs

- ADR-0005: Separate Development, Deployment, and Platform Responsibilities
- ADR-0007: Isolate Autonomous AI Agents
