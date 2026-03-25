# Pipeline Audit Template

**Version:** 1.0.0  
**Purpose:** Standardized framework for end-to-end code review audits of automated pipelines  
**Origin:** Derived from production pipeline audits across multiple domains  
**Companion:** HANDOFF_TEMPLATE.md, CLAUDE_CODE_PROTOCOL.md, AGENT_BEST_PRACTICES.md

---

## When to Use This Template

Use this template when:
- A pipeline has been built but never audited
- A related pipeline had bugs and you want to check for the same patterns
- Before moving from paper/simulation mode to live/production mode
- After significant refactoring to verify no regressions
- As a periodic health check (quarterly recommended)

**An audit is read-only. It produces a document, not code changes.**

---

## Using This With Claude Code

This template works as a direct instruction set for Claude Code. Upload it alongside your codebase and tell Claude Code to use it as the foundation for a pipeline audit.

**Workflow:**

```
1. Upload this template + your codebase to Claude Code
2. Prompt: "Use PIPELINE_AUDIT_TEMPLATE.md as the foundation to run a process audit on [pipeline/project]"
3. Claude Code outputs a structured audit report following Part 1
4. Take the audit report to a new Claude session
5. Use Part 6 to convert findings into a bug-fix handoff
6. Hand the bug-fix document back to Claude Code for implementation
```

**Why separate the audit from the fixes:** Claude Code produces better audits when it knows it won't be asked to fix anything in the same session. It focuses on finding problems rather than pre-filtering to things it knows how to fix. The fix session gets a clean handoff with full context.

---

## How This Template Works

```
1. Define scope          → What pipeline stages exist?
2. Read every file       → Stage-by-stage code review
3. Check bug patterns    → Apply the 10 universal patterns
4. Cross-reference config → Are config values actually used?
5. Catalog findings      → Severity + file:line + impact + fix
6. Prioritize fixes      → Ordered by severity and blast radius
```

The auditor walks through the pipeline in execution order, checking each stage against known bug patterns. Findings are cataloged in a standardized format so they can be directly converted into a bug-fix session handoff.

---

## Part 1: Audit Document Structure

Every audit document should follow this structure exactly. When using this template as instructions for Claude Code, the output should match this format.

```markdown
# [Project Name] — [Pipeline Name] Pipeline Audit

**Date:** YYYY-MM-DD
**Scope:** [What is being audited — files, directories, config]
**Method:** Code review of [file count] source files + [config count] config files
**Reference:** [Prior audits, specs, or architecture docs]

---

## Executive Summary

[2-3 sentences: overall health, bug count by severity, biggest finding.
State clearly whether the pipeline is safe for production use.]

### Bug Severity Summary

| Severity | Count | Impact |
|----------|-------|--------|
| **CRITICAL** | X | [One-line summary] |
| **HIGH** | X | [One-line summary] |
| **MEDIUM** | X | [One-line summary] |
| **LOW** | X | [One-line summary] |

---

## Stage-by-Stage Review

[One section per pipeline stage, in execution order.
Each section reviews the relevant source files, checks for
bug patterns, and catalogs findings.]

### Stage N: [Stage Name]

**Files:** `path/to/file.py` (X lines)

[Description of what this stage does and how it works.]

[For each finding:]

> **ISSUE [ID] [SEVERITY]: [Short description].**
>
> **Location:** `file.py:line`
> ```python
> [Relevant code snippet]
> ```
>
> **Impact:** [What goes wrong and when]
> **Fix:** [Recommended fix]

[If no bugs found: "**No bugs found in [stage name].**"]

---

## Cross-Cutting Issues

[Issues that span multiple stages or are systemic patterns.]

### [Pattern Name]

| Location | Expression | Severity | Impact |
|----------|-----------|----------|--------|
| `file.py:line` | [code] | [sev] | [impact] |

---

## Bug Pattern Checklist

[Results of checking all 10 universal patterns. Each marked
"Found" or "Not found" with details.]

---

## Complete Bug List

[Single table, ordered by severity, with all findings.]

### CRITICAL

| ID | Bug | File:Line | Impact | Fix |
|----|-----|-----------|--------|-----|

### HIGH

| ID | Bug | File:Line | Impact | Fix |
|----|-----|-----------|--------|-----|

[...continue for MEDIUM, LOW]

---

## Recommended Fix Priority

### Phase A: [Immediate — blocks production safety]
### Phase B: [Soon — significant quality impact]
### Phase C: [Harden — defensive improvements]
### Phase D: [Monitor — no code changes needed]

---
```

