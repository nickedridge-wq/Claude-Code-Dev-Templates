# Claude Code Execution Protocol

**Version:** 1.0.0  
**Purpose:** Defines how Claude Code behaves during implementation sessions  
**Companion:** HANDOFF_TEMPLATE.md (session specs), AGENT_BEST_PRACTICES.md (architecture principles), PIPELINE_AUDIT_TEMPLATE.md (audit framework)

---

## Protocol Authority

This document defines execution behavior for Claude Code across all sessions.

- **Handoffs** define WHAT to build (session-scoped intent)
- **This protocol** defines HOW to behave (system-scoped rules)
- **The codebase** defines CURRENT REALITY

If conflicts exist between handoff and codebase, surface them in plan mode before implementation.

This protocol applies to every session unless explicitly overridden by the user.

---

# PART 1: SESSION LIFECYCLE

## The Four Phases

Every session follows this lifecycle:

```
+------------------------------------------------------------------+
|                                                                  |
|  1. PLAN          2. BUILD          3. VERIFY       4. CLOSE     |
|                                                                  |
|  Read handoff     Implement         Test            Impl Summary |
|  Read protocol    Track progress    Fix issues      Commit       |
|  Propose plan     Update checklist  Iterate         Update docs  |
|  Wait for OK                                                     |
|                                                                  |
+------------------------------------------------------------------+
```

### Phase 1: PLAN

**Before writing any code, Claude Code must:**

1. Read the handoff document completely
2. Read this protocol (or confirm familiarity)
3. Read Claude.md
4. Read README.md
5. Identify the current version and session number
6. Verify the `docs/vX.X/` folder exists (create if needed)
7. **Save the handoff to `docs/vX.X/VX.X_SESSION_XX_HANDOFF.md`**
8. Propose an implementation plan that includes:
   - Files to create or modify
   - Approach and rationale
   - Risks or conflicts with existing code
   - Questions or tradeoffs to resolve
   - **Non-goals verification** (confirm no work outside defined scope)
   - **Cost implications** (if using external APIs/services)
9. **Wait for user approval before proceeding**

**Plan Mode Checklist:**

- [ ] Handoff saved to `docs/vX.X/`
- [ ] Files to create/modify identified
- [ ] Approach defined
- [ ] Risks and conflicts surfaced
- [ ] Non-goals reviewed (no out-of-scope work planned)
- [ ] Cost implications noted (if applicable)
- [ ] Questions or tradeoffs listed
- [ ] User approval received

**Plan mode output format:**

```markdown
## Implementation Plan

### Files to Create
- `path/to/new_file.py` - [purpose]

### Files to Modify
- `path/to/existing.py` - [what changes]

### Dependencies / External Services
[Any APIs, services, libraries, or modules this implementation relies on.
Include version requirements if relevant. "None" if purely internal.]

### Approach
[How the implementation will satisfy success criteria]

### Risks / Conflicts
[Any issues discovered between handoff intent and codebase reality]

### Non-Goals Verification
[Confirm: "All planned work is within scope. No non-goal items will be implemented."
Or flag: "Warning: X appears to touch non-goal Y -- clarification needed."]

### Cost Implications
[Estimated API calls, service costs, or "No external cost impact"]

### Questions
[Anything needing clarification before proceeding]

---
Ready to proceed? (yes/no)
```

### Phase 2: BUILD

**During implementation, Claude Code must:**

1. Create a task checklist from the handoff (see Task Tracking below)
2. Implement code to satisfy success criteria
3. Update the checklist after completing each major task
4. Print progress updates at meaningful intervals
5. Surface blockers immediately rather than working around them

### Phase 3: VERIFY

**Before closing, Claude Code must:**

1. Run all specified tests
2. Verify each success criterion is met
3. Document any failures or partial completions
4. Get user confirmation that work is acceptable

### Phase 4: CLOSE

**To close a session, Claude Code must complete ALL of these:**

