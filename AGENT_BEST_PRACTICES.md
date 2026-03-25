# Agent Development & Distribution Checklist

A step-by-step checklist covering best practices from agent creation to shipping a distributable product.

---

## Explicit Non-Goals

Before building, clarify what the agent is NOT responsible for. This prevents scope creep and protects against misuse.

**Agents built with this framework do NOT:**

- Self-modify code without explicit instruction
- Persist personal user data beyond execution context
- Make autonomous decisions outside defined boundaries
- Guarantee external platform acceptance or outcomes
- Replace human judgment on critical decisions

**Customize this list per project** -- add domain-specific non-goals in your project document.

---

## Section 1 -- Agent Foundations

- [ ] Define the reasoning loop (decide -> act -> update state)
- [ ] Separate state from logic (no global variables)
- [ ] Keep tools dumb (execute actions, no reasoning)
- [ ] Establish a clean interface boundary (agent unaware of UI)
- [ ] Implement a single callable entry point (`run(input)`)
- [ ] Ensure agent is language-agnostic if cross-platform is intended

### Quick Check

- Can the agent run independently of the interface?
- Can the agent run repeatedly without errors?
- Are state and tools injected rather than hard-coded?

---

## Section 2 -- Separating Concerns

- [ ] Core Agent: owns reasoning, state updates, decision-making
- [ ] Tools Layer: executes tasks, independent of agent logic
- [ ] Interface Layer: handles user input/output, replaceable
- [ ] Test layer replacement (swap UI without changing agent logic)

### Quick Check

- Does the agent survive UI swaps?
- Are tools independent and unit-testable?
- Are reasoning and execution fully separated?

---

## Section 3 -- Configuration & Secrets

- [ ] Define all configuration variables (paths, flags, features)
- [ ] Identify secrets (API keys, tokens, credentials)
- [ ] Load configuration at runtime (not import time)
- [ ] Use environment variables, config files, or runtime arguments
- [ ] Avoid shipping secrets in packaged apps
- [ ] Provide mechanisms for user-supplied secrets if necessary

### Quick Check

- Can the agent run in multiple environments without modification?
- Are secrets safe and not exposed?
- Can configuration be adjusted per user/environment?

---

## Section 4 -- Making the Agent Callable

- [ ] Single entry point (`run(input)`)
- [ ] Structured input and output (no printing/logging in logic)
- [ ] Support asynchronous operations
- [ ] Inject state and tools for flexibility
- [ ] Test callable agent across CLI, web, desktop
- [ ] Ensure agent returns consistent, predictable outputs

### Quick Check

- Can the agent run headless for UI integration?
- Can different interfaces call the agent without changing logic?
- Are outputs structured and easy to consume by UIs?

---

## Section 5 -- Version Control & Commit Discipline

Commits are treated as first-class artifacts. Each commit represents a meaningful, reversible state transition in the agent's development lifecycle.

### Commit Principles

All commits MUST be:

- [ ] **Atomic** -- One logical change
- [ ] **Reversible** -- Safe to revert
- [ ] **Build-safe** -- Code runs and tests pass
- [ ] **Intentional** -- Purpose clear from message alone
- [ ] **Traceable** -- Maps to a lifecycle step, tool, or contract item

If a commit cannot be reverted cleanly, it is too large.

### Commit Scope Rules

| Rule | Description |
|------|-------------|
| One behavior per commit | No mixed concerns |
| One subsystem per commit | Avoid cross-cutting changes |
| No partial implementations | Stubs must be explicit |
| No broken invariants | Agent contract must still hold |

### Commit Message Format (Required)

Commits MUST follow a lightweight conventional format:

```
<type>(scope): short summary
```

**Allowed Types:**

| Type | Usage |
|------|-------|
| `feat` | New behavior or capability |
| `fix` | Bug or defect |
| `refactor` | No behavior change |
| `test` | Tests only |
| `docs` | Documentation only |
| `chore` | Tooling, config, cleanup |
| `release` | Versioned release commit |

**Examples:**
```
feat(video): add clip cache to prevent duplicate API calls
fix(config): fail fast when API keys are missing
refactor(models): normalize video duration fields
docs: add distribution constraints
release: v0.3.0
```

### Commit Gates (Non-Negotiable)

An agent MUST NOT commit unless:

