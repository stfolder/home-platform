# Sprint 1: Forge Online

## Status

Active.

## Duration

Three weeks.

## Sprint goal

Prepare and bring Forge online as a documented Fedora development server with a stable identity on the home LAN.

## Capacity

10 story points.

Planning assumptions:

- Four to five focused sessions per week.
- Approximately five to seven engineering hours per week.
- Approximately fifteen to twenty engineering hours over the sprint.
- One primary story in progress at a time.

This is the first measured sprint. Future capacity must be adjusted from observed velocity rather than optimism.

## Committed stories

| Issue | Story | Points | Initial board status | Dependency |
|---|---|---:|---|---|
| #9 | Prepare Forge for Fedora installation | 2 | Ready | None |
| #10 | Install Fedora Server on Forge | 5 | Backlog | #9 |
| #11 | Configure stable hostname and LAN addressing | 3 | Backlog | #10 |

Total commitment: **10 story points**.

## Stretch candidate

Issue #12, Establish secure SSH access from the MacBook, may be pulled only after the committed stories are complete or clearly on track. It is not part of the 10-point commitment.

## Execution order

1. Start #9.
2. Move #9 to Review when all preparation evidence and required documentation are committed.
3. After #9 is accepted and closed, promote #10 to Ready.
4. After #10 is accepted and closed, promote #11 to Ready.
5. Preserve the one-primary-story work-in-progress limit.

## Sprint success criteria

Sprint 1 succeeds when:

- Forge is prepared safely for installation.
- Fedora Server is installed on the approved disk.
- The host boots reliably and has wired connectivity.
- The hostname and stable LAN addressing are configured.
- `forge.home.arpa` resolves from an approved LAN client.
- Required hardware, installation, and network documentation is committed.
- No undocumented destructive action or unresolved recovery gap remains.

## Review evidence

The sprint review should include:

- The running Forge host.
- Successful reboot evidence.
- LAN reachability by hostname.
- Links to documentation produced by the committed stories.
- Deviations, follow-up issues, and any ADR changes.

## Retrospective questions

- Was 10 points realistic for three weeks?
- How many focused sessions were actually available?
- Which work produced unexpected friction?
- Did the stories contain enough documentation guidance?
- Should the next sprint retain, reduce, or increase capacity?
