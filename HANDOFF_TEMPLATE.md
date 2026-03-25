# Handoff & Implementation Summary Templates

**Version:** 1.0.0  
**Purpose:** Standardized formats for session planning (handoff) and session closure (implementation summary)  
**Companion:** CLAUDE_CODE_PROTOCOL.md (execution rules), AGENT_BEST_PRACTICES.md (standards), PIPELINE_AUDIT_TEMPLATE.md (audit-to-fix workflow)

---

## How These Documents Work Together

```
Claude Chat                              Claude Code
-----------                              -----------
Creates HANDOFF -----------------------> Reads HANDOFF
(what to build)                          Reads PROTOCOL (how to behave)
                                         Enters plan mode
                                         Proposes implementation
                                         Builds
                                         Creates IMPLEMENTATION_SUMMARY
                                         (what was built)
```

**Handoff = Intent** (session-scoped, disposable)  
**Protocol = Behavior** (system-scoped, stable)  
**Implementation Summary = Record** (session-scoped, permanent)

### Plan Mode Contract

Before writing any code, Claude Code must:

- Read the HANDOFF and PROTOCOL
- Propose an implementation that satisfies all success criteria
- Explicitly list files to be created or modified
- Surface conflicts between handoff intent and codebase reality
- Verify no work is planned outside defined non-goals
- Note cost implications if using external services
- Wait for approval before writing code

---

## Version Folder Structure

All session documents MUST be stored in version-scoped folders:

```
docs/
+-- v1.0/                                    # Version folder (lowercase v)
|   +-- V1.0_PROJECT_DOCUMENT.md             # Master project doc
|   +-- V1.0_SESSION_01_HANDOFF.md           # Session 1 handoff
|   +-- V1.0_SESSION_01_IMPLEMENTATION_SUMMARY.md
|   +-- V1.0_SESSION_02_HANDOFF.md           # Session 2 handoff
|   +-- V1.0_SESSION_02_IMPLEMENTATION_SUMMARY.md
+-- v1.5/                                    # Sessions restart at 01 per version
    +-- V1.5_PROJECT_DOCUMENT.md
    +-- V1.5_SESSION_01_HANDOFF.md
    +-- ...
```

**Naming Rules:**
- Folder: lowercase `v` + version number (e.g., `v1.0/`)
- Files: uppercase `V` + version + `_SESSION_` + zero-padded number (e.g., `V1.0_SESSION_01_`)
- Session numbers restart at `01` for each version

---

# PART 1: HANDOFF TEMPLATE

A handoff document answers one question: **"What must exist when this session is done?"**

It contains intent, not process. Claude Code derives the implementation approach.

---

## Template

```markdown
# Session [N]: [Feature Name] -- Handoff

---
Session: VX.X / SXX
Author: Claude Chat
Depends On: [Prior implementation summary, or "None"]
Supersedes: [Earlier handoff if this is a revision, or "None"]
Risk Level: [Low / Medium / High]
---

## Objective

[One sentence: what must be built or changed.]

## Context

[What already exists. Dependencies. Constraints that matter.
Include links to relevant files or prior sessions if helpful.]

## Requirements

[Functional and non-functional requirements. Be explicit, not verbose.]

### Functional

- [Requirement 1]
- [Requirement 2]

### Non-Functional (if applicable)

- [Performance, security, cost constraints]

## Success Criteria

[Observable, testable conditions. Written so a human can say "yes" or "no".]

- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Constraints

[Rules that MUST be obeyed. Hard limits. Edge cases to handle.]

- [Constraint 1]
- [Constraint 2]

## Non-Goals (Optional)

[Things intentionally NOT attempted, even if feasible. Prevents scope creep.]

- [Non-goal 1]
- [Non-goal 2]

## Testing & Verification

[How correctness will be validated. Commands to run, scenarios to test.]

```bash
# Example verification commands
python -m src.module test
python -m src.scheduler run --dry-run
```

## Files (Optional)

[Only include if scope must be explicitly constrained. Otherwise, let Claude Code
derive file changes in plan mode.]

| File | Action | Notes |
|------|--------|-------|
| `path/to/file.py` | Create | [Purpose] |
| `path/to/other.py` | Modify | [What changes] |

## Open Questions / Tradeoffs

[Anything intentionally undecided. Tradeoffs for Claude Code to surface in plan mode.]

- [Question or tradeoff 1]
- [Question or tradeoff 2]

## Authority

This document defines intent and success criteria.
The codebase defines current reality.
If conflicts exist, surface them in plan mode before implementation begins.
```

---

