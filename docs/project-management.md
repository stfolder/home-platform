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

The GitHub Project board Status field is authoritative. Issue bodies may include sprint metadata and requirements, but must not be used as a substitute for moving cards on the board.

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

Working calibration:

- `1`: very small change or verification.
- `2`: one focused session or low-risk preparation story.
- `3`: roughly two focused sessions or moderate validation effort.
- `5`: several sessions with meaningful implementation or uncertainty.
- `8`: substantial work that should be reviewed carefully for splitting.
- `13`: too large for normal sprint commitment; groom again.

## Sprint cadence and capacity

Project Forge uses three-week sprints inside the continuous-flow Kanban system.

The initial planning assumption is:

- Four to five focused sessions per week.
- Approximately five to seven engineering hours per week.
- Approximately fifteen to twenty engineering hours per sprint.
- Initial sprint capacity: 10 story points.

Capacity is empirical, not aspirational. After each sprint, completed points, interruptions, and friction are reviewed before changing the next sprint commitment.

A stretch story may be pulled only when all committed stories are complete or clearly on track and the work-in-progress limit remains respected.

## Work-in-progress limits

- One primary story in progress.
- One additional small operational or documentation item may be active when it does not distract from the primary story.
- Do not start a new story merely because the current one has become uncomfortable.

## Definition of Ready

A story is Ready when:

- The outcome and value are clear.
- Acceptance criteria are testable.
- Dependencies are known.
- Documentation deliverables are explicitly defined where applicable.
- Destructive actions and rollback expectations are documented.
- Required hardware, credentials, and access are available.
- An estimate is assigned.
- The Definition of Done is clear.
- No major unanswered technical question blocks execution.
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

Project Forge uses continuous-flow Kanban with lightweight sprint rituals:

- Sprint planning: set one outcome-oriented sprint goal and commit only Ready or dependency-ordered stories within capacity.
- Check-in: report progress, blockers, discoveries, and scope changes.
- Grooming: clarify future stories before promoting them to Ready.
- Review: demonstrate or verify the completed sprint outcome.
- Retrospective: record what worked, what caused friction, and what should change.

No ceremony exists merely to imitate a larger organization.

## Sprint 1 baseline

Sprint 1 is documented in [`docs/sprints/sprint-001.md`](sprints/sprint-001.md).

- Length: three weeks.
- Capacity: 10 story points.
- Committed stories: #9, #10, and #11.
- Stretch candidate: #12.
- First executable story: #9.

Stories may be committed to a sprint while remaining in Backlog until their dependencies are satisfied. Only dependency-clear, fully groomed work belongs in Ready.

## GitHub Project board

Board name: `Project Forge`.

Recommended fields:

- Status.
- Epic.
- Priority.
- Story Points.
- Sprint.
- Roadmap Phase.

Recommended views:

- Current Sprint: filtered to the active sprint and grouped by Status.
- Current Work: Ready, In Progress, and Review.
- Roadmap: grouped by Epic and ordered by phase.
- Icebox: deferred capabilities such as later identity, local-model, and multi-node expansion work.

GitHub Issues remain the source of truth for requirements and acceptance criteria. The Project board is the authoritative view for workflow status and sprint assignment.