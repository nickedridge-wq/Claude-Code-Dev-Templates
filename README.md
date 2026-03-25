# Agent Foundation Kit

A structured framework for building autonomous AI agents with Claude and Claude Code.

---

## Overview

The Agent Foundation Kit provides the planning system, execution protocols, and production infrastructure needed to build AI agents consistently and reliably. It eliminates the overhead of recreating project structure, documentation standards, and execution rules from scratch for each new project.

The framework is built around three clearly defined roles:

- **Claude Chat** — strategic planning, architecture decisions, research, and handoff documents
- **Claude Code** — implementation, testing, commits, and session documentation
- **You** — requirements, constraints, and final approval

Every session produces structured documentation. Every agent follows the same architecture. Context is preserved across sessions.

---

## Quick Start

1. Load all files into a Claude Project as knowledge files
2. Describe your agent idea — what it does, who it's for, what services it uses
3. Claude generates your project document from `AGENT_PROJECT_TEMPLATE.md`, covering architecture, costs, risks, and roadmap
4. Claude specs out your agent using the spec templates — capabilities, configuration, tool definitions, and test matrix — before any code is written
5. When ready to build, Claude writes the handoff document from `HANDOFF_TEMPLATE.md` for your review and approval
6. Pass the handoff and `CLAUDE_CODE_PROTOCOL.md` to Claude Code — it handles plan mode, implementation, and session closure automatically
7. For scheduled agents, Claude adapts `SCHEDULER_TEMPLATE.md` to your project
8. For agents that generate accumulating files, Claude configures `CLEANUP_TEMPLATE.md`
9. Before bug-fix sessions, upload `PIPELINE_AUDIT_TEMPLATE.md` to Claude Code for a structured audit — the output becomes the input for a focused fix handoff

---

## File Map

### Start Here

| File | Purpose | When to Use |
|------|---------|-------------|
| **PROJECT_INSTRUCTIONS.md** | Defines the three-role workflow and working style for Claude Chat | Load as your Claude Chat system prompt at project start |
| **AGENT_PROJECT_TEMPLATE.md** | Master project document — architecture, costs, risks, roadmap, session log | Claude generates this from your idea at the start of every new agent project |

### Planning & Execution

| File | Purpose | When to Use |
|------|---------|-------------|
| **HANDOFF_TEMPLATE.md** | Standardized format for session planning (handoff) and session closure (implementation summary) | Claude writes a handoff before every build session; Claude Code writes the implementation summary after |
| **CLAUDE_CODE_PROTOCOL.md** | Execution rules for Claude Code — plan mode, task tracking, commit standards, session lifecycle | Include with every handoff to Claude Code |
| **AGENT_BEST_PRACTICES.md** | Development standards — architecture, separation of concerns, commit discipline, packaging, security, logging | Claude references this when generating project documents and handoffs |
| **PIPELINE_AUDIT_TEMPLATE.md** | Structured audit framework — bug pattern classification, severity tiers, audit-to-fix workflow | Upload to Claude Code before bug-fix sessions to produce a structured audit report |

### Agent Spec Templates

| File | Purpose | When to Use |
|------|---------|-------------|
| **AGENT_CAPABILITY_SPECS_TEMPLATE.md** | Defines agent purpose, inputs/outputs, reasoning loop, and success criteria | Claude fills this in when speccing a new agent's capabilities |
| **AGENT_CONFIG_AND_SECRETS_TEMPLATE.md** | Environment variables, runtime limits, feature flags, secrets management | Claude generates this when the agent requires external services or API keys |
| **AGENT_TOOL_DEF_TEMPLATE.md** | Tool inventory — interfaces, schemas, error modes | Claude fills this in when defining the tools the agent will use |
| **AGENT_TEST_MATRIX_TEMPLATE.md** | Structured test cases with edge cases and verification criteria | Claude builds the test plan from the capability specs |

### Production Infrastructure