## Handoff Principles

| Principle | Guidance |
|-----------|----------|
| **Intent over implementation** | Say what, not how. Let Claude Code design the approach. |
| **Explicit over implicit** | Do not assume context. State dependencies clearly. |
| **Testable success criteria** | If you cannot verify it, you cannot ship it. |
| **Constraints are rules** | Things that MUST be obeyed. |
| **Non-goals prevent overreach** | Things intentionally NOT done, even if feasible. |
| **Small is better** | A handoff that requires scrolling is too long. |

---

## When to Include the Files Section

| Scenario | Include Files? |
|----------|----------------|
| Greenfield feature (new capability) | No -- let Claude derive |
| Refactor with strict boundaries | Yes -- prevent scope creep |
| Bug fix in known location | Yes -- focus the work |
| Migration or rename | Yes -- explicit mapping needed |
| Exploratory / R&D | No -- flexibility needed |

---

# PART 2: IMPLEMENTATION SUMMARY TEMPLATE

An implementation summary document answers one question: **"What was actually built?"**

Created by Claude Code at session close. Permanent record of what happened.

---

## Template

```markdown
# Session [N]: [Feature Name] -- Implementation Summary

---
Session: VX.X / SXX
Author: Claude Code
Handoff: VX.X_SESSION_XX_HANDOFF.md
Duration: [X hours / X minutes]
---

## Overview

[One paragraph: what was built, the approach taken, and the end result.]

## Deliverables

| Deliverable | Status | Notes |
|-------------|--------|-------|
| [Feature/component 1] | DONE | [Brief note] |
| [Feature/component 2] | PARTIAL | [What remains] |
| [Feature/component 3] | BLOCKED | [Blocker description] |

**Status values:** DONE | PARTIAL | SKIPPED | BLOCKED

If any deliverable is PARTIAL or BLOCKED, explain the blocker
and what is required to complete it in a future session.

## Files Created

| File | Purpose |
|------|---------|
| `path/to/file.py` | [What it does] |

## Files Modified

| File | Change |
|------|--------|
| `path/to/file.py` | [What changed and why] |

## Technical Decisions

[Decisions made during implementation that were not specified in the handoff.]

| Decision | Choice | Rationale |
|----------|--------|-----------|
| [What was decided] | [What was chosen] | [Why] |

## Testing Results

[Test output or summary. Include command run and results.]

```text
$ python -m pytest tests/
========================= X passed, Y failed =========================
```

## Deviations from Handoff

[Anything that differed from the original plan.]

| Handoff Said | Actual | Reason |
|--------------|--------|--------|
| [Original plan] | [What changed] | [Why] |

If none: "Implemented as specified."

## Known Issues / Technical Debt

| Issue | Severity | Recommendation |
|-------|----------|----------------|
| [Description] | Low/Med/High | [Fix in session N or accept] |

If none: "No known issues."

## Cost Impact (if applicable)

| Service | Usage | Cost |
|---------|-------|------|
| [Service] | [X calls] | $X.XX |

Ongoing impact: [$/month change or "None"]

## Next Session Notes

[What future sessions should know. Dependencies created, patterns established,
landmines to avoid, or suggested next steps.]
```

---

## Implementation Summary Principles

| Principle | Guidance |
|-----------|----------|
| **Accuracy over optimism** | Report what happened, not what was intended. |
| **Deviations are normal** | Document them without judgment. |
| **PARTIAL/BLOCKED need explanation** | Always state what remains and why. |
| **Technical debt is acceptable** | Just make it visible. |
| **Next session notes are gold** | Future-you will thank present-you. |

---

# PART 3: QUICK REFERENCE

## Handoff Checklist (for Claude Chat)

Before handing off to Claude Code:

- [ ] Metadata block is complete (Session, Depends On, Risk Level)
- [ ] Objective is one clear sentence
- [ ] Context includes all dependencies
- [ ] Requirements are explicit and testable
- [ ] Success criteria are yes/no verifiable
- [ ] Constraints state rules that MUST be obeyed
- [ ] Non-goals clarify what is intentionally out of scope (if needed)
- [ ] Testing commands are provided
- [ ] Open questions are surfaced (not hidden)

## Implementation Summary Checklist (for Claude Code)

Before closing a session:

- [ ] Metadata block is complete (Session, Handoff, Duration)
- [ ] All deliverables have status (DONE/PARTIAL/SKIPPED/BLOCKED)
- [ ] PARTIAL/BLOCKED items explain what remains and why
- [ ] All files created/modified are listed
- [ ] Technical decisions are documented
- [ ] Test results are included
- [ ] Deviations are explained (or "Implemented as specified")
- [ ] Known issues are captured (or "No known issues")
- [ ] Next session notes are written

