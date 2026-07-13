# Home Platform Naming

## Status

Initial naming convention established for story #11.

## Goals

- Make devices and services easy to recognize during operations.
- Keep names short enough for shell, SSH, DNS, and dashboards.
- Avoid names that encode temporary implementation details.
- Avoid `.local` names because `.local` can conflict with mDNS behavior.
- Keep public DNS, TLS, reverse proxy naming, and dynamic DNS out of scope until later stories require them.

## Local Domain

Canonical home-platform LAN names use:

```text
home.arpa
```

`home.arpa` is preferred for local home-network naming. The platform should not use `.local` for canonical hostnames.

## Hostname Pattern

Use short functional hostnames for durable machines.

Examples:

| Hostname | Purpose | Canonical DNS name |
|---|---|---|
| `forge` | Fedora development workstation | `forge.home.arpa` |

Guidelines:

- Prefer a single clear noun for a durable host.
- Use service names only when a service has its own stable network identity.
- Do not encode hardware model, operating system version, or temporary role into the hostname.
- Do not reuse hostnames for unrelated machines.
- Document exceptions with an ADR when they affect architecture or automation.

## Service Names

Service names should be introduced only when the service needs to be reached independently from the host.

Future examples may include:

| Name | Purpose | Status |
|---|---|---|
| `finance` | Finance App service endpoint | Future |
| `grafana` | Observability dashboard | Future |

Service naming, internal TLS, reverse proxy rules, and split-horizon DNS are future concerns and are not part of story #11.

## Validation Expectations

A canonical name is accepted when:

- The owning host or service is documented.
- The DNS name resolves from an approved LAN client.
- The name survives reboot or lease renewal.
- At least one additional LAN device can validate resolution or connectivity.

## Security Notes

- Names are not secrets.
- Names must not imply that a service is publicly reachable.
- Credentials, private keys, pfSense exports containing secrets, and API tokens must not be committed.