- [ ] Code executes without runtime errors
- [ ] Tests pass (or skips are documented)
- [ ] No secrets are added
- [ ] Agent contract invariants still hold
- [ ] State transitions remain valid

Agents must explicitly report when a commit is blocked.

### Commits as Lifecycle Transitions

Commits are the mechanism by which lifecycle progress is recorded.

| Lifecycle Phase | Expected Commit Types |
|-----------------|----------------------|
| Design | `docs` |
| Implementation | `feat` |
| Stabilization | `fix` |
| Hardening | `refactor`, `test` |
| Release | `release`, `chore` |

Each lifecycle transition SHOULD be visible in git history.

### Agent-Specific Commit Rules (Claude Code)

When operating autonomously, agents MUST:

- [ ] Commit only when instructed or when a lifecycle step completes
- [ ] Never bundle unrelated tool changes
- [ ] Never commit failing or speculative code
- [ ] Explicitly name the commit message before executing it

**Example instruction:**
```
"Commit with message feat(video): implement video compositor"
```

### Prohibited Commit Content

Agents MUST NEVER commit:

- API keys or credentials
- `.env` files
- Generated artifacts (binaries, build outputs)
- Temporary debug logs
- Unvalidated experimental code

Violations are treated as terminal failures.

### Release Commits & Tags

- [ ] Every distributable version requires a release commit
- [ ] Releases MUST be tagged (e.g., `v0.3.0`)
- [ ] Release commits MUST NOT include functional changes

```
release: v1.0.0

- Feature A implemented
- Feature B working
```

Then: `git tag v1.0.0`

### Quick Check

- Is every commit atomic and reversible?
- Do commit messages follow the conventional format?
- Are commits mapped to lifecycle phases?
- Does Claude Code have explicit commit instructions?
- Are release versions tagged?

---

## Section 6 -- Packaging & Distribution

- [ ] Choose packaging method (Electron, Tauri, web, hybrid)
- [ ] Keep agent running in background process if desktop
- [ ] Separate UI from agent during packaging
- [ ] Plan for installers, signing, and permissions
- [ ] Handle versioning and updates cleanly
- [ ] Abstract OS-specific paths and system calls
- [ ] Test installation on all supported platforms

### Quick Check

- Can someone install and run the agent without dev tools?
- Is the agent isolated from UI and system specifics?
- Are updates and versioning manageable?

### Distribution Upgrade Path

Plan for evolution from development to commercial distribution:

| Phase | Execution Model | Characteristics |
|-------|-----------------|-----------------|
| Local dev | CLI + local filesystem | Fast iteration, full access |
| Power user | Desktop app (Electron/Tauri) | Packaged, self-contained |
| Team/Beta | Shared config, local execution | Multi-user, same codebase |
| Commercial | Backend-mediated APIs | Centralized, scalable, metered |

Design decisions in early phases should not block later phases.

---

## Section 7 -- User & Product Considerations

- [ ] Support per-user configurations and profiles
- [ ] Implement access control, licensing, or feature limits
- [ ] Provide error reporting and observability (ethically)
- [ ] Ensure logs are useful for debugging
- [ ] Test agent behavior under unexpected inputs and failures

### Logging & Privacy Rules

Logs are essential for debugging but must respect privacy and security:

**NEVER log:**
- API keys, tokens, or credentials (even partially)
- Raw API responses containing user data
- Full file paths that reveal system structure
- Personal identifiable information (PII)

**ALWAYS:**
- Hash or truncate identifiers when persisting logs
- Use log levels appropriately (DEBUG vs INFO vs ERROR)
- Implement log rotation to prevent unbounded growth
- Make log verbosity configurable

**Example safe logging:**
```python
# Bad: logger.info(f"API response: {response.json()}")
# Good: logger.info(f"API call succeeded, status={response.status_code}")

# Bad: logger.debug(f"Processing user {user_email}")  
# Good: logger.debug(f"Processing user {hash(user_id)[:8]}")
```

### Operational Controls

Production agents need operational controls for maintenance and emergencies:

- [ ] **Enable/Disable toggle** -- Pause agent without uninstalling
- [ ] **Dry-run mode** -- Test execution without side effects
- [ ] **Force mode** -- Bypass safeguards when needed (with caution)
- [ ] **Status command** -- Show current state and health
- [ ] **Reset command** -- Clear state to defaults (with confirmation)