---

## Part 2: Severity Classification

Use this scale consistently across all audits. The key distinction is **impact on output correctness**, not code aesthetics.

| Severity | Definition | Examples |
|----------|------------|---------|
| **CRITICAL** | Produces incorrect outputs that would cause financial loss, data corruption, or safety issues. Must fix before production use. | Wrong calculations used for decisions, data written to wrong location, security vulnerability, infinite loops consuming resources |
| **HIGH** | Significant accuracy or reliability impact. Outputs are degraded but not dangerous. | Valid inputs silently dropped, metrics unreliable, business logic wrong for one code path, systematic bias in outputs |
| **MEDIUM** | Feature broken, dead code that's a maintenance risk, or fragile patterns that will break under edge cases. | Dead code that overwrites correct code, hardcoded values that should be config-driven, heuristics that fail at boundaries |
| **LOW** | Code cleanliness, redundant logic, cosmetic issues. No functional impact with current inputs. | Redundant conditions, unused config, defensive patterns that never trigger, magic numbers that are documented |

### Severity Decision Tree

```
Does the bug produce WRONG outputs?
├── YES → Does it affect financial/safety decisions?
│   ├── YES → CRITICAL
│   └── NO  → HIGH
└── NO  → Does it SKIP valid inputs or BREAK under edge cases?
    ├── YES → MEDIUM
    └── NO  → LOW
```

---

## Part 3: The 10 Universal Bug Patterns

These patterns were discovered across multiple production pipeline audits. Check every one during any pipeline audit. They are language-general but examples use Python.

---

### Pattern 1: Numeric Truthiness

**What:** Using language truthiness checks on numeric values where `0` is a valid input.

**Why it's dangerous:** In Python, `0`, `0.0`, `""`, `[]`, `{}`, `None` are all falsy. If `0` is a valid value (price, temperature, count, index), truthiness checks silently treat it as missing.

**Search for:**
```python
# DANGEROUS — 0 is treated as falsy
value = data.get("price") or default
if value and other_value:
value or fallback
not all([..., value, ...])
```

**Fix:**
```python
# SAFE — explicit None check
value = data.get("price")
if value is not None:
value if value is not None else fallback
any(v is None for v in [...])
```

**Severity guide:**
- Truthiness on a value that controls output (price, probability, score) → CRITICAL/HIGH
- Truthiness on a value that controls filtering (skip vs include) → HIGH/MEDIUM
- Truthiness on a value guaranteed non-zero by upstream logic → LOW

---

### Pattern 2: Asymmetric Input Handling

**What:** Using the same value for both sides of a binary or directional operation when each side has a different basis.

**Why it's dangerous:** Many operations have two sides that aren't symmetric. Using one value for both means one side's logic is silently wrong — often producing zero-value results or inverted calculations.

**Search for:**
- Input values stored without checking direction/side
- Sizing or allocation formulas receiving the same input regardless of direction
- P&L or cost formulas assuming symmetric cost basis
- Buy vs. sell, request vs. response, inbound vs. outbound using shared values

**Examples:**
```python
# DANGEROUS — bid ≠ ask, but using one price for both
cost = quantity * price  # Which price? Buy or sell?

# DANGEROUS — same timeout for request and response
timeout = config["api_timeout"]  # Sending may need 5s, receiving may need 30s

# DANGEROUS — same rate limit for reads and writes
if requests_this_minute > limit:  # Reads are cheap, writes are expensive
```

**Fix:** At every point where a value is used, check: "Does this value need to differ by side/direction?" If yes, branch explicitly.

**Severity guide:**
- Wrong value in a sizing or allocation formula → CRITICAL (outputs unsizeable)
- Wrong value in cost or performance calculation → HIGH (metrics unreliable)
- Wrong value in display/logging → LOW

---

### Pattern 3: Config Defined but Unused

**What:** Configuration values defined in YAML/JSON/env but never loaded or enforced in code.

**Why it's dangerous:** Creates a false sense of safety. Operators think thresholds are enforced because they're in config, but the code never reads them.

**Search for:**
- Every key in every config file → grep for its usage in source code
- Config sections with no corresponding loader
- Default values in code that shadow config values