---

# PART 4: EXAMPLES

## Example Handoff (Good)

```markdown
# Session 5: Scheduler Safeguards -- Handoff

---
Session: V1.0 / S05
Author: Claude Chat
Depends On: V1.0_SESSION_04_IMPLEMENTATION_SUMMARY.md
Supersedes: None
Risk Level: Medium
---

## Objective

Add safeguards to prevent runaway execution in the scheduler.

## Context

The scheduler (`src/scheduler.py`) currently has no protection against:
- Running multiple times after a single success
- Infinite retry loops on persistent failures
- Runaway execution from misconfigured launchd

Related: SCHEDULER_TEMPLATE.md documents the target patterns.

## Requirements

### Functional

- Skip execution if already succeeded today
- Limit retries to 3 per day
- Emergency brake at 10 total runs per day
- All skips must exit 0 (prevent launchd restart)

### Non-Functional

- No additional dependencies
- State changes must be atomic (no partial updates)

## Success Criteria

- [ ] `scheduler run` after success exits 0 with "already succeeded" message
- [ ] `scheduler run` after 3 failures exits 0 with "max retries" message
- [ ] `scheduler run` after 10 runs exits 0 with "emergency brake" message
- [ ] `scheduler status` shows safeguard state

## Constraints

[Rules that MUST be obeyed.]

- Do not modify the launchd plist
- Do not change the lock mechanism
- Preserve backward compatibility with existing state.json

## Non-Goals

[Things intentionally NOT attempted, even if feasible.]

- Alerting/notification system (future session)
- Configuration file for safeguard limits (hardcode for now)
- Web dashboard for status (out of scope)

## Testing & Verification

```bash
python -m src.scheduler run --dry-run
python -m src.scheduler status
# Manually verify state.json after runs
```

## Open Questions / Tradeoffs

- Should emergency brake send a notification? (Recommend: yes, but defer to Non-Goals)
- Should --force bypass all safeguards or just "already succeeded"? (Recommend: all)

## Authority

This document defines intent and success criteria.
The codebase defines current reality.
If conflicts exist, surface them in plan mode before implementation begins.
```

---

## Example Handoff (Bad -- Too Implementation-Focused)

```markdown
# Session 5: Scheduler Safeguards -- Handoff

## Objective

Add safeguards to the scheduler.

## Requirements

1. Add `last_success_date` field to SchedulerState dataclass
2. Add `retries_today` field to SchedulerState dataclass  
3. Add `runs_today` field to SchedulerState dataclass
4. Create `already_succeeded_today()` method that checks if last_success_date equals today
5. Create `can_retry()` method that returns True if retries_today < 3
6. Create `check_emergency_brake()` method that returns True if runs_today >= 10
7. Modify `run()` function to call these methods before executing
8. Update `cmd_status()` to display safeguard information
...
```

**Why it is bad:** Dictates implementation instead of intent. Does not let Claude Code
design the solution. No success criteria. No constraints. No non-goals. No metadata.

---

# PART 5: METADATA REFERENCE

## Handoff Metadata Fields

| Field | Required | Description |
|-------|----------|-------------|
| Session | Yes | Version and session number (e.g., V1.0 / S05) |
| Author | Yes | Who created this handoff (Claude Chat or human) |
| Depends On | Yes | Prior implementation summary this builds on, or "None" |
| Supersedes | No | Earlier handoff if this is a revision, or "None" |
| Risk Level | Yes | Low / Medium / High -- subjective but useful |

## Implementation Summary Metadata Fields

| Field | Required | Description |
|-------|----------|-------------|
| Session | Yes | Version and session number (e.g., V1.0 / S05) |
| Author | Yes | Who created this summary (Claude Code) |
| Handoff | Yes | Reference to the handoff document |
| Duration | Yes | How long the session took |

## Risk Level Guidelines

| Level | Criteria |
|-------|----------|
| **Low** | Isolated change, well-understood area, easy to revert |
| **Medium** | Touches multiple files, some uncertainty, moderate blast radius |
| **High** | Core system change, new patterns, hard to revert, external dependencies |

---

## Document Metadata

| Field | Value |
|-------|-------|
| Template Version | 1.0.0 |
| Created | 2026-02-03 |
| Companion Docs | CLAUDE_CODE_PROTOCOL.md, AGENT_BEST_PRACTICES.md, PIPELINE_AUDIT_TEMPLATE.md |