**Enable/Disable Pattern:**

```python
DISABLED_FILE = DATA_DIR / ".disabled"

def run():
    if DISABLED_FILE.exists():
        print("[PAUSED] Agent is DISABLED")
        return 0  # Clean exit, don't restart
    # ... normal execution
```

**Why this matters:**
- Debugging: Pause while investigating issues
- Maintenance: Stop runs during updates
- Cost control: Temporarily halt API spend
- Testing: Disable prod while testing dev

### Quick Check

- Can users configure the agent safely?
- Is support feasible for multiple users?
- Is the agent resilient under edge cases?
- Can the agent be paused without uninstalling?

---

## Section 8 -- Common Mistakes to Avoid

- Coupling logic to UI
- Stateless agents for multi-step tasks
- Tools making decisions instead of executing
- Hard-coded configs or secrets
- Printing instead of returning structured output
- Ignoring asynchronous or long-running task requirements
- Committing broken or speculative code
- Bundling unrelated changes in one commit
- Missing commit gates (tests, secrets check)
- Vague commit messages ("fix stuff", "WIP")

### Final Check Before Shipping

- Can your agent survive UI changes, packaging, and multiple environments?
- Are secrets secure and configuration flexible?
- Is the agent fully callable and testable?
- Have you avoided the common pitfalls listed above?
- Is every commit atomic, reversible, and properly messaged?
- Are releases tagged and immutable?

---

## Success Criteria Standards

All success criteria in handoffs and project documents MUST be:

| Property | Requirement |
|----------|-------------|
| **Testable** | Can be verified with a command, observation, or measurement |
| **Observable** | Outcome is visible (file created, output returned, state changed) |
| **Binary** | Clear yes/no answer -- no ambiguity |
| **Independent** | Can be verified without other criteria passing first |

**Good Example:**
```
- [ ] `python -m src.scheduler run --dry-run` exits with code 0
- [ ] `data/state.json` contains `last_run` timestamp after execution
- [ ] macOS notification appears on success
```

**Bad Example:**
```
- [ ] Scheduler works correctly
- [ ] Code is clean and well-structured
- [ ] Performance is acceptable
```

---

## Companion Documents

This checklist works with the following templates:

| Document | Purpose | When to Use |
|----------|---------|-------------|
| `AGENT_PROJECT_TEMPLATE.md` | Master project document structure | Start of every project |
| `SCHEDULER_TEMPLATE.md` | macOS launchd automation | When agent needs scheduling |
| `CLEANUP_TEMPLATE.md` | Automated file cleanup | When agent generates >1GB/year of files |
| `HANDOFF_TEMPLATE.md` | Session planning (intent) | Before each build session |
| `CLAUDE_CODE_PROTOCOL.md` | Claude Code execution rules | During implementation |
| `PIPELINE_AUDIT_TEMPLATE.md` | Codebase/pipeline audit framework | Before bug-fix sessions |
| `PROJECT_INSTRUCTIONS.md` | Workflow and role definitions | Project setup |

All session documentation should be stored in version-scoped folders: `docs/vX.X/`

---

## Glossary

| Term | Definition | Used In |
|------|------------|---------|
| **Task** | A unit of work to be completed in a session | CLAUDE_CODE_PROTOCOL, task checklists |
| **Deliverable** | A completed output reported in implementation summary | IMPLEMENTATION_SUMMARY |
| **Success Criterion** | A testable, observable condition that defines "done" | HANDOFF_TEMPLATE |
| **[!] Marker** | Indicates a blocked or partially completed task | Task checklists |
| **DONE** | Deliverable fully completed as specified | Status reporting |
| **PARTIAL** | Deliverable partially completed with explanation | Status reporting |
| **BLOCKED** | Deliverable cannot proceed due to dependency/issue | Status reporting |
| **SKIPPED** | Deliverable intentionally not implemented | Status reporting |
| **Plan Mode** | Phase where Claude Code proposes approach before implementing | CLAUDE_CODE_PROTOCOL |
| **Handoff** | Document defining what to build (intent, not implementation) | Session workflow |
| **Implementation Summary** | Document recording what was actually built | Session workflow |

---

**Usage:** Follow this checklist as you build, refactor, and prepare your agent for distribution. Tick off each item as you verify compliance.
