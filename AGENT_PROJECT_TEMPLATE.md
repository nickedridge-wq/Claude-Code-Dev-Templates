# [PROJECT_NAME] Agent -- Project Document

**Version:** 0.0.1
**Last Updated:** [DATE]
**Status:** [ ] Planning | [WIP] Building | [DONE] Complete
**Location:** `/Users/[USERNAME]/[project-name]`
**Author:** [YOUR_NAME] + Claude

---

## [!] MANDATORY COMPLIANCE CHECK

**Before creating ANY output file -- especially handoff documents for Claude Code -- you MUST:**

1. **Reference `AGENT_BEST_PRACTICES.md`** and verify the planned work aligns with all applicable best practices
2. **Reference all `*_PROJECT` files** (e.g., `MY_AGENT_PROJECT.md`) to ensure consistency with existing architecture, decisions, and patterns
3. **Reference `SCHEDULER_TEMPLATE.md`** if the agent requires scheduling/automation -- use the template structure exactly
4. **Reference `PIPELINE_AUDIT_TEMPLATE.md`** before bug-fix sessions -- audit first, then fix

**Confirm compliance:**
- [ ] Separation of concerns (Tools != Pipelines != Agent)
- [ ] Tools are dumb (execute only, no decisions)
- [ ] Configuration loaded at runtime, not import time
- [ ] Secrets via environment variables
- [ ] Structured input/output patterns
- [ ] Work aligned with stated goals and non-goals
- [ ] Scheduling follows `SCHEDULER_TEMPLATE.md` (if applicable)
- [ ] Bug fixes preceded by audit using `PIPELINE_AUDIT_TEMPLATE.md` (if applicable)

```
+---------------------------------------------------------------------+
| [!] STOP: Before writing any handoff document or code spec...       |
|                                                                     |
| 1. Open AGENT_BEST_PRACTICES.md                                     |
| 2. Open all *_PROJECT files for this project                        |
| 3. If scheduling needed -> Open SCHEDULER_TEMPLATE.md               |
| 4. If fixing bugs -> Open PIPELINE_AUDIT_TEMPLATE.md                |
| 5. Verify compliance with all applicable templates                  |
| 6. Only then proceed with creating output                           |
+---------------------------------------------------------------------+
```

---

## How to Use This Template

1. **Copy this template** to your new project directory
2. **Fill in the placeholders** marked with `[PLACEHOLDER]`
3. **Delete sections** that don't apply (e.g., Scheduling if not needed)
4. **Update status indicators** as work progresses (`[ ]` -> `[WIP]` -> `[DONE]`)
5. This is a **living document** -- it evolves from planning through completion

```
PROJECT LIFECYCLE:

Planning                   Building                   Complete
--------------------------------------------------------->

[?] Placeholder      [[WIP]] In Progress      [[DONE]] Done
Status: [ ] Planning   Status: [WIP] Building   Status: [DONE] Complete
```

---

## Table of Contents