1. Verify handoff was saved to `docs/vX.X/VX.X_SESSION_XX_HANDOFF.md`
2. Write the implementation summary (`VX.X_SESSION_XX_IMPLEMENTATION_SUMMARY.md`)
3. Commit all changes with proper message format
4. Update the master project document
5. Update Claude.md to reflect current status
6. Update README.md to reflect current status
5. Report final status to user

**A session is not complete until all close tasks are done.**

---

# PART 2: TASK TRACKING

## Creating the Task Checklist

At the START of every session, Claude Code must:

1. Parse the handoff for all deliverables and success criteria
2. Create a numbered checklist
3. Print the checklist to confirm scope with the user

**Task list format:**

```markdown
## VX.X Session XX Task List

### From Handoff
- [ ] 1. [First deliverable/requirement]
- [ ] 2. [Second deliverable/requirement]
- [ ] 3. [Third deliverable/requirement]

### Success Criteria
- [ ] 4. [Criterion 1 - testable]
- [ ] 5. [Criterion 2 - testable]

### Standard Close
- [ ] 6. Verify handoff saved to docs/vX.X/
- [ ] 7. Write VX.X_SESSION_XX_IMPLEMENTATION_SUMMARY.md
- [ ] 8. Commit all changes
- [ ] 9. Update VX.X_PROJECT_DOCUMENT.md
- [ ] 10. Update Claude.md
- [ ] 11. Update README.md

---
Progress: 0/9 tasks complete
```

## Updating the Checklist

Throughout the session, Claude Code must:

1. Check off each task as it is completed
2. Print the updated checklist after each major task
3. Note any blockers or scope changes with `[!]` marker

**Example progress update:**

```markdown
## V1.0 Session 05 Task List

### From Handoff
- [x] 1. Add retry limits to scheduler
- [x] 2. Add emergency brake safeguard
- [ ] 3. Update status command
- [!] 4. Add notification on brake -- BLOCKED: needs notification system first

### Success Criteria
- [x] 5. scheduler run exits 0 after success
- [ ] 6. scheduler status shows safeguard state

### Standard Close
- [x] 7. Verify handoff saved to docs/v1.0/
- [ ] 8. Write V1.0_SESSION_05_IMPLEMENTATION_SUMMARY.md
- [ ] 9. Commit all changes
- [ ] 10. Update V1.0_PROJECT_DOCUMENT.md
- [ ] 11. Update Claude.md
- [ ] 12. Update README.md

---
Progress: 4/10 tasks complete (1 blocked)
```

## Task Tracking Rules

| Rule | Rationale |
|------|-----------|
| Always create checklist before writing code | Confirms scope with user |
| Tasks must map 1:1 to handoff deliverables | Prevents drift |
| Print updated list after each major task | Visibility |
| Mark blocked items with `[!]` and explain | Transparency |
| Final checklist must show all items addressed | Audit trail |

---

# PART 3: DEFINITION OF DONE

## Session Completion Criteria

**A session is DONE when ALL of the following are true:**

- [ ] Handoff saved to `docs/vX.X/VX.X_SESSION_XX_HANDOFF.md`
- [ ] All tasks from handoff checklist are complete (or marked `[!]` with explanation)
- [ ] All tests passing (or failures explicitly documented)
- [ ] No stubs remain (or documented as intentional with TODO)
- [ ] Code executes without runtime errors
- [ ] Agent contract invariants still hold (if applicable)
- [ ] Implementation summary written to `docs/vX.X/VX.X_SESSION_XX_IMPLEMENTATION_SUMMARY.md`
- [ ] Changes committed with proper message format
- [ ] Project document updated
- [ ] Claude.md updated
- [ ] README.md updated

**If any box cannot be checked, the session is not complete.**

## Partial Completion

If a session cannot be fully completed:

1. Mark incomplete items as PARTIAL or BLOCKED in the summary
2. Explain what remains and why
3. Document what is required to complete in a future session
4. Still commit working code and write implementation summary
5. Update project document with accurate status

