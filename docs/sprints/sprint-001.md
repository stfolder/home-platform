# Sprint 1: Forge Online

## Status

Complete.

## Duration

Three weeks.

## Sprint goal

Prepare and bring Forge online as a documented Fedora development server with a stable identity on the home LAN.

**Result:** Achieved.

## Capacity and delivery

- Original planned capacity: **10 story points**.
- Original commitment delivered: **10 of 10 points**.
- Pulled stretch work delivered: **3 points**.
- Total delivered: **13 story points**.

The pulled work remains separate from the original commitment so future planning does not treat stretch delivery as guaranteed capacity.

## Delivered stories

| Issue | Story | Points | Commitment | Final status |
|---|---|---:|---|---|
| #9 | Prepare Forge for Fedora installation | 2 | Committed | Done |
| #10 | Install and validate Fedora Server on Forge | 5 | Committed | Done |
| #11 | Establish Forge's permanent network identity | 3 | Committed | Done |
| #12 | Establish secure SSH access from the MacBook | 3 | Pulled stretch | Done |

## Delivered platform state

- Fedora Server 44 is installed and boots reliably.
- Forge operates headlessly in the server room.
- Permanent hostname is `forge`.
- Canonical LAN name is `forge.home.arpa`.
- Stable addressing is provided by a pfSense DHCP reservation.
- Secure SSH uses a dedicated MacBook key and the `ssh forge` alias.
- Root SSH login, password authentication, and keyboard-interactive authentication are disabled.
- Installation, network identity, recovery, and SSH procedures are documented.
- Follow-up work discovered during delivery is tracked separately.

## Sprint review evidence

- Running Forge host and successful reboot.
- LAN resolution and reachability through `forge.home.arpa`.
- Fresh key-based `ssh forge` login.
- Failed password-only and root-login tests.
- Hardware, installation, network, and SSH documentation committed.
- Public-repository review completed for unique machine identifiers and secrets.

## Retrospective observations

### What worked

- One primary story in progress at a time.
- Grooming before implementation.
- Explicit safe sequences for destructive and lockout-prone work.
- Capturing discoveries as follow-up issues rather than expanding stories silently.
- Reviewing documentation before accepting stories.

### Improvements carried forward

- Keep unique machine identifiers in private or host-local generated inventory.
- Keep planned capacity conservative until several sprint data points exist.
- Record final story status in sprint documents as part of sprint closure.
- Preserve a clear boundary between committed and pulled stretch work.

## Next sprint

Sprint 2 focuses on turning Forge from a secure server into a practical remote development workstation.

See [Sprint 2: Forge Workbench](sprint-002.md).