# ADR-0007: Isolate Autonomous AI Agents

## Status

Accepted

## Context

The home engineering platform will include Hermes AI as a persistent personal AI-agent runtime.

Unlike interactive coding assistants, a persistent agent may combine memory, scheduled tasks, tool execution, filesystem access, and long-lived credentials. Running such an agent directly on the host with broad permissions would create an unnecessarily large security boundary and could expose development repositories, infrastructure credentials, personal data, or management systems.

## Decision

Hermes AI and any future persistent autonomous agents will run inside a dedicated, least-privilege runtime.

The initial runtime will be Docker Compose on the Fedora development server. Kubernetes may become the runtime after the service is stable and the cluster is operational.

The runtime must:

- Use a dedicated non-root user or rootless container.
- Use a dedicated private container network.
- Have no Docker socket access.
- Have no unrestricted `sudo` access.
- Have no direct access to the hardware KVM or work laptop.
- Have no access to personal SSH private keys.
- Use dedicated least-privilege service credentials.
- Use read-only mounts by default.
- Have one explicit writable workspace.
- Be reachable only from LAN or VPN.
- Keep secrets outside Git.
- Log tool invocations and administrative actions.

High-impact actions require explicit human approval, including:

- Sending external communications.
- Pushing or merging source-control changes.
- Deploying applications.
- Modifying infrastructure or security controls.
- Destructive filesystem operations.
- Accessing or modifying financial records.

## Rationale

A persistent AI agent is closer to a privileged service account than to a passive chatbot. Isolation limits blast radius, makes permissions auditable, and allows the service to evolve without granting it ownership of the development host.

Starting with Docker Compose keeps the first deployment understandable and avoids making Kubernetes a prerequisite. A later Kubernetes migration remains possible after storage, secrets, observability, and policy controls are proven.

## Consequences

Positive consequences:

- Reduced blast radius.
- Clear credential ownership.
- Auditable permissions.
- Easier backup and recovery.
- Portable path to Kubernetes.
- Safer experimentation with tools and memory.

Negative consequences:

- Additional configuration and secrets management.
- Some tools may require explicit adapters or narrowly scoped mounts.
- Human approval gates reduce full autonomy.
- Local model support must be introduced separately from the initial agent deployment.

## Rejected Alternatives

### Run Hermes AI directly as the normal development user

Rejected because it would inherit broad access to repositories, SSH keys, credentials, and host tooling.

### Give Hermes AI access to the Docker socket

Rejected because Docker socket access is effectively host-level administrative access.

### Deploy directly to Kubernetes first

Rejected because Kubernetes is not yet operational and would add platform complexity before the agent's behavior and requirements are understood.

### Grant broad credentials and rely on prompt instructions

Rejected because prompts are not an enforceable security boundary.