Partial completion is acceptable. Undocumented partial completion is not.

---

# PART 4: VERSION CONTROL

## Commit Requirements

Claude Code must commit at the end of every session.

**Commits must be:**

| Property | Requirement |
|----------|-------------|
| **Atomic** | One logical change per commit |
| **Reversible** | Safe to revert without breaking other features |
| **Build-safe** | Code executes and tests pass |
| **Intentional** | Purpose clear from message alone |

## Commit Message Format

```
<type>(scope): short summary

- What was added/changed
- What was added/changed

Refs: docs/vX.X/VX.X_SESSION_XX_*
```

**Allowed types:**

| Type | When to Use |
|------|-------------|
| `feat` | New behavior or capability |
| `fix` | Bug or defect fix |
| `refactor` | Code restructure, no behavior change |
| `test` | Tests only |
| `docs` | Documentation only |
| `chore` | Tooling, config, cleanup |
| `release` | Version release (tags only) |

**Example:**

```
feat(scheduler): add runaway execution safeguards

- Add retry limits (max 3/day)
- Add emergency brake (max 10 runs/day)
- Add already-succeeded-today check
- Update state tracking with daily counters

Refs: docs/v1.0/V1.0_SESSION_05_*
```

## Commit Gates (Non-Negotiable)

Before committing, Claude Code MUST verify:

- [ ] Code executes without runtime errors
- [ ] Tests pass (or failures documented in summary)
- [ ] No secrets or `.env` files included
- [ ] No generated artifacts (binaries, build outputs, node_modules)
- [ ] No temporary debug code or console.log statements
- [ ] Commit message follows format

**If any gate fails, do not commit. Fix or document first.**

## Prohibited Commit Content

Never commit:

- API keys, tokens, or credentials
- `.env` files
- Generated artifacts (binaries, `__pycache__`, `node_modules`)
- Temporary debug code
- Unvalidated experimental code
- Large binary files (images, videos, data files)

---

# PART 5: PROJECT DOCUMENT UPDATES

## When to Update

The master project document (`docs/vX.X/VX.X_PROJECT_DOCUMENT.md`) must be updated at the end of every session.

## What to Update

| Section | What to Update |
|---------|----------------|
| **Current State** | Component status indicators (`[ ]` -> `[WIP]` -> `[DONE]`) |
| **Key Metrics** | Actual values if changed |
| **Session Log** | Add row for completed session |
| **Key Pivots & Decisions** | Any decisions made during implementation |
| **Project Structure** | New files if structure changed |
| **External Services & Costs** | New services or cost changes |
| **Risk Register** | New risks discovered, retire resolved risks |
| **Roadmap** | Update completion status |
| **Lessons Learned** | Insights from this session |

## Update Process

1. Read current project document
2. Update only sections that changed
3. Preserve all other content exactly
4. Add session log entry

**Session log entry format:**

```markdown
| [N] | YYYY-MM-DD | [Feature Name] | [What was delivered] | [Duration] |
```

**Key pivots entry format:**

```markdown
| YYYY-MM-DD | [Decision] | [Options] | **[Choice]** | [Rationale] |
```

---

# PART 6: FILE ORGANIZATION

## Standard Paths

| Purpose | Path |
|---------|------|
| Session handoffs | `docs/vX.X/VX.X_SESSION_XX_HANDOFF.md` |
| Implementation summaries | `docs/vX.X/VX.X_SESSION_XX_IMPLEMENTATION_SUMMARY.md` |
| Project document | `docs/vX.X/VX.X_PROJECT_DOCUMENT.md` |
| Claude.md | `docs/Claude.md` |
| README.md | `docs/README.md` |
| Source code | `src/` |
| Tests | `tests/` |
| Configuration | `config/` |
| Runtime data | `data/` (git-ignored) |
| Scripts | `scripts/` |

## Version Folder Rules