**Fix:** Either wire the config value into the code path, or remove it from config with a comment explaining why.

**Severity guide:**
- Safety threshold defined but unenforced (min_edge, max_loss) → HIGH
- Feature flag defined but unchecked → MEDIUM
- Metadata or documentation-only config → LOW (not a bug)

---

### Pattern 4: Missing Deduplication

**What:** Pipeline can produce duplicate outputs for the same input across multiple runs.

**Why it's dangerous:** Multiple runs on the same day (retries, manual triggers, scheduler overlap) generate duplicate records. Downstream aggregation double-counts them, inflating metrics or triggering duplicate actions.

**Search for:**
- Output tables without unique constraints on natural keys
- Pipeline entry points without "already processed" checks
- Idempotency gaps in the write path

**Fix:** Add a dedup guard at the point of output creation. Check for existing records with the same natural key (entity + date + type) before inserting. Decide whether re-processing should update existing records or skip them.

**Severity guide:**
- Duplicate records causing financial or operational errors → CRITICAL
- Duplicate records inflating metrics or counts → HIGH
- Duplicate log entries → LOW

---

### Pattern 5: Missing Expiry/Staleness

**What:** Records that should have a finite lifetime accumulate indefinitely.

**Why it's dangerous:** Stale records, cache entries, or pending jobs remain in an active state forever if their completion condition never fires. This bloats queries, inflates counts, and can cause resources to be consumed attempting to process ancient records.

**Search for:**
- Queries like `WHERE status = 'pending'` with no date filter
- Records that depend on an external event or API response that may never arrive
- Cache entries with no TTL
- Job queues with no max-age policy

**Fix:** Add expiry logic: records past a configurable age are marked EXPIRED before resolution is attempted. Expired records should be excluded from active metrics.

**Severity guide:**
- Stale records blocking new ones (queue saturation) → HIGH
- Stale records inflating metrics → MEDIUM
- Stale records consuming storage only → LOW

---

### Pattern 6: Dead Code / Overwritten Logic

**What:** Code that computes a value that is immediately overwritten, never called, or unreachable.

**Why it's dangerous:** The dead code itself doesn't cause bugs — but it's a maintenance bomb. Someone reading the code may trust the dead logic, or a future refactor may remove the live override while keeping the dead code, silently introducing a bug.

**Search for:**
- Variables assigned on consecutive lines without being read between assignments
- Functions defined but never called (grep for the function name)
- Branches that can never execute (always-true/always-false conditions)
- Imports that are unused

**Fix:** Remove the dead code. If the dead code represents an intentional alternative approach, convert it to a comment explaining why it was rejected.

**Severity guide:**
- Dead code that overwrites correct code (wrong line kept → bug) → MEDIUM
- Dead functions/imports → LOW
- Intentional stubs (TODO, future feature) → not a bug

---

### Pattern 7: Authority Source Mismatch

**What:** The pipeline validates, resolves, or checks outcomes using a different data source than the one that determines the actual truth.

**Why it's dangerous:** If your pipeline checks state using Source A, but the actual truth lives in Source B, you get false results. Even if A and B are derived from the same underlying data, differences in timing, rounding, caching, or revision history can cause disagreements.

**Search for:**
- Document what the source of truth is for each decision (database, API, config, external authority)
- Verify the code queries that same source
- Check for timing gaps (cached data vs. live data)
- Check for rounding/precision differences between sources

**Examples:**
```python
# DANGEROUS — checking inventory from cache, but DB is source of truth
if cached_inventory > 0:  # Cache may be stale

# DANGEROUS — validating permissions from local copy
if local_permissions.has("admin"):  # Auth service may have revoked

# DANGEROUS — comparing results against a snapshot
assert result == expected_from_last_week  # Spec may have changed
```

**Fix:** Always resolve from the source of truth directly. If the authority is an API, query that API. If the authority is a specific database table, read from that table. Document which source is authoritative for each decision.

**Severity guide:**
- Different data source entirely → CRITICAL
- Same source but different pipeline/timing (e.g., cached vs. live) → HIGH
- Same source, minor precision difference → MEDIUM

---

### Pattern 8: Hardcoded Values / Magic Numbers

**What:** Numeric constants embedded in code that should come from configuration.

**Why it's dangerous:** When behavior needs to change, operators look in config files. If the value is hardcoded, it's invisible to operations and requires a code change + deployment to modify.

