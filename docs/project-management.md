# Project Forge Delivery System

## Purpose

Project Forge uses a lightweight Kanban system backed by GitHub Issues. The system should make the next useful action obvious without creating a second job devoted to managing the work.

## Work hierarchy

### Epics

Epics describe a platform outcome spanning multiple stories. Epic issue titles begin with `[EPIC]`.

### Stories

Stories describe independently verifiable engineering outcomes. Story issue titles begin with `[STORY]` and reference their parent epic in the body.

### Tasks

Tasks are checkboxes inside a story unless they need an independent discussion, owner, or acceptance test.

## Workflow

Use these statuses:

1. `Icebox`: intentionally deferred ideas that are not part of the active roadmap.
2. `Backlog`: valid work that is not yet executable or prioritized.
3. `Ready`: sufficiently understood, dependency-clear work that may be started next.
4. `In Progress`: actively being implemented. Limit to one primary story at a time.
5. `Review`: implementation is complete and awaiting validation, documentation review, or acceptance.
6. `Done`: acceptance criteria and the Definition of Done are satisfied; the GitHub issue is closed.

Until a GitHub Project board is created, status is recorded in each issue body. When the board exists, its Status field becomes authoritative and the body status may be removed.

## Prioritization

Work is selected in roadmap order unless a security issue, outage, data-loss risk, or blocking dependency requires an exception.

Priority guidance:

- `P0`: active security incident, data loss, or unavailable critical service.
- `P1`: blocks the current phase or removes a material operational risk.
- `P2`: normal roadmap work.
- `P3`: useful improvement that can wait.
- `Icebox`: intentionally deferred.

## Estimation

Use Fibonacci story points: `1, 2, 3, 5, 8, 13`.

Points describe relative complexity, uncertainty, and validation effort. They are not hour estimates. A story estimated above 8 points should normally be split before work begins.

## Work-in-progress limits

- One primary story in progress.
- One additional small operational or documentation item may be active when it does not distract from the primary story.
- Do not start a new story merely because the current one has become uncomfortable.

## Definition of Ready

A story is Ready when:

- The outcome is clear.
- Acceptance criteria are testable.
- Dependencies are known.
- Destructive actions and rollback expectations are documented.
- Required hardware, credentials, and access are available.
- The estimate is 8 points or less, or the reason for keeping it larger is documented.

## Definition of Done

A story is Done only when:

- Acceptance criteria pass.
- Security implications have been reviewed where applicable.
- Relevant documentation is updated.
- An ADR is added or updated when an architectural decision changed.
- Verification commands or evidence are recorded.
- Recovery or rollback behavior is documented for operational changes.
- Changes are committed to the correct repository.
- Follow-up work is captured as issues rather than hidden in notes.

## Planning rhythm

Project Forge uses continuous-flow Kanban with lightweight sprint-style rituals:

- Planning: choose the next Ready story and confirm its acceptance criteria.
- Check-in: report progress, blockers, discoveries, and scope changes.
- Review: demonstrate or verify the completed outcome.
- Retrospective: record what worked, what caused friction, and what should change.

No ceremony exists merely to imitate a larger organization.

## Initial active queue

The first active epic is `Platform Foundation`.

The initial Ready story is:

- Issue #9: Inventory Forge hardware and installation prerequisites.

Stories #10 through #15 remain in dependency order in the Backlog until their prerequisites are complete.

## GitHub Project board

Recommended board name: `Project Forge`.

Recommended fields:

- Status
- Epic
- Priority
- Story Points
- Roadmap Phase

Recommended views:

- Current Work: Ready, In Progress, Review.
- Roadmap: grouped by Epic and ordered by phase.
- Icebox: deferred capabilities such as later identity, local-model, and multi-node expansion work.

GitHub Issues remain the source of truth for requirements and acceptance criteria. The Project board is a view over that work, not a separate backlog.