1. [Project Overview & Vision](#1-project-overview--vision)
2. [Development Philosophy](#2-development-philosophy)
3. [Architecture Blueprint](#3-architecture-blueprint)
4. [Project Structure](#4-project-structure)
5. [Technical Specifications](#5-technical-specifications)
6. [Configuration & Secrets](#6-configuration--secrets)
7. [External Services & Costs](#7-external-services--costs)
8. [Scheduling & Automation](#8-scheduling--automation)
9. [Housekeeping & Cleanup](#9-housekeeping--cleanup)
10. [Development Process](#10-development-process)
11. [Session Tracking](#11-session-tracking)
12. [Best Practices Checklist](#12-best-practices-checklist)
13. [Success Criteria](#13-success-criteria)
14. [Risk Register](#14-risk-register)
15. [Roadmap & What Remains](#15-roadmap--what-remains)
16. [Operations Guide](#16-operations-guide)
17. [Quick Reference](#17-quick-reference)
18. [Lessons Learned](#18-lessons-learned)

---

## 1. Project Overview & Vision

### Project Identity

| Field | Value |
|-------|-------|
| **Project Name** | [PLACEHOLDER: e.g., My Content Agent] |
| **Codename** | [PLACEHOLDER: e.g., edge-content-agent] |
| **Version** | 0.0.1 |
| **Status** | [ ] Planning |
| **Location** | `/Users/[USERNAME]/[project-name]` |

### What This Is

> [PLACEHOLDER: One sentence describing what this agent does. Be specific about the autonomous capability.]

**Example formats:**
- "An **autonomous AI content agent** that generates and publishes content across multiple platforms with zero manual intervention."
- "An **AI-powered data analysis agent** that monitors market signals, generates reports, and tracks performance over time."

### The End Goal

> [PLACEHOLDER: The aspirational vision. One paragraph max.]

**Template:** Build a fully autonomous AI system that [CORE_ACTION], [VALUE_PROPOSITION], and [OUTCOME] -- all with minimal human intervention.

### Why This Matters

- **[Reason 1]** - [Explanation]
- **[Reason 2]** - [Explanation]
- **[Reason 3]** - [Explanation]

This system solves: **[the hard problem in one sentence].**

### What Success Looks Like

**Daily Automated Cycle:**

```
+---------------------------------------------------------------------+
| [TIME]   | Scheduler triggers automatically                        |
| [TIME+1] | [Step 1 description]                                    |
| [TIME+2] | [Step 2 description]                                    |
| [TIME+3] | [Step 3 description]                                    |
| [TIME+4] | Log results, send notification                          |
|          |                                                         |
| You:     | Sleeping / Working / Living your life                   |
+---------------------------------------------------------------------+
```

### Current State

```
+---------------------------------------------------------------------+
|                        [PROJECT_NAME] AGENT                         |
|                         [STATUS INDICATOR]                          |
+---------------------------------------------------------------------+
|                                                                     |
| [STATUS] [Component 1]                                    [STATE]   |
|          * [Sub-capability 1]                                       |
|          * [Sub-capability 2]                                       |
|                                                                     |
| [STATUS] [Component 2]                                    [STATE]   |
|          * [Sub-capability 1]                                       |
|          * [Sub-capability 2]                                       |
|                                                                     |
| [STATUS] [Component 3]                                    [STATE]   |
|          * [Sub-capability 1]                                       |
|          * [Sub-capability 2]                                       |
|                                                                     |
| [ ] [Deferred Component]                         DEFERRED TO v2.0   |
|          * [Future capability]                                      |
|                                                                     |
+---------------------------------------------------------------------+

Status: [DONE] Complete  [WIP] In Progress  [ ] Planned  [ ] Blocked
```

### Key Metrics

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| [Primary metric 1] | [target] | - | [ ] |
| [Primary metric 2] | [target] | - | [ ] |
| [Cost metric] | < $X/unit | - | [ ] |
| Automation | Daily, hands-off | - | [ ] |
| Human intervention | < X min/day | - | [ ] |

### Non-Goals (Explicit Scope Limits)

- [PLACEHOLDER: What this agent will NOT do]
- [PLACEHOLDER: Features explicitly deferred]
- [PLACEHOLDER: Things that seem related but are out of scope]

---

## 2. Development Philosophy

### Roles & Responsibilities

```
+---------------------------------------------------------------------+
|                       DEVELOPMENT WORKFLOW                          |
|                                                                     |
| [BRAIN] CLAUDE CHAT (Brains)                                        |
|         +-- Strategic planning & architecture decisions             |
|         +-- Research & cost optimization                            |
|         +-- Push back on bad ideas with alternatives                |
|         +-- Create detailed handoff documents                       |
|         +-- Never lose sight of the end product                     |
|                                                                     |
| [BUILD] CLAUDE CODE (Muscle)                                        |
|         +-- Implement code from handoff documents                   |
|         +-- Run tests and verify functionality                      |
|         +-- Report results and blockers                             |
|         +-- Execute physical build tasks                            |
|                                                                     |
| [USER]  YOU (Decision Maker)                                        |
|         +-- Provide requirements and constraints                    |
|         +-- Approve architectural decisions                         |
|         +-- Test end-to-end functionality                           |
|         +-- Make final go/no-go calls                               |
+---------------------------------------------------------------------+
```

### Core Principles

| Principle | Description | Why It Matters |
|-----------|-------------|----------------|
| **Data-Driven Decisions** | Research options, compare costs, validate assumptions | Avoids expensive mistakes |
| **Best Practices First** | Architecture > shortcuts | Maintainable, scalable code |
| **Cost Consciousness** | Minimize costs without sacrificing quality | Sustainable operation |
| **End Goal Focus** | Every decision serves the final product | Prevents scope creep |
| **Push Back Culture** | Challenge ideas that seem wrong | Better outcomes |

### Cost Optimization Guidelines

- Always research multiple providers before committing
- Calculate per-unit AND monthly costs
- Consider free tiers and credits
- Batch operations where possible
- Cache expensive results
- Monitor costs actively, not reactively

---

## 3. Architecture Blueprint

### System Overview

```
+---------------------------------------------------------------------+
|                            INTERFACES                               |
|         CLI    |    Scheduler    |    API (future)    |    Web UI  |
+---------------------------------------------------------------------+
                                   |
                                   v
+---------------------------------------------------------------------+
|                           ORCHESTRATION                             |
| +---------------------------------------------------------------+   |
| |                      Agent Core (v2.0)                        |   |
| |           OBSERVE -> DECIDE -> ACT -> LEARN reasoning loop    |   |
| +---------------------------------------------------------------+   |
|                                 |                                   |
| +---------------------------------------------------------------+   |
| |                    Orchestrator / Runner                      |   |
| |                    * Coordinates pipelines                    |   |
| |                    * Manages state                            |   |
| |                    * Handles errors & retries                 |   |
| |                    * Logs results                             |   |
| +---------------------------------------------------------------+   |
+---------------------------------------------------------------------+
                                   |
                                   v
+---------------------------------------------------------------------+
|                            PIPELINES                                |
|   +-------------+    +-------------+    +-------------+             |
|   | Pipeline 1  |    | Pipeline 2  |    | Pipeline N  |             |
|   |  [purpose]  |    |  [purpose]  |    |  [purpose]  |             |
|   +-------------+    +-------------+    +-------------+             |
+---------------------------------------------------------------------+
                                   |
                                   v
+---------------------------------------------------------------------+
|                           TOOLS LAYER                               |
|   +-------------+  +-------------+  +-------------+  +-------------+|
|   |   Tool A    |  |   Tool B    |  |   Tool C    |  |   Tool D    ||
|   |  [service]  |  |  [service]  |  |  [service]  |  |  [service]  ||
|   +-------------+  +-------------+  +-------------+  +-------------+|
+---------------------------------------------------------------------+
                                   |
                                   v
+---------------------------------------------------------------------+
|                        EXTERNAL SERVICES                            |
|      [Service A]    [Service B]    [Service C]    [Service D]       |
|       [purpose]      [purpose]      [purpose]      [purpose]        |
|        $X/unit       $X/unit        $X/unit        $X/month         |
+---------------------------------------------------------------------+
```

### Layer Responsibilities

| Layer | Responsibility | Rules |
|-------|---------------|-------|
| **Interfaces** | User interaction | Replaceable, no business logic |
| **Orchestration** | Coordination & decisions | Owns state, manages flow |
| **Pipelines** | Multi-step processes | Compose tools, no external calls |
| **Tools** | Single actions | Dumb executors, no decisions |
| **External** | Third-party services | Abstracted behind tools |

### Data Flow

```
+---------------------------------------------------------------------+
|                              INPUT                                  |
|              [PLACEHOLDER: What triggers the agent?]                |
+---------------------------------------------------------------------+
                                   |
                                   v
+---------------------------------------------------------------------+
|                       PROCESSING STAGE 1                            |
|                 [PLACEHOLDER: First transformation]                 |
|                   Input: [type] -> Output: [type]                   |
+---------------------------------------------------------------------+
                                   |
                                   v
+---------------------------------------------------------------------+
|                       PROCESSING STAGE 2                            |
|                [PLACEHOLDER: Second transformation]                 |
|                   Input: [type] -> Output: [type]                   |
+---------------------------------------------------------------------+
                                   |
                                   v
+---------------------------------------------------------------------+
|                              OUTPUT                                 |
|                  [PLACEHOLDER: Final deliverable]                   |
+---------------------------------------------------------------------+
```

---

## 4. Project Structure

```
[project-name]/
|
+-- config/                              # Configuration
|   +-- default.yaml                     # Main configuration
|   +-- [domain].yaml                    # Domain-specific config
|   +-- com.[projectid].daily.plist      # launchd configuration (if scheduled)
|
+-- src/                                 # Source code
|   +-- __init__.py
|   +-- scheduler.py                     # Scheduler entry point (flat, at src/ level)
|   |
|   +-- agent/                           # Agent layer (orchestration)
|   |   +-- __init__.py
|   |   +-- runner.py                    # Main orchestrator
|   |   +-- state.py                     # State management
|   |
|   +-- tools/                           # Tools layer (dumb executors)
|   |   +-- __init__.py
|   |   +-- base.py                      # Tool interface contract
|   |   +-- [tool_name].py               # Individual tools
|   |
|   +-- pipelines/                       # Pipeline layer (compositions)
|   |   +-- __init__.py
|   |   +-- [pipeline_name].py           # Individual pipelines
|   |
|   +-- utils/                           # Shared utilities
|       +-- __init__.py
|       +-- config.py                    # Configuration loader
|       +-- logging.py                   # Structured logging
|       +-- costs.py                     # Cost tracking
|
+-- interfaces/                          # Entry points
|   +-- cli.py                           # Command-line interface
|
+-- scripts/                             # Management scripts
|   +-- run_daily.sh                     # launchd wrapper (if scheduled)
|   +-- install_scheduler.sh             # Installation (if scheduled)
|   +-- uninstall_scheduler.sh           # Removal (if scheduled)
|   +-- health_check.sh                  # System verification
|
+-- data/                                # Runtime data (git-ignored)
|   +-- state.json                       # Persistent state
|   +-- .lock                            # Concurrency lock
|   +-- .disabled                        # If exists, agent is paused
|   +-- logs/
|       +-- agent.log                    # Application logs
|       +-- production.jsonl             # Production activity
|
+-- tests/                               # Test suite
|   +-- conftest.py                      # Test fixtures
|   +-- test_[module].py
|
+-- docs/                                # Documentation (version-scoped)
|   +-- v1.0/
|   |   +-- V1.0_PROJECT_DOCUMENT.md     # Master project doc for v1.0
|   |   +-- V1.0_SESSION_01_HANDOFF.md
|   |   +-- V1.0_SESSION_01_IMPLEMENTATION_SUMMARY.md
|   |   +-- V1.0_SESSION_02_HANDOFF.md
|   |   +-- V1.0_SESSION_02_IMPLEMENTATION_SUMMARY.md
|   +-- v1.5/                            # Sessions restart at 01 for each version
|       +-- V1.5_PROJECT_DOCUMENT.md
|       +-- V1.5_SESSION_01_HANDOFF.md
|       +-- ...
|
+-- .env                                 # API keys (git-ignored, in project root)
+-- requirements.txt                     # Python dependencies
+-- .gitignore
+-- README.md
```

### Key Files Reference

| Purpose | File(s) |
|---------|---------|
| **Configuration** | `config/default.yaml`, `.env` |
| **Entry Points** | `interfaces/cli.py`, `src/scheduler.py` |
| **Core Logic** | `src/agent/runner.py`, `src/pipelines/*.py` |
| **Tools** | `src/tools/*.py` |
| **Scheduling** | `config/com.[projectid].daily.plist`, `scripts/run_daily.sh` |
| **State** | `data/state.json`, `data/logs/production.jsonl` |
| **Project Doc** | `docs/vX.X/VX.X_PROJECT_DOCUMENT.md` |
| **Session Docs** | `docs/vX.X/VX.X_SESSION_XX_HANDOFF.md`, `docs/vX.X/VX.X_SESSION_XX_IMPLEMENTATION_SUMMARY.md` |

---

## 5. Technical Specifications

### Tool Interface Contract

Every tool MUST follow this contract:

```python
from dataclasses import dataclass
from typing import Any

@dataclass
class ToolInput:
    """Base class for tool inputs - extend per tool"""
    pass

@dataclass
class ToolOutput:
    """Standard output for all tools"""
    success: bool
    data: Any = None
    error: str | None = None
    retryable: bool = False
    cost: float = 0.0

class Tool:
    """Base class for all tools"""
    
    def execute(self, input: ToolInput) -> ToolOutput:
        """
        Execute the tool's single responsibility.
        
        Rules:
        - No decisions (dumb executor)
        - No side effects beyond stated purpose
        - Always return structured output
        - Handle own errors gracefully
        - Track costs accurately
        """
        raise NotImplementedError
```

### State Management

```python
@dataclass
class AgentState:
    """Persistent state across runs"""
    last_run: str | None = None
    run_count: int = 0
    total_cost: float = 0.0
    # Add project-specific state fields
    
    def save(self, path: str) -> None:
        """Persist state to JSON"""
        pass
    
    @classmethod
    def load(cls, path: str) -> "AgentState":
        """Load state from JSON"""
        pass
```

### Error Handling Pattern

```python
class AgentError(Exception):
    """Base exception for agent errors"""
    retryable: bool = False

class RetryableError(AgentError):
    """Errors that can be retried"""
    retryable = True

class FatalError(AgentError):
    """Errors that should not be retried"""
    retryable = False
```

### Logging Standards

```python
import logging
from datetime import datetime

# Structured log format
LOG_FORMAT = {
    "timestamp": datetime.utcnow().isoformat(),
    "level": "INFO",
    "component": "tool_name",
    "message": "action completed",
    "data": {},
    "cost": 0.0
}
```

### Processing Pipeline

```
+---------------------------------------------------------------------+
| STAGE 1: [NAME] ([duration])                                        |
|          Input: [type] -> Output: [type]                            |
+---------------------------------------------------------------------+
| STAGE 2: [NAME] ([duration])                                        |
|          Input: [type] -> Output: [type]                            |
+---------------------------------------------------------------------+
| STAGE 3: [NAME] ([duration])                                        |
|          Input: [type] -> Output: [type]                            |
+---------------------------------------------------------------------+
| STAGE 4: [NAME] ([duration])                                        |
|          Input: [type] -> Output: [type]                            |
+---------------------------------------------------------------------+

Total Duration: [X-Y] [units]
```

---

## 6. Configuration & Secrets

### Configuration File (config/default.yaml)

```yaml
# [PROJECT_NAME] Configuration
# Environment: development | production

app:
  name: "[project-name]"
  version: "0.0.1"
  environment: "development"
  
paths:
  data: "data"
  logs: "data/logs"
  output: "data/output"

features:
  dry_run: false
  verbose: false
  
# Add project-specific configuration
```

### Environment Variables (.env)

```bash
# API Keys (NEVER commit this file)
# Copy to .env and fill in values

# Primary Services
[SERVICE_A]_API_KEY=
[SERVICE_B]_API_KEY=

# Optional Services
[SERVICE_C]_API_KEY=

# Platform Credentials (if applicable)
[PLATFORM]_ACCOUNT_ID=
```

### Configuration Loading

```python
import os
import yaml
from pathlib import Path

def load_config(env: str = None) -> dict:
    """Load configuration at runtime, not import time"""
    env = env or os.getenv("APP_ENV", "development")
    config_path = Path("config/default.yaml")
    
    with open(config_path) as f:
        config = yaml.safe_load(f)
    
    # Override with environment variables
    for key in os.environ:
        if key.startswith("APP_"):
            # Map env vars to config
            pass
    
    return config
```

### Secrets Checklist

- [ ] All secrets in `.env` file
- [ ] `.env` added to `.gitignore`
- [ ] No secrets in config files
- [ ] No secrets in code
- [ ] Secrets loaded at runtime only
- [ ] `.env.example` provided for new setups

---

## 7. External Services & Costs

### Service Inventory

| Service | Purpose | Plan | Cost | Status |
|---------|---------|------|------|--------|
| [PLACEHOLDER] | [purpose] | [plan] | $X/unit | [ ] Research |
| [PLACEHOLDER] | [purpose] | [plan] | $X/unit | [ ] Research |
| [PLACEHOLDER] | [purpose] | [plan] | $X/month | [ ] Research |

### Cost Analysis

**Per-Unit Cost Breakdown:**

```
+------------------------------------------------------+
| COST PER [UNIT]:                            $X.XX    |
+------------------------------------------------------+
| [Component A]                             | $X.XXX   |
| [Component B]                             | $X.XXX   |
| [Component C]                             | $X.XXX   |
| [Local Processing]                        | $0.000   |
+------------------------------------------------------+
| TOTAL                                     | $X.XXX   |
+------------------------------------------------------+
```

**Monthly Projection ([N] units):**

```
+------------------------------------------------------+
| MONTHLY COST:                              ~$XXX     |
+------------------------------------------------------+
| [Service A] ([N] units)                   | $XX.XX   |
| [Service B] ([N] units)                   | $XX.XX   |
| [Service C] (flat fee)                    | $XX.XX   |
+------------------------------------------------------+
| TOTAL                                     | $XXX.XX  |
| Target Budget                             | $XXX.XX  |
| Under/Over Budget                         | +/-$XX.XX|
+------------------------------------------------------+
```

### Service Selection Decisions

| Decision | Options Considered | Choice | Rationale |
|----------|-------------------|--------|-----------|
| [Component] | Option A, B, C | **[Choice]** | [Why - cost/quality/features] |

---

## 8. Scheduling & Automation

### Does This Agent Need Scheduling?

| If the agent needs to... | Scheduling Required |
|--------------------------|---------------------|
| Run automatically at set times | Yes |
| Run continuously in background | Yes |
| Run only when manually triggered | No -- delete this section |
| React to external events (webhooks) | Maybe (event-driven, not scheduled) |

### If Scheduling Is Required

**[!] MANDATORY: Use `SCHEDULER_TEMPLATE.md` exactly.**

Do NOT create a custom scheduling solution. The scheduler template provides:
- launchd configuration (native macOS, survives reboots)
- Lock mechanism (prevents concurrent runs)
- State persistence (JSON)
- Retry logic with daily limits
- Emergency brake safeguard
- Enable/disable toggle (pause without uninstalling)
- macOS notifications
- Optional rotation logic
- Install/uninstall scripts

### Scheduler Configuration (Project-Specific)

| Placeholder | Value |
|-------------|-------|
| `[PROJECT_NAME]` | [PLACEHOLDER] |
| `[PROJECT_ID]` | [PLACEHOLDER] |
| `[USERNAME]` | [PLACEHOLDER] |
| `[SCHEDULE_HOUR]` | [PLACEHOLDER] |
| `[SCHEDULE_MINUTE]` | [PLACEHOLDER] |
| `[ROTATION_ITEMS]` | [PLACEHOLDER or "none"] |

### Schedule Type

```
[ ] Daily at specific time (most common)
[ ] Multiple times per day
[ ] Every N seconds/minutes
[ ] Specific weekday only
[ ] Monthly
[ ] Other: _______________
```

### Quick Reference

```bash
# Scheduler commands (after implementing SCHEDULER_TEMPLATE.md)
python -m src.scheduler run              # Execute task
python -m src.scheduler run --dry-run    # Test without side effects
python -m src.scheduler run --force      # Bypass safeguards
python -m src.scheduler status           # Show state & safeguards
python -m src.scheduler enable           # Resume (remove pause)
python -m src.scheduler disable          # Pause (without uninstalling)
```

---

## 9. Housekeeping & Cleanup

### Does This Agent Need Cleanup?

| If the agent generates... | Cleanup Recommended |
|---------------------------|---------------------|
| Video/audio/image files | Yes -- these accumulate fast |
| Large data files or caches | Yes |
| Log files | Yes -- rotate after 7-30 days |
| Only small text outputs | Maybe -- low priority |
| Nothing persistent | No -- delete this section |

**Rule of thumb:** If your agent could generate >1GB/year of files, implement cleanup.

### What to Clean vs. Protect

| Action | File Types |
|--------|------------|
| **DELETE after N days** | Generated media (videos, audio, images), intermediate files, temp/cache, old logs |
| **NEVER auto-delete** | `state.json`, `production.jsonl`, configuration files, session documentation |

### If Cleanup Is Required

**[!] MANDATORY: Use `CLEANUP_TEMPLATE.md` exactly.**

Do NOT create a custom cleanup solution. The cleanup template provides:
- File cleaner tool (dumb executor)
- Cleanup runner (orchestration)
- Retention policy configuration
- Separate launchd plist (weekly schedule)
- Separate lock file (prevents conflicts with daily runs)
- Dry-run mode for testing
- Protected path safeguards

### Storage Impact Estimation

Calculate your expected storage with and without cleanup:

| Asset Type | Size Each | Daily Volume | Retention | Max Storage |
|------------|-----------|--------------|-----------|-------------|
| [Type 1] | ~XX MB | X/day | X days | ~XXX MB |
| [Type 2] | ~XX MB | X/day | X days | ~XXX MB |
| [Type 3] | ~XX KB | X/day | X days | ~XXX KB |
| **Total** | -- | -- | -- | **~X GB** |

**Without cleanup:** ~XX GB/year
**With cleanup:** Never exceeds ~X GB

### Quick Reference

```bash
# Cleanup commands (after implementing CLEANUP_TEMPLATE.md)
python -m src.scheduler cleanup              # Run cleanup now
python -m src.scheduler cleanup --dry-run    # Preview what would be deleted
```

### Session Documentation Archiving

**[!] Do NOT auto-archive session docs.** They're small (~5KB each) and valuable for context.

Only consider manual archiving when:
- You have 20+ session folders AND
- You're releasing a major version (v1 -> v2)

---

## 10. Development Process

### Session Workflow

```
+---------------------------------------------------------------------+
|                        SESSION WORKFLOW                             |
|                                                                     |
| 1. PLAN (Claude Chat)                                               |
|    +-- Define session objective                                     |
|    +-- Research requirements                                        |
|    +-- Make architectural decisions                                 |
|    +-- Create handoff document (use HANDOFF_TEMPLATE.md)            |
|    +-- Define success criteria                                      |
|                                                                     |
| 2. BUILD (Claude Code)                                              |
|    +-- Follow CLAUDE_CODE_PROTOCOL.md exactly                       |
|    +-- Save handoff, enter plan mode, build, verify                 |
|                                                                     |
| 3. VERIFY (You + Claude Chat)                                       |
|    +-- Review implementation                                        |
|    +-- Test end-to-end                                              |
|    +-- Iterate until success criteria met                           |
|                                                                     |
| 4. CLOSE (Claude Code -- MANDATORY)                                 |
|    +-- Write implementation summary (see HANDOFF_TEMPLATE.md)       |
|    +-- Commit all changes                                           |
|    +-- Update this project document                                 |
|                                                                     |
+---------------------------------------------------------------------+
```

### Session Documentation

**[!] MANDATORY: Use `HANDOFF_TEMPLATE.md` for all session documents.**

Every session produces TWO documents in `docs/vX.X/`:

| Document | Created By | When | Template |
|----------|-----------|------|----------|
| `VX.X_SESSION_XX_HANDOFF.md` | Claude Chat | Before build | HANDOFF_TEMPLATE.md Part 1 |
| `VX.X_SESSION_XX_IMPLEMENTATION_SUMMARY.md` | Claude Code | After build | HANDOFF_TEMPLATE.md Part 2 |

```
docs/
+-- v1.0/
|   +-- V1.0_PROJECT_DOCUMENT.md                    # This document
|   +-- V1.0_SESSION_01_HANDOFF.md                  # Input
|   +-- V1.0_SESSION_01_IMPLEMENTATION_SUMMARY.md   # Output
+-- v1.5/                                           # Sessions restart at 01 per version
    +-- V1.5_SESSION_01_HANDOFF.md
```

### Claude Code Execution Rules

**[!] MANDATORY: Claude Code must follow `CLAUDE_CODE_PROTOCOL.md` exactly.**

The protocol defines:
- Plan mode requirements (propose before building)
- Task tracking format and rules
- Definition of done checklist
- Commit message format and gates
- Session close requirements

### Definition of Done (Per Session)

Before closing any session, verify ALL items:

- [ ] Handoff saved to `docs/vX.X/VX.X_SESSION_XX_HANDOFF.md`
- [ ] All tasks from handoff completed (or marked [!] with explanation)
- [ ] All tests passing (or failures documented)
- [ ] Code executes without runtime errors
- [ ] Implementation summary written
- [ ] Changes committed with proper message format
- [ ] This project document updated

### Master Project Document Updates

**[!] MANDATORY: Update this document at the end of every session.**

| Section | What to Update |
|---------|----------------|
| **Current State** (S1) | Component status indicators |
| **Key Metrics** (S1) | Actual values if changed |
| **Session Log** (S11) | Add row for completed session |
| **Key Pivots & Decisions** (S11) | Decisions made during implementation |
| **Risk Register** (S14) | New risks discovered |
| **Roadmap** (S15) | Update completion status |
| **Lessons Learned** (S18) | Insights from this session |

---

## 11. Session Tracking

### Session Timeline

```
[Project Start Date]
+-- Session 1: [Name] ---------------------------- [ ] Planned
+-- Session 2: [Name] ---------------------------- [ ] Planned
+-- Session 3: [Name] ---------------------------- [ ] Planned
|
| 
|                        v1.0 TARGET
| 
|
+-- Session N: [Future Feature] ------------------ [TODO] Future
+-- v2.0: [Major Enhancement] -------------------- [TODO] Future
```

### Session Log

| Session | Date | Delivered | Key Decisions | Duration |
|---------|------|-----------|---------------|----------|
| 1 | - | - | - | - |

### Key Pivots & Decisions

| Date | Decision | Options Considered | Choice | Rationale |
|------|----------|-------------------|--------|-----------|
| YYYY-MM-DD | [decision] | A, B, C | **B** | [why] |

---

## 12. Best Practices Checklist

**Reference: AGENT_BEST_PRACTICES.md**

### Section 1 -- Agent Foundations

- [ ] Define the reasoning loop (decide -> act -> update state)
- [ ] Separate state from logic (no global variables)
- [ ] Keep tools dumb (execute actions, no reasoning)
- [ ] Establish a clean interface boundary (agent unaware of UI)
- [ ] Implement a single callable entry point (`run(input)`)

**Quick Check:** Can the agent run independently of the interface? Can it run repeatedly without errors?

### Section 2 -- Separating Concerns

- [ ] Core Agent: owns reasoning, state updates, decision-making
- [ ] Tools Layer: executes tasks, independent of agent logic
- [ ] Interface Layer: handles user input/output, replaceable
- [ ] Test layer replacement (swap UI without changing agent logic)

**Quick Check:** Does the agent survive UI swaps? Are tools independent and unit-testable?

### Section 3 -- Configuration & Secrets

- [ ] Define all configuration variables (paths, flags, features)
- [ ] Identify secrets (API keys, tokens, credentials)
- [ ] Load configuration at runtime (not import time)
- [ ] Use environment variables, config files, or runtime arguments
- [ ] Avoid shipping secrets in packaged apps

**Quick Check:** Can the agent run in multiple environments without modification?

### Section 4 -- Making the Agent Callable

- [ ] Single entry point (`run(input)`)
- [ ] Structured input and output (no printing/logging in logic)
- [ ] Support asynchronous operations
- [ ] Inject state and tools for flexibility
- [ ] Ensure agent returns consistent, predictable outputs

**Quick Check:** Can different interfaces call the agent without changing logic?

### Section 5 -- Version Control & Commit Discipline

- [ ] Commits are atomic and reversible
- [ ] Commit messages follow conventional format
- [ ] No secrets or .env files committed
- [ ] Release versions are tagged

**Quick Check:** Is every commit atomic, reversible, and properly messaged?

### Section 6 -- Packaging & Distribution

- [ ] Choose packaging method (Electron, Tauri, web, hybrid)
- [ ] Keep agent running in background process if desktop
- [ ] Separate UI from agent during packaging
- [ ] Plan for installers, signing, and permissions
- [ ] Abstract OS-specific paths and system calls

**Quick Check:** Can someone install and run the agent without dev tools?

### Section 7 -- User & Product Considerations

- [ ] Support per-user configurations and profiles
- [ ] Implement access control, licensing, or feature limits
- [ ] Provide error reporting and observability (ethically)
- [ ] Ensure logs are useful for debugging
- [ ] Test agent behavior under unexpected inputs and failures
- [ ] Implement operational controls (enable/disable, dry-run, status, reset)

**Quick Check:** Is the agent resilient under edge cases? Can it be paused without uninstalling?

### Section 8 -- Common Mistakes to Avoid

- [ ] NOT coupling logic to UI
- [ ] NOT making tools stateless for multi-step tasks
- [ ] NOT letting tools make decisions instead of executing
- [ ] NOT hard-coding configs or secrets
- [ ] NOT printing instead of returning structured output

### Code Quality

| Practice | Status | Notes |
|----------|--------|-------|
| Type hints | [ ] | All functions typed |
| Dataclasses | [ ] | Input/Output models |
| Error handling | [ ] | Try/except with logging |
| Configuration | [ ] | YAML + env vars |
| Secrets management | [ ] | .env files, git-ignored |
| Logging | [ ] | Structured, leveled |
| State persistence | [ ] | JSON state file |
| Lock mechanism | [ ] | Atomic locking (if concurrent) |

### Testing

| Level | Status | Notes |
|-------|--------|-------|
| Unit tests | [ ] | [X] tests |
| Integration tests | [ ] | Per-tool verification |
| E2E tests | [ ] | Full pipeline tested |
| Dry-run mode | [ ] | Test without side effects |

### Security

| Concern | Status | Mitigation |
|---------|--------|------------|
| API keys in code | [ ] | Environment variables |
| Secrets in git | [ ] | .gitignore |
| Concurrent access | [ ] | Lock file mechanism |

---

## 13. Success Criteria

### Definition of Done (v1.0)

| Criterion | Status | How to Verify |
|-----------|--------|---------------|
| [Core function 1] works end-to-end | [ ] | `python -m src.scheduler run --dry-run` |
| [Core function 2] produces correct output | [ ] | [Command] |
| [Core function 3] handles edge cases | [ ] | [Command] |
| Error handling: failures logged, not fatal | [ ] | Kill API mid-run, check logs |
| Cost tracking: per-unit and monthly | [ ] | `cat data/state.json` |
| Daily automation: runs on schedule | [ ] | `launchctl list \| grep [projectid]` |
| Failure notifications: macOS alerts | [ ] | `run --dry-run` with forced error |
| State persistence: survives reboots | [ ] | Reboot, check state.json |
| Manual override: `--dry-run`, `--force` | [ ] | Run both flags |

### Quality Gates

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| [Domain-specific quality 1] | [target] | - | [ ] |
| [Domain-specific quality 2] | [target] | - | [ ] |
| [Domain-specific quality 3] | [target] | - | [ ] |
| Automation reliability | 95%+ | - | [ ] |

### End-to-End Verification

```bash
# Full test (should complete without errors):
python -m src.scheduler run --dry-run

# Expected: [Description of expected outcome]
# Verify: data/state.json updated, logs written, no errors
```

---

## 14. Risk Register

### Active Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| [API provider downtime] | Low | High | Retry logic, fallback provider |
| [Cost overrun] | Medium | Medium | Alerts, budgets, monitoring |
| [Rate limiting] | Medium | Low | Exponential backoff, caching |
| [Platform policy changes] | Low | High | [Mitigation strategy] |
| Mac sleep during run | Low | Low | launchd runs on wake |

### Retired Risks

| Risk | Resolution |
|------|------------|
| [PLACEHOLDER] | [How it was resolved] |

---

## 15. Roadmap & What Remains

### Version Evolution

```
v1.0 (MVP)              v1.5 (Enhanced)         v2.0 (Intelligent)
                                                
Core pipeline           + [Enhancement 1]       + Agent Core
Basic CLI               + [Enhancement 2]       + Self-learning
Scheduler               + [Enhancement 3]       + Auto-optimize
Error handling          + Cost optimization     + Dashboard
```

### v1.0 -- Core Agent

```
+---------------------------------------------------------------------+
| v1.0: [Core Functionality]                               [STATUS]   |
+---------------------------------------------------------------------+
| [STATUS] [Core Feature 1]                                           |
| [STATUS] [Core Feature 2]                                           |
| [STATUS] [Core Feature 3]                                           |
| [STATUS] Daily automation                                           |
| [STATUS] Error handling & notifications                             |
| [STATUS] State persistence & cost tracking                          |
+---------------------------------------------------------------------+
```

### v1.5 -- Enhancements

```
+---------------------------------------------------------------------+
| v1.5: [Enhancement Theme]                            [TODO] Planned |
|       Estimated: [X-Y] hours                                        |
+---------------------------------------------------------------------+
| [Enhancement 1]                                                     |
| [Enhancement 2]                                                     |
| [Enhancement 3]                                                     |
| [Enhancement 4]                                                     |
+---------------------------------------------------------------------+
```

### v2.0 -- Agent Intelligence

```
+---------------------------------------------------------------------+
| v2.0: Self-Optimizing Agent                            [TODO] Future|
|       Estimated: [X-Y] hours                                        |
|       Prerequisite: [What must happen first]                        |
+---------------------------------------------------------------------+
| OBSERVE -> DECIDE -> ACT -> LEARN loop                              |
| Performance-based optimization                                      |
| Self-improvement capabilities                                       |
| Dashboard/reporting                                                 |
+---------------------------------------------------------------------+
```

### What Remains (Next Priority)

**Session [N]: [Feature Name]**
**Priority:** [HIGH/MEDIUM/LOW] | **Effort:** [X-Y] hours

```
[TODO] [Component 1]
       - [Detail]
       - [Detail]

[TODO] [Component 2]
       - [Detail]
       - [Detail]
```

---

## 16. Operations Guide

### Daily Operations

**The system runs automatically. No daily action required.**

Optional monitoring:

```bash
# Check status
python -m src.scheduler status

# View logs
tail -f ~/Library/Logs/[ProjectName]/stdout.log

# Health check
./scripts/health_check.sh
```

### Commands Reference

```bash
# Main execution
python -m src.scheduler run

# Dry run (no side effects)
python -m src.scheduler run --dry-run

# Force run (skip "already ran today")
python -m src.scheduler run --force

# Status check
python -m src.scheduler status

# Reset state (recovery)
python -m src.scheduler reset --confirm

# Scheduler management
launchctl list | grep [projectid]              # Check if loaded
launchctl start com.[projectid].daily          # Manually trigger
./scripts/install_scheduler.sh                 # Install
./scripts/uninstall_scheduler.sh               # Uninstall
```

### Log Locations

| Log | Location | Purpose |
|-----|----------|---------|
| Scheduler stdout | `~/Library/Logs/[ProjectName]/stdout.log` | Main output |
| Scheduler stderr | `~/Library/Logs/[ProjectName]/stderr.log` | Errors |
| Wrapper log | `data/logs/wrapper.log` | Environment setup |
| Application log | `data/logs/agent.log` | Detailed operations |
| Production log | `data/logs/production.jsonl` | Completed runs |

### Troubleshooting

| Problem | Check | Fix |
|---------|-------|-----|
| Scheduler not running | `launchctl list \| grep [projectid]` | `./scripts/install_scheduler.sh` |
| Lock file stuck | `ls data/.lock` | `rm data/.lock` |
| State corrupted | Check `data/state.json` | `python -m src.scheduler reset --confirm` |
| No notifications | System Settings -> Notifications | Enable for Terminal/Python |
| API failures | Check stderr.log | Verify API keys in .env |

---

## 17. Quick Reference

### Essential Commands

```bash
# Check system status
python -m src.scheduler status

# Manual run (full)
python -m src.scheduler run

# Manual run (no side effects)
python -m src.scheduler run --dry-run

# View logs
tail -f ~/Library/Logs/[ProjectName]/stdout.log

# Health check
./scripts/health_check.sh
```

### Key Files

```
config/default.yaml          # Main configuration
.env                         # API keys
src/scheduler.py             # Scheduler entry point
src/agent/runner.py          # Agent task logic
data/state.json              # Persistent state
data/logs/production.jsonl   # Run history
```

### Costs at a Glance

```
Per Unit:   $X.XX
Per Month:  ~$XXX (N units)
Per Year:   ~$X,XXX
```

### Schedule

```
Daily @ [TIME] [TIMEZONE]
[Rotation info if applicable]
```

### Support & Resources

- **[Service A]:** [support link]
- **[Service B]:** [support link]
- **Documentation:** `docs/`

---

## 18. Lessons Learned

*Updated after each session and at project completion.*

### What Went Well

| Area | Detail |
|------|--------|
| [PLACEHOLDER] | [What worked and why] |

### What Could Be Improved

| Area | Detail | Action for Next Project |
|------|--------|------------------------|
| [PLACEHOLDER] | [What didn't work] | [What to do differently] |

### Key Insights

- [PLACEHOLDER: Insight that applies to future projects]
- [PLACEHOLDER: Unexpected discovery worth remembering]

---

## Document Metadata

| Field | Value |
|-------|-------|
| Template Version | 1.0.0 |
| Created | [DATE] |
| Last Updated | [DATE] |
| Status | [ ] Planning / [WIP] Building / [DONE] Complete] |
| Next Milestone | [What's next] |
| Author | [YOUR_NAME] + Claude |

---

> *"The people who are crazy enough to think they can change the world are the ones who do."* -- Steve Jobs

---

**End of Project Document**