| File | Purpose | When to Use |
|------|---------|-------------|
| **SCHEDULER_TEMPLATE.md** | macOS scheduling with launchd — Python scheduler, state persistence, retry logic, emergency brake, install/uninstall scripts | Claude adapts this when the agent needs to run automatically on a schedule |
| **CLEANUP_TEMPLATE.md** | Automated file cleanup — retention policies, protected paths, dry-run mode | Claude configures this when the agent generates files that accumulate over time |

---

## How the Files Connect

```
You describe your agent idea to Claude
            |
            v
  Claude generates your
  AGENT_PROJECT_TEMPLATE  <--- architecture, costs, risks,
            |                   roadmap (referencing
            |                    AGENT_BEST_PRACTICES)
            v
  Claude specs out the agent
  BEFORE the first build session:
  - CAPABILITY_SPECS -----> what it does, inputs/outputs, success criteria
  - CONFIG & SECRETS -----> env vars, runtime limits, API keys
  - TOOL DEFINITIONS -----> tools it uses, interfaces, error modes
  - TEST MATRIX ----------> test cases, edge cases, verification
            |
            v
  You plan the first build session
  with Claude, who writes
  the HANDOFF document     ---> ready for Claude Code
            |
            v
  Hand off to Claude Code
  with CLAUDE_CODE_PROTOCOL ---> Claude Code enters plan mode,
            |                     proposes approach, builds,
            |                     writes implementation summary
            v
  Need scheduling? -----> Claude adapts SCHEDULER_TEMPLATE
  Need cleanup? --------> Claude adapts CLEANUP_TEMPLATE
  Need an audit? -------> Upload PIPELINE_AUDIT_TEMPLATE to Claude Code
            |               Claude Code outputs audit report
            |               Take report to Claude Chat -> bug-fix handoff
            v
  Claude updates AGENT_PROJECT_TEMPLATE
  with session results, decisions, lessons
            |
            v
  Repeat for next session
```

---

## Session Workflow

Every build session follows this pattern:

1. **Plan** (You + Claude Chat) — Discuss what to build. Claude researches options, proposes architecture, and writes the handoff document. You approve.
2. **Build** (Claude Code) — Reads the handoff and protocol, enters plan mode, proposes an implementation plan, waits for approval, then implements and tracks tasks.
3. **Verify** (You) — Review output, test behaviour, iterate.
4. **Close** (Claude Code) — Writes the implementation summary, commits, and updates the project document.

Session documents are stored in version-scoped folders:

```
docs/
├── v1.0/
│   ├── V1.0_PROJECT_DOCUMENT.md
│   ├── V1.0_SESSION_01_HANDOFF.md
│   ├── V1.0_SESSION_01_IMPLEMENTATION_SUMMARY.md
│   ├── V1.0_SESSION_02_HANDOFF.md
│   └── V1.0_SESSION_02_IMPLEMENTATION_SUMMARY.md
└── v1.5/
    ├── V1.5_PROJECT_DOCUMENT.md
    └── V1.5_SESSION_01_HANDOFF.md
```

---

## Requirements

- **Claude Pro** with Claude Code access
- **macOS** for the scheduling and cleanup templates (all planning and protocol files are platform-agnostic)

---

## Usage Notes

- **Start with the project template.** It creates a shared context that prevents Claude from drifting between sessions. Describe your idea and let Claude generate it.
- **Handoffs define intent, not process.** Specify what must exist when the session is done — Claude Code determines how to build it.
- **The protocol is required with every handoff.** Plan mode, task tracking, and implementation summaries are what make the system reliable across sessions.
- **Use `--dry-run` before enabling scheduling.** Always validate the scheduler without side effects before running it live.
- **Audit before fixing bugs.** Upload `PIPELINE_AUDIT_TEMPLATE.md` to Claude Code for a structured analysis first — the audit report becomes the input for a focused, well-scoped fix session.

---

## License

MIT