**Search for:**
- Numeric literals in business logic (thresholds, limits, scaling factors)
- String literals that represent configurable categories or modes
- Default values in function signatures that override config

**Fix:** Move to config with a descriptive key name. If the value truly cannot change (mathematical constant, protocol requirement), document it with a comment explaining why it's hardcoded.

**Severity guide:**
- Hardcoded safety threshold (max retries, resource limit, rate cap) → HIGH
- Hardcoded business parameter (scaling factor, timeout) → MEDIUM
- Hardcoded constant with clear mathematical basis → LOW

---

### Pattern 9: Error Handling Gaps

**What:** External calls (APIs, databases, file I/O) without proper error handling, causing silent failures or crashes.

**Why it's dangerous:** Pipelines run unattended. A single unhandled exception can crash the entire run, leaving downstream stages with stale data. Silent failures (swallowed exceptions, default returns) can be worse — the pipeline "succeeds" but produces wrong outputs.

**Search for:**
- API calls without try/except or timeout
- Database operations without transaction safety
- File operations without existence checks
- JSON/YAML parsing without validation
- Rate limit handling (429 responses)

**Fix:** Every external call should have: timeout, retry logic (with limits), error logging, and a fail-closed default (skip the record, don't fabricate data).

**Severity guide:**
- Unhandled exception crashes the entire pipeline → HIGH
- Silent failure produces wrong data downstream → HIGH
- Exception handled but logged poorly (no context) → MEDIUM
- Redundant error handling (belt and suspenders) → LOW (not a bug)

---

### Pattern 10: Data Leakage / Temporal Violations

**What:** Using future data to make decisions that should only use past data.

**Why it's dangerous:** In any pipeline that makes predictions, backtests, or decisions based on historical data, accidentally including data from after the decision point invalidates all results. Backtests look artificially good. Models appear calibrated when they aren't.

**Search for:**
- Date filters that don't cap at the decision/computation date
- Backtester using the full dataset instead of as-of snapshots
- Cache entries from future dates leaking into historical queries
- "Look-ahead" in rolling calculations (using future values in a window)

**Fix:** Every query that feeds into a decision must have a `date <= decision_date` cap. Backtesters should recompute all derived data as-of each historical point. Verify with a test: shuffle the data order and confirm results don't change.

**Severity guide:**
- Backtester uses future data → CRITICAL (invalidates all validation)
- Production query can include future-dated records → HIGH
- Historical analysis includes preliminary/revised data → MEDIUM

---

## Part 4: Cross-Reference Checklist

Beyond the 10 bug patterns, verify these structural concerns:

### Config Completeness

For every config file:

| Check | How |
|-------|-----|
| Every key is used in code | Grep each key in source files |
| No dead keys | Keys with zero grep hits → dead config |
| Defaults in code match config | Compare function defaults to YAML values |
| Config is loaded at startup, not per-request | Check for repeated file reads |

### Strategy/Module Isolation

If the system has multiple independent pipelines or strategies:

| Check | How |
|-------|-----|
| No cross-imports between pipelines | Grep imports in each module directory |
| Separate database tables | Check schema for table name prefixes |
| Separate CLI commands | Verify no shared state between commands |
| Failure isolation | One pipeline's failure doesn't block another |

### Database Safety

| Check | How |
|-------|-----|
| Primary keys on all tables | Check CREATE TABLE statements |
| Indexes on query columns | Check for missing indexes on WHERE/JOIN columns |
| No SQL injection risk | Parameterized queries, no string formatting |
| Schema migration path | Can new columns be added without breaking old code? |
| Connection cleanup | Connections closed after use (context managers) |

---

## Part 5: Conducting the Audit

### Step-by-Step Process

Follow these steps in order. Do not skip steps or combine them.

**1. Preparation (before reading code)**
- List all source files in the pipeline, in execution order
- List all config files
- Read any existing specs or architecture docs
- Read any prior audit documents (to check for regressions)

**2. Stage-by-stage review**
- Read each file top to bottom
- For each function: what does it do, what are its inputs, what can go wrong?
- Check each of the 10 bug patterns at every decision point
- Document findings immediately with file:line references (don't batch — you'll lose context)

**3. Config cross-reference**
- For every key in every config file, grep the codebase
- Flag unused keys
- Flag hardcoded values that should be in config

**4. Integration review**
- How do stages connect? (function calls, database, message queue)
- What happens if a stage fails? (retry, skip, crash)
- What happens if a stage runs twice? (idempotent or duplicate)
- What happens if stages run out of order? (parallel schedulers, race conditions)

**5. Write the audit document**
- Follow the structure in Part 1 exactly
- Every finding gets: ID, severity, file:line, impact, fix
- Include code snippets for context — the fix session needs to see the actual code
- End with prioritized fix recommendations
- If no bugs are found in a stage, state that explicitly ("No bugs found in [stage name]")

### Audit as a Handoff

The audit document is the input to a bug-fix session. It should be detailed enough that a developer who has never seen the codebase can:
1. Find the bug (file:line)
2. Understand the impact (what goes wrong)
3. Implement the fix (recommended approach)
4. Verify the fix (what to test)

If your audit doesn't pass this bar, add more detail.

---

## Part 6: Audit Handoff Template

When the audit is complete and you're ready to fix the findings, use this handoff structure:

```markdown
# [Pipeline Name] Bug Fixes -- Handoff

---
Session: VX.X / SXX
Author: Claude Chat
Depends On: [Pipeline audit document filename]
Supersedes: None
Risk Level: [Based on highest severity finding]
---

## Objective

Fix [N] bugs identified in the [pipeline name] audit — [severity breakdown].

## Context

[Audit summary. Link to full audit document.]

## Requirements

[One requirement per bug, referencing audit ID:]

**R1: [Short description] (Audit [ID] — [SEVERITY])**
- [Location, cause, fix approach]

## Success Criteria

- [ ] [Testable criterion per bug]
- [ ] ≥[N] new tests added
- [ ] All tests pass (0 regressions from [baseline] baseline)

## Constraints

- Do not refactor beyond what is needed to fix each bug
- Preserve existing test coverage

## Non-Goals

- Performance optimization (unless it causes incorrect output)
- Code style or formatting changes

## Files

[Table of files to modify, mapping to requirement IDs]

| File | Action | Requirements |
|------|--------|-------------|
| `path/to/file.py` | Modify | R1, R3 |

## Authority

This document defines intent and success criteria.
The codebase defines current reality.
If conflicts exist, surface them in plan mode before implementation begins.
```

---

## Part 7: Quick Reference Card

### Before Starting an Audit

```
- [ ] List all source files in execution order
- [ ] List all config files
- [ ] Read specs/architecture docs
- [ ] Read prior audits (if any)
- [ ] Confirm: audit is READ-ONLY (no code changes)
```

### During the Audit (Check Every Stage)

```
- [ ] Pattern 1:  Numeric truthiness (0 is valid?)
- [ ] Pattern 2:  Asymmetric input handling (both sides same value?)
- [ ] Pattern 3:  Config defined but unused?
- [ ] Pattern 4:  Missing deduplication?
- [ ] Pattern 5:  Missing expiry/staleness?
- [ ] Pattern 6:  Dead code / overwritten logic?
- [ ] Pattern 7:  Authority source mismatch?
- [ ] Pattern 8:  Hardcoded values / magic numbers?
- [ ] Pattern 9:  Error handling gaps?
- [ ] Pattern 10: Data leakage / temporal violations?
```

### After the Audit

```
- [ ] Every source file reviewed
- [ ] Every config file cross-referenced
- [ ] All 10 patterns checked (found or not found)
- [ ] All findings have: ID, severity, file:line, impact, fix
- [ ] Bug list table complete (ordered by severity)
- [ ] Fix priority recommendations written
- [ ] Findings reviewed with pipeline owner before handoff
- [ ] Audit document committed to version control
```

---

## What This Template Is NOT

- **Not a linter or static analysis tool.** This template guides human-led code review, not automated scanning.
- **Not a security audit.** It covers code correctness, not penetration testing or vulnerability assessment.
- **Not a performance review.** It flags correctness bugs, not optimization opportunities (unless they cause incorrect output).
- **Not a replacement for tests.** An audit finds bugs; tests prevent regressions. You need both.

---

## Document Metadata

| Field | Value |
|-------|-------|
| Template Version | 1.0.0 |
| Created | 2026-02-12 |
| Origin | Derived from production pipeline audits across multiple domains |
| Companion Docs | HANDOFF_TEMPLATE.md, CLAUDE_CODE_PROTOCOL.md, AGENT_BEST_PRACTICES.md |