- Each major/minor version gets its own folder: `docs/v1.0/`, `docs/v1.5/`, `docs/v2.0/`
- Session numbers restart at 01 for each version
- Project document lives in the version folder
- Claude.md and README.md live in main docs folder
- Never modify documents in older version folders

## Creating New Version Folders

When starting a new version:

1. Create `docs/vX.X/` folder
2. Copy and update project document from previous version
3. Start session numbering at 01
4. Reference previous version's final implementation summary as "Depends On" in first handoff

---

# PART 7: ERROR HANDLING

## When Things Go Wrong

| Situation | Action |
|-----------|--------|
| Test failures | Document in summary, mark deliverable as PARTIAL |
| Blocker discovered | Stop, surface to user, mark task as `[!]` |
| Handoff unclear | Ask for clarification before proceeding |
| Scope creep detected | Stop, confirm with user if new work is in scope |
| Conflict with codebase | Surface in plan mode, propose resolution |
| Cannot complete session | Write summary with PARTIAL/BLOCKED status, still commit working code |

## Surfacing Problems

When a problem is discovered:

1. **Stop** - Do not work around silently
2. **Describe** - State what the problem is
3. **Impact** - Explain what it affects
4. **Options** - Propose solutions if possible
5. **Wait** - Get user input before proceeding

**Never silently work around a problem. Transparency is mandatory.**

---

# PART 8: QUICK REFERENCE

## Session Start Checklist

```markdown
- [ ] Read Claude.md
- [ ] Read README.md
- [ ] Read handoff document
- [ ] Identify version (X.X) and session number (XX)
- [ ] Verify docs/vX.X/ folder exists
- [ ] Save handoff to docs/vX.X/VX.X_SESSION_XX_HANDOFF.md
- [ ] Create implementation plan
- [ ] Wait for user approval
- [ ] Create task checklist
- [ ] Print checklist to confirm scope
```

## Session Close Checklist

```markdown
- [ ] All tasks addressed (complete or marked [!])
- [ ] Tests run and results documented
- [ ] Handoff saved to docs/vX.X/ (verify)
- [ ] Implementation summary written to docs/vX.X/
- [ ] Changes committed with proper message
- [ ] Project document updated
- [ ] Claude.md updated
- [ ] README.md updated
- [ ] Final status reported to user
```

## Commit Checklist

```markdown
- [ ] Code executes without errors
- [ ] Tests pass (or failures documented)
- [ ] No secrets included
- [ ] No generated artifacts
- [ ] Message follows format
- [ ] Refs session docs
```

---

# PART 9: PROTOCOL VIOLATIONS

## What Constitutes a Violation

| Violation | Severity |
|-----------|----------|
| Committing secrets | **Critical** - immediate remediation required |
| Skipping plan mode | High - may require rework |
| Not writing implementation summary | High - breaks audit trail |
| Silent scope changes | High - erodes trust |
| Not updating project doc | Medium - causes drift |
| Incomplete task tracking | Medium - reduces visibility |
| Poor commit messages | Low - fixable later |

## Recovery from Violations

**Critical (secrets committed):**
1. Immediately notify user
2. Rotate compromised credentials
3. Rewrite git history if possible
4. Document incident

**High:**
1. Stop current work
2. Notify user of violation
3. Correct before proceeding
4. Document in summary

**Medium/Low:**
1. Correct in current session
2. Note in summary as deviation

---

## Document Metadata

| Field | Value |
|-------|-------|
| Protocol Version | 1.0.0 |
| Created | 2026-02-03 |
| Companion Docs | HANDOFF_TEMPLATE.md, AGENT_BEST_PRACTICES.md, PIPELINE_AUDIT_TEMPLATE.md |

---

## Protocol Acknowledgment

Claude Code should acknowledge this protocol at session start:

```
Protocol acknowledged: CLAUDE_CODE_PROTOCOL v1.0.0
Session: VX.X / SXX
Ready to review handoff.
```

This confirms the protocol has been read and will be followed.
