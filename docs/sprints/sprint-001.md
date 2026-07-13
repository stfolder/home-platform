# Sprint 1: Forge Online

## Status

Active.

## Duration

Three weeks.

## Sprint goal

Prepare and bring Forge online as a documented Fedora development server with a stable identity on the home LAN.

## Capacity

Original planned capacity: **10 story points**.

Planning assumptions:

- Four to five focused sessions per week.
- Approximately five to seven engineering hours per week.
- Approximately fifteen to twenty engineering hours over the sprint.
- One primary story in progress at a time.

This is the first measured sprint. Future capacity must be adjusted from observed velocity rather than optimism.

## Original commitment

| Issue | Story | Points | Initial board status | Dependency |
|---|---|---:|---|---|
| #9 | Prepare Forge for Fedora installation | 2 | Ready | None |
| #10 | Install Fedora Server on Forge | 5 | Backlog | #9 |
| #11 | Configure stable hostname and LAN addressing | 3 | Backlog | #10 |

Original commitment: **10 story points**.

## Pulled stretch work

| Issue | Story | Points | Pull status | Dependency |
|---|---|---:|---|---|
| #12 | Establish secure SSH access from the MacBook | 3 | Ready | #11 |

Issue #12 was pulled into Sprint 1 after the original committed stories were completed or clearly on track. It remains recorded separately from the original commitment so the sprint review can distinguish planned capacity from additional delivered work.

Current sprint scope: **13 story points**, consisting of **10 committed points** plus **3 pulled stretch points**.

## Execution order

1. Complete and accept #9.
2. Complete and accept #10.
3. Complete and accept #11.
4. Pull #12 into the sprint as stretch work.
5. Preserve the one-primary-story work-in-progress limit.
6. Move #12 to Review only after key authentication, negative authentication tests, reboot validation, and documentation are complete.

## Sprint success criteria

Sprint 1 succeeds when:

- Forge is prepared safely for installation.
- Fedora Server is installed on the approved disk.
- The host boots reliably and has wired connectivity.
- The hostname and stable LAN addressing are configured.
- `forge.home.arpa` resolves from an approved LAN client.
- Required hardware, installation, and network documentation is committed.
- No undocumented destructive action or unresolved recovery gap remains.

Pulled stretch success for #12 additionally requires:

- `ssh forge` succeeds from the MacBook using a dedicated key.
- Root SSH login and password-only authentication fail.
- Secure access survives SSH reload and Forge reboot.
- SSH setup, recovery, and key revocation are documented.

## Review evidence

The sprint review should include:

- The running Forge host.
- Successful reboot evidence.
- LAN reachability by hostname.
- Links to documentation produced by the committed stories.
- For #12, successful key-based login plus failed password-only and root-login tests.
- Deviations, follow-up issues, and any ADR changes.
- A clear distinction between the original 10-point commitment and the 3 points pulled afterward.

## Retrospective questions

- Was 10 points realistic for three weeks?
- How many focused sessions were actually available?
- Which work produced unexpected friction?
- Did the stories contain enough documentation guidance?
- Was pulling #12 healthy use of available capacity, or did it create avoidable pressure?
- Should pulled stretch points count separately when calculating future planning velocity?
- Should the next sprint retain, reduce, or increase planned capacity?
