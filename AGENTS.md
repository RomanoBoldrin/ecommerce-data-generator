# Merchant Pulse Data Generator — Agent Operating Instructions

These instructions govern all coding agents working in this repository, including agents beginning in new conversation sessions without prior context.

---

## 1. Source of Truth

The hierarchy of authority in this project is strictly defined:

- **`AGENTS.md`**: Defines how coding agents must operate, verify work, and manage workflows.
- **`docs/IMPLEMENTATION.md`**: The authoritative project brief defining all functional requirements, scope constraints, and intended behavior.
- **`MILESTONES.md`**: Defines the approved sequential implementation roadmap, hard milestone boundaries, deliverables, and acceptance criteria.
- **`PROGRESS.md`**: The factual operational status log of the project (milestone states, verified commits, deviations, next actions).
- **`DECISIONS.md`**: The architectural and technical decision log distinguishing accepted decisions from proposals.
- **`reference/`**: The authoritative, read-only database contract (`schema.prisma` and database migrations).

*Note: Conversation history, previous model responses, or assumptions are not substitutes for these documents.*

---

## 2. Mandatory Preparation Before Every Work Session

Before making changes or executing work, every agent must perform the following preparation:

1. Read `AGENTS.md`.
2. Read `docs/IMPLEMENTATION.md`.
3. Read `MILESTONES.md`.
4. Read `PROGRESS.md`.
5. Read `DECISIONS.md` when relevant to proposed work.
6. Inspect the current Git status (`git status`) and recent commit history (`git log -n 5 --oneline`).
7. Verify that the actual repository state strictly agrees with the recorded status in `PROGRESS.md`.
8. Identify the currently approved milestone and its acceptance criteria.

**Rule:** If the repository state conflicts with the documentation, stop immediately and report the discrepancy to the user before proceeding.

---

## 3. Milestone Execution

- **Single Milestone Focus**: Work on exactly one milestone at a time.
- **User Approval Required**: Never begin a subsequent milestone without explicit user approval.
- **Hard Scope Boundaries**: Treat milestone boundaries as hard limits. Strictly adhere to the active milestone's objective, scope, exclusions, and dependencies.
- **No Scope Creep**: Do not silently expand scope, add speculative features, or implement future milestone components prematurely.
- **Requirement Integrity**: Do not reinterpret or weaken existing requirements to simplify implementation.
- **Escalation**: If a requirement is ambiguous, contradictory, or incompatible with the database schema, explain the issue and request clarification rather than guessing.
- **Roadmap Changes**: If execution reveals a necessary roadmap change, document the proposal and obtain approval before implementing out-of-scope work.

---

## 4. Implementation Discipline

- **Simplicity First**: Implement the minimum amount of complexity necessary to satisfy the requirements.
- **No Unnecessary Abstractions**: Do not introduce complex frameworks, extra services, daemons, digital twins, or speculative design patterns.
- **Preserve Established Decisions**: Follow decisions recorded in `DECISIONS.md`.
- **Schema Immutability**: Treat `reference/` as read-only. Do not modify the Prisma schema or migration files, and do not execute schema changes without explicit authorization.
- **No External LLM APIs**: Synthetic catalog and behavioral data must be generated programmatically and deterministically without external LLM dependencies.
- **Project Isolation**: Keep the Python generator completely independent of the Next.js application. Do not depend on Next.js source code or Prisma Client.
- **Data Integrity**: Enforce valid foreign keys, chronological event sequences, positive quantities, correct order totals (`total = subtotal - discount + shippingAmount`), and reproducible random seeds.

---

## 5. Git Discipline

- **Scope Isolation**: Keep changes strictly limited to the active milestone.
- **Logical Commits**: Commit changes in clean, descriptive, single-purpose commits associated with milestone completion.
- **Pre-commit Verification**: Run all relevant tests and verification checks prior to committing.
- **Diff Inspection**: Review the full `git diff` before committing.
- **Clean Repository**: Never commit credentials, `.env` files, virtual environments (`.venv`), temporary files, cache directories (`__pycache__`, `.pytest_cache`), generated datasets (`output/*`), or machine-specific artifacts.
- **History Preservation**: Never amend, rewrite, or delete previous commits without explicit authorization.
- **Atomic Documentation**: Update `PROGRESS.md` and `DECISIONS.md` as part of the milestone commit during handoff.

---

## 6. Milestone Completion and Handoff

A milestone is complete **only** when its acceptance criteria have been verified and its handoff has been documented.

Before declaring a milestone complete, the agent must:

1. Individually verify every acceptance criterion listed in `MILESTONES.md`.
2. Execute the verification commands and record actual output.
3. Review the complete Git diff.
4. Update `PROGRESS.md` with:
   - Completion date and commit hash.
   - Deliverables summary.
   - Specific verification results.
   - Documented deviations, limitations, or blockers.
   - The next action (awaiting approval for the next milestone).
5. Update `DECISIONS.md` if any technical decisions were finalized.
6. Commit all changes including documentation updates.
7. Present a concise handoff summary to the user and await explicit approval before proceeding.

---

## 7. Resuming After Interruption or Context Loss

When resuming work in a new conversation or after an interruption:

1. Re-read `AGENTS.md`, `docs/IMPLEMENTATION.md`, `MILESTONES.md`, `PROGRESS.md`, and `DECISIONS.md`.
2. Inspect `git status` and recent commit logs.
3. Compare the repository state against `PROGRESS.md`.
4. Verify the last completed milestone and commit hash.
5. Identify the active approved milestone.
6. Resume only the outstanding work within that milestone.

*Never assume a milestone is complete based on conversational claims. Always rely on factual repository state and project documentation.*

