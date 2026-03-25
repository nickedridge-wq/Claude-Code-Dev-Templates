# macOS Scheduler Template

**Platform:** macOS (launchd)
**Use Case:** Automated task execution with state persistence
**Copy this template and fill in the `[PLACEHOLDERS]`**

---

## Quick Start Checklist

```
1. [ ] Copy this template to your project
2. [ ] Replace ALL [PLACEHOLDERS] with your values
3. [ ] Create the file structure below
4. [ ] Implement your task in execute_task()
5. [ ] Run: chmod +x scripts/*.sh
6. [ ] Run: ./scripts/install_scheduler.sh
7. [ ] Test: python -m src.scheduler run --dry-run
8. [ ] Verify: launchctl list | grep [YOUR_PROJECT]
```

### Placeholders to Replace

| Placeholder | Example | Your Value |
|-------------|---------|------------|
| `[PROJECT_NAME]` | `edge-content-agent` | __________ |
| `[PROJECT_ID]` | `edgecontentagent` | __________ |
| `[USERNAME]` | `yourname` | __________ |
| `[SCHEDULE_HOUR]` | `9` | __________ |
| `[SCHEDULE_MINUTE]` | `0` | __________ |
| `[ROTATION_ITEMS]` | `["niche_a", "niche_b"]` | __________ |

---

## Architecture

```
+---------------------------------------------------------------------+
|                         SCHEDULING LAYER                            |
|                                                                     |
|  +-----------------+    +-----------------+    +-----------------+  |
|  |    launchd      |--->|  run_daily.sh   |--->|  scheduler.py   |  |
|  |  (OS trigger)   |    |   (wrapper)     |    |   (Python)      |  |
|  +-----------------+    +-----------------+    +-----------------+  |
|         |                       |                      |            |
|         |                       |      +---------------+--------+   |
|    Triggers at          Safeguards:    |               v        |   |
|    scheduled time       * Lock check   |  +--------------------+|   |
|                         * Stale lock   |  |  State Management  ||   |
|                         * Exit codes   |  |  * state.json      ||   |
|                                        |  |  * .lock file      ||   |
|                                        |  |  * Daily counters  ||   |
|                                        |  +--------------------+|   |
|                                        +------------------------+   |
|                                                                     |
|  SAFEGUARDS (prevent runaway execution):                            |
|  [x] Already succeeded today -> skip                                |
|  [x] Retry limit (3/day) -> stop retrying                           |
|  [x] Emergency brake (10 runs/day) -> halt                          |
|  [x] Lock file with stale detection                                 |
|  [x] Smart exit codes (0 on skip, launchd won't restart)            |
+---------------------------------------------------------------------+
                                    |
                                    v
+---------------------------------------------------------------------+
|                    YOUR TASK (implement in execute_task)            |
+---------------------------------------------------------------------+
```

---

## File Structure

```
[PROJECT_NAME]/
|
+-- config/
|   +-- com.[PROJECT_ID].daily.plist    # launchd configuration
|
+-- src/
|   +-- scheduler.py                     # Main scheduler module
|
+-- scripts/
|   +-- run_daily.sh                     # Wrapper (launchd calls this)
|   +-- install_scheduler.sh             # Installation
|   +-- uninstall_scheduler.sh           # Removal
|
+-- data/                                # Git-ignored
|   +-- state.json                       # Persistent state
|   +-- .lock                            # Concurrency lock
|   +-- .disabled                        # If exists, agent is paused (touch to disable)
|   +-- logs/
|       +-- scheduler.log
|
+-- .env                                 # API keys (git-ignored)
+-- requirements.txt
```

---

## File 1: launchd plist

**Path:** `config/com.[PROJECT_ID].daily.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.[PROJECT_ID].daily</string>
    
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/[USERNAME]/[PROJECT_NAME]/scripts/run_daily.sh</string>
    </array>
    
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>[SCHEDULE_HOUR]</integer>
        <key>Minute</key>
        <integer>[SCHEDULE_MINUTE]</integer>
    </dict>
    
    <key>StandardOutPath</key>
    <string>/Users/[USERNAME]/Library/Logs/[PROJECT_NAME]/stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/[USERNAME]/Library/Logs/[PROJECT_NAME]/stderr.log</string>
    
    <key>WorkingDirectory</key>
    <string>/Users/[USERNAME]/[PROJECT_NAME]</string>
    
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin</string>
    </dict>
</dict>
</plist>
```

### WARNING: DANGEROUS plist Options -- DO NOT USE

These options can cause infinite loop / runaway execution:

```xml
<!-- NEVER USE: Causes restart on ANY exit, including intentional stops -->
<key>KeepAlive</key>
<true/>

<!-- NEVER USE: Restarts when exit code is non-zero (failures trigger loops) -->
<key>KeepAlive</key>
<dict>
    <key>SuccessfulExit</key>
    <false/>
</dict>

<!-- AVOID: Can cause unexpected restart behavior -->
<key>RunAtLoad</key>
<true/>

<!-- AVOID: Can interact badly with retry logic -->
<key>ThrottleInterval</key>
<integer>...</integer>
```

**Safe plist pattern:** Use ONLY `StartCalendarInterval` for scheduling. Let the Python scheduler handle retries internally.

### Schedule Options Reference

```xml
<!-- OPTION A: Daily at specific time (RECOMMENDED) -->
<key>StartCalendarInterval</key>
<dict>
    <key>Hour</key>
    <integer>9</integer>
    <key>Minute</key>
    <integer>0</integer>
</dict>

<!-- OPTION B: Multiple times per day -->
<key>StartCalendarInterval</key>
<array>
    <dict><key>Hour</key><integer>9</integer><key>Minute</key><integer>0</integer></dict>
    <dict><key>Hour</key><integer>17</integer><key>Minute</key><integer>0</integer></dict>
</array>

<!-- OPTION C: Every N seconds (use with caution - ensure safeguards) -->
<key>StartInterval</key>
<integer>3600</integer>

<!-- OPTION D: Specific weekday (0=Sunday, 1=Monday...) -->
<key>StartCalendarInterval</key>
<dict>
    <key>Weekday</key>
    <integer>1</integer>
    <key>Hour</key>
    <integer>9</integer>
</dict>

<!-- OPTION E: Monthly (1st of month) -->
<key>StartCalendarInterval</key>
<dict>
    <key>Day</key>
    <integer>1</integer>
    <key>Hour</key>
    <integer>9</integer>
</dict>
```

---

## File 2: Wrapper Script

**Path:** `scripts/run_daily.sh`

```bash
#!/bin/bash
#
# Wrapper script for launchd
# Sets up environment and runs Python scheduler with safeguards
#
# SAFEGUARDS:
# 1. Lock file check (prevents concurrent runs)
# 2. Stale lock detection (auto-clears locks older than 1 hour)
# 3. Smart exit codes (exit 0 on skip so launchd doesn't restart)
#
# CUSTOMIZE: Update PROJECT_DIR and VENV_PATH for your project

set -e

# ============================================
# CONFIGURATION - UPDATE THESE
# ============================================
PROJECT_DIR="/Users/[USERNAME]/[PROJECT_NAME]"
VENV_PATH="$PROJECT_DIR/venv"
LOG_FILE="$PROJECT_DIR/data/logs/wrapper.log"
LOCK_FILE="$PROJECT_DIR/data/.lock"
STALE_LOCK_MINUTES=60

# ============================================
# SETUP
# ============================================
mkdir -p "$(dirname "$LOG_FILE")"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

cleanup() {
    # Lock is managed by Python, but ensure we log exit
    log "Wrapper exiting"
}
trap cleanup EXIT

log "========================================"
log "Wrapper started"

cd "$PROJECT_DIR"
log "Working directory: $PROJECT_DIR"

# ============================================
# SAFEGUARD: Stale lock detection
# ============================================
if [ -f "$LOCK_FILE" ]; then
    # Check if lock is stale (older than STALE_LOCK_MINUTES)
    if [ "$(find "$LOCK_FILE" -mmin +$STALE_LOCK_MINUTES 2>/dev/null)" ]; then
        log "WARNING: Removing stale lock file (older than $STALE_LOCK_MINUTES minutes)"
        rm -f "$LOCK_FILE"
    fi
fi

# ============================================
# ACTIVATE VIRTUAL ENVIRONMENT
# ============================================
if [ -f "$VENV_PATH/bin/activate" ]; then
    source "$VENV_PATH/bin/activate"
    log "Virtual environment activated"
else
    log "ERROR: venv not found at $VENV_PATH"
    exit 1
fi

# ============================================
# LOAD ENVIRONMENT VARIABLES
# ============================================
if [ -f "$PROJECT_DIR/.env" ]; then
    set -a
    source "$PROJECT_DIR/.env"
    set +a
    log "Environment variables loaded from .env"
fi

# ============================================
# RUN SCHEDULER
# ============================================
log "Starting scheduler..."
python -m src.scheduler run 2>&1 | tee -a "$LOG_FILE"

EXIT_CODE=${PIPESTATUS[0]}
log "Scheduler exited with code: $EXIT_CODE"
log "========================================"

# IMPORTANT: Exit codes
# 0 = Success OR intentional skip (launchd will NOT restart)
# 1 = Error (launchd will NOT restart because we don't use KeepAlive)
exit $EXIT_CODE
```

---

## File 3: Python Scheduler

**Path:** `src/scheduler.py`

```python
#!/usr/bin/env python3
"""
Generic Scheduler Template with Runaway Execution Safeguards

Features:
- Lock mechanism (prevents concurrent runs)
- State persistence (JSON)
- Retry logic with daily limits
- macOS notifications
- Optional rotation logic

SAFEGUARDS (prevent infinite loops):
- Already succeeded today check
- Daily retry limit (default: 3)
- Emergency brake (default: 10 total runs/day)
- Smart exit codes

CUSTOMIZE:
1. Update ROTATION_ITEMS (or remove if not needed)
2. Implement execute_task() with your actual logic
"""

import argparse
import fcntl
import json
import subprocess
import sys
import time
from dataclasses import dataclass, asdict, field
from datetime import datetime, date
from pathlib import Path
from typing import Optional, Tuple, Dict, Any, List


# ============================================================================
# CONFIGURATION - UPDATE THESE
# ============================================================================

PROJECT_ROOT = Path(__file__).parent.parent
DATA_DIR = PROJECT_ROOT / "data"
STATE_FILE = DATA_DIR / "state.json"
LOCK_FILE = DATA_DIR / ".lock"
DISABLED_FILE = DATA_DIR / ".disabled"  # If exists, agent is paused

# Retry settings
MAX_RETRIES = 3           # Max retries per day
RETRY_DELAY_SECONDS = 300  # 5 minutes between retries

# Emergency brake - if this many runs happen in one day, something is wrong
MAX_RUNS_PER_DAY = 10

# Rotation items (set to empty list if not using rotation)
ROTATION_ITEMS: List[str] = [
    "item_a",
    "item_b", 
    "item_c",
]


# ============================================================================
# STATE MANAGEMENT
# ============================================================================

@dataclass
class SchedulerState:
    """Persistent state across runs. Add/remove fields as needed."""
    
    # Core tracking
    last_run: Optional[str] = None
    last_success: Optional[str] = None
    run_count: int = 0
    success_count: int = 0
    failure_count: int = 0
    
    # Daily safeguard tracking
    last_success_date: Optional[str] = None   # ISO date: "2026-01-31"
    retries_today: int = 0
    retry_date: Optional[str] = None          # Which date the retries are for
    runs_today: int = 0
    runs_date: Optional[str] = None           # Which date the run count is for
    last_outcome: Optional[str] = None        # "success", "failure", "skipped"
    last_error: Optional[str] = None
    
    # Cost tracking (optional - remove if not needed)
    total_cost: float = 0.0
    
    # Rotation tracking (optional - remove if not using rotation)
    current_rotation_index: int = 0
    rotation_week: int = 0
    
    # Add your custom fields here:
    # custom_field: str = ""
    
    def already_succeeded_today(self) -> bool:
        """Check if we already had a successful run today."""
        if not self.last_success_date:
            return False
        return self.last_success_date == date.today().isoformat()
    
    def can_retry(self) -> bool:
        """Check if we can retry (haven't hit daily limit)."""
        today = date.today().isoformat()
        if self.retry_date != today:
            return True  # New day, can retry
        return self.retries_today < MAX_RETRIES
    
    def check_emergency_brake(self) -> bool:
        """Check if emergency brake should activate (too many runs today)."""
        today = date.today().isoformat()
        if self.runs_date != today:
            return False  # New day, brake not active
        return self.runs_today >= MAX_RUNS_PER_DAY
    
    def record_run_attempt(self) -> None:
        """Record that a run is being attempted."""
        today = date.today().isoformat()
        
        # Reset counters if new day
        if self.runs_date != today:
            self.runs_date = today
            self.runs_today = 0
        if self.retry_date != today:
            self.retry_date = today
            self.retries_today = 0
        
        self.runs_today += 1
        self.last_run = datetime.now().isoformat()
    
    def record_success(self) -> None:
        """Record a successful run."""
        self.last_success = datetime.now().isoformat()
        self.last_success_date = date.today().isoformat()
        self.last_outcome = "success"
        self.last_error = None
        self.success_count += 1
    
    def record_failure(self, error: str) -> None:
        """Record a failed run."""
        today = date.today().isoformat()
        if self.retry_date != today:
            self.retry_date = today
            self.retries_today = 0
        self.retries_today += 1
        self.last_outcome = "failure"
        self.last_error = error
        self.failure_count += 1
    
    def record_skip(self, reason: str) -> None:
        """Record a skipped run."""
        self.last_outcome = "skipped"
        self.last_error = reason
    
    def save(self) -> None:
        """Persist state to disk."""
        DATA_DIR.mkdir(parents=True, exist_ok=True)
        with open(STATE_FILE, 'w') as f:
            json.dump(asdict(self), f, indent=2)
    
    @classmethod
    def load(cls) -> "SchedulerState":
        """Load state from disk, or create fresh if missing/corrupted."""
        if STATE_FILE.exists():
            try:
                with open(STATE_FILE, 'r') as f:
                    data = json.load(f)
                # Filter to only known fields (handles schema changes)
                known_fields = {f.name for f in cls.__dataclass_fields__.values()}
                filtered = {k: v for k, v in data.items() if k in known_fields}
                return cls(**filtered)
            except (json.JSONDecodeError, TypeError) as e:
                print(f"Warning: Could not load state: {e}")
        return cls()


# ============================================================================
# LOCK MECHANISM
# ============================================================================

class LockError(Exception):
    """Raised when lock cannot be acquired."""
    pass


def acquire_lock():
    """
    Acquire exclusive lock using fcntl (kernel-level, atomic).
    Returns file descriptor on success.
    """
    DATA_DIR.mkdir(parents=True, exist_ok=True)
    fd = open(LOCK_FILE, 'w')
    try:
        fcntl.flock(fd.fileno(), fcntl.LOCK_EX | fcntl.LOCK_NB)
        fd.write(f"Locked at {datetime.now().isoformat()}\nPID: {subprocess.os.getpid()}")
        fd.flush()
        return fd
    except BlockingIOError:
        fd.close()
        raise LockError("Another instance is already running")


def release_lock(fd) -> None:
    """Release lock and clean up."""
    if fd:
        try:
            fcntl.flock(fd.fileno(), fcntl.LOCK_UN)
            fd.close()
            LOCK_FILE.unlink(missing_ok=True)
        except Exception:
            pass


# ============================================================================
# NOTIFICATIONS (macOS)
# ============================================================================

def notify(title: str, message: str, sound: bool = True) -> None:
    """Send macOS notification."""
    sound_cmd = 'sound name "default"' if sound else ""
    script = f'display notification "{message}" with title "{title}" {sound_cmd}'
    try:
        subprocess.run(['osascript', '-e', script], capture_output=True, timeout=5)
    except Exception:
        pass  # Notifications are nice-to-have, don't fail on error


# ============================================================================
# ROTATION LOGIC (Optional - delete if not needed)
# ============================================================================

def get_rotation_item(state: SchedulerState) -> Optional[str]:
    """
    Get current rotation item based on ISO week number.
    Rotates every Monday. Returns None if ROTATION_ITEMS is empty.
    """
    if not ROTATION_ITEMS:
        return None
    
    current_week = datetime.now().isocalendar()[1]
    
    if current_week != state.rotation_week:
        state.rotation_week = current_week
        state.current_rotation_index = (state.current_rotation_index + 1) % len(ROTATION_ITEMS)
        state.save()
    
    return ROTATION_ITEMS[state.current_rotation_index]


# ============================================================================
# YOUR TASK - IMPLEMENT THIS
# ============================================================================

def execute_task(rotation_item: Optional[str] = None, dry_run: bool = False) -> Dict[str, Any]:
    """
    *** IMPLEMENT YOUR ACTUAL TASK HERE ***
    
    Args:
        rotation_item: Current rotation item (if using rotation)
        dry_run: If True, simulate without side effects
    
    Returns:
        Dict with keys:
        - success: bool
        - cost: float (optional, for cost tracking)
        - data: any additional data
        - error: str (if failed)
    """
    
    if dry_run:
        print(f"[DRY RUN] Would execute task with rotation_item={rotation_item}")
        return {"success": True, "cost": 0.0, "data": {"dry_run": True}}
    
    try:
        # =============================================
        # YOUR IMPLEMENTATION HERE
        # =============================================
        # Example:
        # from src.your_module import YourTask
        # result = YourTask().run(rotation_item)
        # return {"success": True, "cost": result.cost, "data": result.data}
        
        # Placeholder - replace with your logic
        print(f"Executing task with rotation_item={rotation_item}")
        return {
            "success": True,
            "cost": 0.0,
            "data": {"placeholder": "Replace execute_task() with your implementation"}
        }
        
    except Exception as e:
        return {"success": False, "cost": 0.0, "error": str(e)}


# ============================================================================
# CORE EXECUTION WITH SAFEGUARDS
# ============================================================================

def run_with_retries(rotation_item: Optional[str], state: SchedulerState, dry_run: bool) -> Tuple[bool, Dict]:
    """Execute task with retry logic (respects daily retry limit)."""
    last_result = {}
    attempts = 0
    
    while attempts <= MAX_RETRIES:
        # Check retry limit before each attempt (except first)
        if attempts > 0:
            if not state.can_retry():
                print(f"Max retries ({MAX_RETRIES}) reached for today. Stopping.")
                return False, last_result
            print(f"Retry {attempts}/{MAX_RETRIES} in {RETRY_DELAY_SECONDS}s...")
            time.sleep(RETRY_DELAY_SECONDS)
        
        result = execute_task(rotation_item=rotation_item, dry_run=dry_run)
        last_result = result
        
        if result.get("success"):
            return True, result
        
        print(f"Attempt {attempts + 1} failed: {result.get('error', 'Unknown')}")
        
        # Record failure (increments retry counter)
        if not dry_run:
            state.record_failure(result.get('error', 'Unknown'))
            state.save()
        
        attempts += 1
    
    return False, last_result


def run(dry_run: bool = False, force: bool = False) -> int:
    """
    Main entry point with safeguards.
    
    Exit codes:
    - 0: Success OR intentional skip (launchd won't restart)
    - 1: Error (launchd won't restart because we don't use KeepAlive)
    """
    lock_fd = None
    
    try:
        # ========================================
        # SAFEGUARD: Disabled check (quick toggle)
        # ========================================
        if DISABLED_FILE.exists() and not force:
            print("[PAUSED] Agent is DISABLED. Use 'scheduler enable' to resume.")
            print(f"   (or delete {DISABLED_FILE})")
            return 0  # Exit 0 so launchd doesn't restart
        
        # 1. Acquire lock
        print("Acquiring lock...")
        lock_fd = acquire_lock()
        
        # 2. Load state
        state = SchedulerState.load()
        print(f"Loaded state: {state.run_count} total runs, {state.runs_today} today")
        
        # ========================================
        # SAFEGUARD: Emergency brake
        # ========================================
        if state.check_emergency_brake():
            msg = f"EMERGENCY BRAKE: {state.runs_today} runs today (max: {MAX_RUNS_PER_DAY})"
            print(f"[ALERT] {msg}")
            print("Something is wrong. Investigate before resetting.")
            state.record_skip(msg)
            state.save()
            notify("[ALERT] Emergency Brake", msg)
            return 0  # Exit 0 so launchd doesn't restart
        
        # ========================================
        # SAFEGUARD: Already succeeded today
        # ========================================
        if not force and state.already_succeeded_today():
            msg = "Already succeeded today"
            print(f"[OK] {msg}. Use --force to override.")
            state.record_skip(msg)
            state.save()
            return 0  # Exit 0 so launchd doesn't restart
        
        # ========================================
        # SAFEGUARD: Retry limit
        # ========================================
        if not force and not state.can_retry():
            msg = f"Max retries ({MAX_RETRIES}) reached today"
            print(f"[WARN] {msg}. Waiting until tomorrow or use --force.")
            state.record_skip(msg)
            state.save()
            return 0  # Exit 0 so launchd doesn't restart
        
        # 3. Record run attempt
        state.record_run_attempt()
        state.run_count += 1
        state.save()
        
        # 4. Get rotation item (if applicable)
        rotation_item = get_rotation_item(state)
        if rotation_item:
            print(f"Rotation item: {rotation_item}")
        
        # 5. Execute with retries
        print("Executing task...")
        success, result = run_with_retries(rotation_item, state, dry_run)
        
        # 6. Update state based on outcome
        if success:
            state.record_success()
            state.total_cost += result.get("cost", 0.0)
            state.save()
            notify("[OK] Task Complete", "Scheduler ran successfully")
            print("[OK] Success!")
            return 0
        else:
            # Failure already recorded in run_with_retries
            error_msg = result.get("error", "Unknown error")[:50]
            notify("[FAIL] Task Failed", error_msg)
            print(f"[FAIL] Failed after {MAX_RETRIES + 1} attempts")
            return 1
        
    except LockError as e:
        print(f"Lock error: {e}")
        return 0  # Exit 0 - another instance running is not an error condition
    except Exception as e:
        print(f"Unexpected error: {e}")
        notify("[FAIL] Scheduler Error", str(e)[:50])
        return 1
    finally:
        release_lock(lock_fd)


# ============================================================================
# CLI COMMANDS
# ============================================================================

def cmd_status() -> int:
    """Display current status including safeguard state."""
    state = SchedulerState.load()
    today = date.today().isoformat()
    
    print("\n" + "=" * 50)
    print("SCHEDULER STATUS")
    print("=" * 50)
    print(f"Today:          {today}")
    print(f"Last Run:       {state.last_run or 'Never'}")
    print(f"Last Success:   {state.last_success or 'Never'}")
    print(f"Last Outcome:   {state.last_outcome or 'N/A'}")
    print(f"Runs:           {state.run_count} total, {state.success_count} succeeded, {state.failure_count} failed")
    print(f"Total Cost:     ${state.total_cost:.2f}")
    
    print("\n" + "-" * 50)
    print("SAFEGUARD STATUS")
    print("-" * 50)
    print(f"Runs Today:     {state.runs_today}/{MAX_RUNS_PER_DAY}")
    print(f"Retries Today:  {state.retries_today}/{MAX_RETRIES}")
    
    # Status indicators
    if DISABLED_FILE.exists():
        print("[PAUSED] DISABLED -- agent is paused, use 'scheduler enable' to resume")
    elif state.already_succeeded_today():
        print("[OK] Already succeeded today -- will skip if triggered again")
    elif not state.can_retry():
        print("[WARN] Max retries reached -- will skip until tomorrow")
    elif state.check_emergency_brake():
        print("[ALERT] EMERGENCY BRAKE ACTIVE -- too many runs today")
    else:
        print("[READY] Ready to run")
    
    if state.last_error:
        print(f"\nLast Error: {state.last_error}")
    
    if ROTATION_ITEMS:
        print(f"\nRotation:       {ROTATION_ITEMS[state.current_rotation_index]} (index {state.current_rotation_index})")
    
    print("=" * 50 + "\n")
    return 0


def cmd_reset(confirm: bool) -> int:
    """Reset state to defaults."""
    if not confirm:
        print("[WARN] This will reset all state including safeguard counters.")
        print("Use --confirm to proceed.")
        return 1
    SchedulerState().save()
    print("[OK] State reset to defaults.")
    return 0


def cmd_next_rotation() -> int:
    """Show current and next rotation items."""
    if not ROTATION_ITEMS:
        print("Rotation not configured (ROTATION_ITEMS is empty)")
        return 0
    
    state = SchedulerState.load()
    current = ROTATION_ITEMS[state.current_rotation_index]
    next_idx = (state.current_rotation_index + 1) % len(ROTATION_ITEMS)
    
    print(f"Current: {current}")
    print(f"Next:    {ROTATION_ITEMS[next_idx]}")
    return 0


def cmd_enable() -> int:
    """Enable the scheduler (remove .disabled file)."""
    if not DISABLED_FILE.exists():
        print("[OK] Scheduler is already enabled")
        return 0
    
    DISABLED_FILE.unlink()
    print("[OK] Scheduler ENABLED -- will run on next trigger")
    return 0


def cmd_disable() -> int:
    """Disable the scheduler (create .disabled file)."""
    if DISABLED_FILE.exists():
        print("[PAUSED] Scheduler is already disabled")
        return 0
    
    DATA_DIR.mkdir(parents=True, exist_ok=True)
    DISABLED_FILE.touch()
    print("[PAUSED] Scheduler DISABLED -- will skip all runs until enabled")
    print(f"   To re-enable: python -m src.scheduler enable")
    return 0


# ============================================================================
# MAIN
# ============================================================================

def main():
    parser = argparse.ArgumentParser(
        description="Task Scheduler with Safeguards",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  python -m src.scheduler run              # Normal execution
  python -m src.scheduler run --dry-run    # Test without side effects
  python -m src.scheduler run --force      # Bypass safeguards (use with caution)
  python -m src.scheduler status           # Show current state & safeguards
  python -m src.scheduler enable           # Enable scheduler (resume)
  python -m src.scheduler disable          # Disable scheduler (pause)
  python -m src.scheduler reset --confirm  # Reset all state
        """
    )
    
    subparsers = parser.add_subparsers(dest="command")
    
    # run
    p_run = subparsers.add_parser("run", help="Execute scheduled task")
    p_run.add_argument("--dry-run", action="store_true", help="Simulate without executing")
    p_run.add_argument("--force", action="store_true", help="Bypass safeguards (use with caution)")
    
    # status
    subparsers.add_parser("status", help="Show scheduler status")
    
    # enable/disable
    subparsers.add_parser("enable", help="Enable scheduler (remove pause)")
    subparsers.add_parser("disable", help="Disable scheduler (pause without uninstalling)")
    
    # reset
    p_reset = subparsers.add_parser("reset", help="Reset scheduler state")
    p_reset.add_argument("--confirm", action="store_true", help="Confirm reset")
    
    # next-rotation
    subparsers.add_parser("next-rotation", help="Show rotation info")
    
    args = parser.parse_args()
    
    commands = {
        "run": lambda: run(dry_run=args.dry_run, force=args.force),
        "status": cmd_status,
        "enable": cmd_enable,
        "disable": cmd_disable,
        "reset": lambda: cmd_reset(args.confirm),
        "next-rotation": cmd_next_rotation,
    }
    
    if args.command in commands:
        sys.exit(commands[args.command]())
    else:
        parser.print_help()
        sys.exit(0)


if __name__ == "__main__":
    main()
```

---

## File 4: Install Script

**Path:** `scripts/install_scheduler.sh`

```bash
#!/bin/bash
#
# Install launchd scheduler
#
# CUSTOMIZE: Update PROJECT_ID and PROJECT_NAME

set -e

# ============================================
# CONFIGURATION - UPDATE THESE
# ============================================
PROJECT_ID="[PROJECT_ID]"
PROJECT_NAME="[PROJECT_NAME]"

# ============================================
# DERIVED PATHS (usually don't need to change)
# ============================================
PROJECT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
PLIST_NAME="com.${PROJECT_ID}.daily"
PLIST_SOURCE="$PROJECT_DIR/config/$PLIST_NAME.plist"
PLIST_DEST="$HOME/Library/LaunchAgents/$PLIST_NAME.plist"
LOG_DIR="$HOME/Library/Logs/$PROJECT_NAME"

echo "[SETUP] Installing $PROJECT_NAME scheduler..."
echo ""

# Create directories
mkdir -p "$LOG_DIR"
mkdir -p "$PROJECT_DIR/data/logs"
echo "[OK] Created directories"

# Unload if exists
if launchctl list 2>/dev/null | grep -q "$PLIST_NAME"; then
    echo "  Unloading existing scheduler..."
    launchctl unload "$PLIST_DEST" 2>/dev/null || true
fi

# Validate plist exists
if [ ! -f "$PLIST_SOURCE" ]; then
    echo "[FAIL] ERROR: Plist not found at $PLIST_SOURCE"
    exit 1
fi

# ============================================
# SAFETY CHECK: Verify plist doesn't have dangerous options
# ============================================
echo "  Checking plist for dangerous options..."
if grep -qi "keepalive" "$PLIST_SOURCE"; then
    echo "[WARN] WARNING: plist contains KeepAlive - this can cause infinite loops!"
    echo "   Remove KeepAlive from $PLIST_SOURCE before installing."
    read -p "   Continue anyway? (y/N): " confirm
    if [[ ! "$confirm" =~ ^[Yy]$ ]]; then
        echo "Aborted."
        exit 1
    fi
fi

# Copy and load
cp "$PLIST_SOURCE" "$PLIST_DEST"
echo "[OK] Copied plist to LaunchAgents"

launchctl load "$PLIST_DEST"
echo "[OK] Loaded scheduler"

# Verify
echo ""
if launchctl list 2>/dev/null | grep -q "$PLIST_NAME"; then
    echo "[OK] Scheduler installed successfully!"
    echo ""
    echo "Commands:"
    echo "  Status:     python -m src.scheduler status"
    echo "  Manual run: python -m src.scheduler run"
    echo "  Dry run:    python -m src.scheduler run --dry-run"
    echo "  Trigger:    launchctl start $PLIST_NAME"
    echo "  Logs:       tail -f $LOG_DIR/stdout.log"
    echo "  Uninstall:  ./scripts/uninstall_scheduler.sh"
    echo ""
    echo "Safeguards active:"
    echo "  * Already succeeded today -> skip"
    echo "  * Max 3 retries per day"
    echo "  * Emergency brake at 10 runs/day"
else
    echo "[FAIL] Installation failed - scheduler not in launchctl list"
    exit 1
fi
```

---

## File 5: Uninstall Script

**Path:** `scripts/uninstall_scheduler.sh`

```bash
#!/bin/bash
#
# Uninstall launchd scheduler
#

# ============================================
# CONFIGURATION - UPDATE THIS
# ============================================
PROJECT_ID="[PROJECT_ID]"

# ============================================
# UNINSTALL
# ============================================
PLIST_NAME="com.${PROJECT_ID}.daily"
PLIST_PATH="$HOME/Library/LaunchAgents/$PLIST_NAME.plist"

echo "Uninstalling scheduler..."

if launchctl list 2>/dev/null | grep -q "$PLIST_NAME"; then
    launchctl unload "$PLIST_PATH" 2>/dev/null
    echo "[OK] Unloaded from launchd"
fi

if [ -f "$PLIST_PATH" ]; then
    rm "$PLIST_PATH"
    echo "[OK] Removed plist"
fi

echo "[OK] Scheduler uninstalled"
```

---

## State File Example

**Path:** `data/state.json` (auto-generated)

```json
{
  "last_run": "2026-01-31T09:00:15.123456",
  "last_success": "2026-01-31T09:00:15.123456",
  "run_count": 42,
  "success_count": 40,
  "failure_count": 2,
  "last_success_date": "2026-01-31",
  "retries_today": 0,
  "retry_date": "2026-01-31",
  "runs_today": 1,
  "runs_date": "2026-01-31",
  "last_outcome": "success",
  "last_error": null,
  "total_cost": 51.66,
  "current_rotation_index": 1,
  "rotation_week": 5
}
```

---

## Commands Reference

```bash
# ============================================
# SCHEDULER COMMANDS
# ============================================
python -m src.scheduler run              # Execute task (with safeguards)
python -m src.scheduler run --dry-run    # Test without executing
python -m src.scheduler run --force      # Bypass safeguards (use with caution)
python -m src.scheduler status           # Show state & safeguard status
python -m src.scheduler enable           # Enable scheduler (resume runs)
python -m src.scheduler disable          # Disable scheduler (pause without uninstalling)
python -m src.scheduler reset --confirm  # Reset state
python -m src.scheduler next-rotation    # Show rotation info

# ============================================
# QUICK TOGGLE (alternative to CLI)
# ============================================
touch data/.disabled                     # Disable (pause)
rm data/.disabled                        # Enable (resume)

# ============================================
# LAUNCHD COMMANDS
# ============================================
launchctl list | grep [PROJECT_ID]       # Check if loaded
launchctl start com.[PROJECT_ID].daily   # Manually trigger
launchctl stop com.[PROJECT_ID].daily    # Stop running job
launchctl unload ~/Library/LaunchAgents/com.[PROJECT_ID].daily.plist  # Uninstall

# ============================================
# TROUBLESHOOTING
# ============================================
tail -f ~/Library/Logs/[PROJECT_NAME]/stdout.log   # Live logs
cat data/state.json | python -m json.tool           # View state
rm data/.lock                                       # Clear stuck lock
```

---

## Troubleshooting

| Problem | Check | Fix |
|---------|-------|-----|
| Not running | `launchctl list \| grep PROJECT_ID` | Run `install_scheduler.sh` |
| Runs but fails | `tail ~/Library/Logs/.../stderr.log` | Check error logs |
| Lock stuck | `ls data/.lock` | `rm data/.lock` (auto-clears after 1 hour) |
| Wrong time | Check plist Hour/Minute | Edit plist, unload, reload |
| Env vars missing | Check wrapper.log | Verify `.env` file exists |
| venv not found | Check wrapper.log | Verify `venv/` path |
| **Agent paused** | `ls data/.disabled` | `scheduler enable` or `rm data/.disabled` |
| **Infinite loop** | Check state.json runs_today | See below |
| **Ran too many times** | Emergency brake should activate | Check if KeepAlive in plist |

### Diagnosing Infinite Loop / Runaway Execution

If your scheduler ran many times unexpectedly:

1. **Check the plist for dangerous options:**
   ```bash
   grep -i "keepalive\|runatload\|throttle" config/com.*.plist
   ```
   If found, REMOVE them and reinstall.

2. **Check state.json:**
   ```bash
   cat data/state.json | python -m json.tool
   ```
   Look at `runs_today` -- if > 10, emergency brake should have activated.

3. **Check exit codes in logs:**
   ```bash
   grep "exited with code" data/logs/wrapper.log
   ```
   All skips should exit with code 0.

4. **Reset and reinstall:**
   ```bash
   ./scripts/uninstall_scheduler.sh
   python -m src.scheduler reset --confirm
   # Fix any plist issues
   ./scripts/install_scheduler.sh
   ```

---

## Safeguards Summary

| Safeguard | What It Prevents | Trigger | Action |
|-----------|------------------|---------|--------|
| **Disabled toggle** | Unwanted runs during maintenance | `.disabled` file exists | Skip, exit 0 |
| **Already succeeded today** | Duplicate successful runs | Success recorded for today | Skip, exit 0 |
| **Retry limit** | Infinite retry loops | 3 failures in one day | Skip, exit 0 |
| **Emergency brake** | Runaway execution | 10 runs in one day | Skip, exit 0, notify |
| **Stale lock detection** | Stuck processes | Lock file > 1 hour old | Auto-clear lock |
| **Smart exit codes** | launchd restart loops | Any intentional skip | Exit 0 (not 1) |
| **No KeepAlive** | launchd auto-restart | N/A | Not in plist |

---

## Why These Design Choices?

| Choice | Why |
|--------|-----|
| **launchd** (not cron) | Native macOS, survives reboots, handles sleep/wake |
| **fcntl lock** (not file existence) | Atomic, kernel-level, no race conditions |
| **JSON state** (not SQLite) | Human-readable, easy to debug, no dependencies |
| **Wrapper script** | launchd runs minimal env - wrapper sets up venv/env vars |
| **Weekly rotation** | Survives missed days, predictable schedule |
| **Retry with delay** | Resilience for API failures without hammering |
| **Exit 0 on skip** | Prevents launchd from misinterpreting skips as failures |
| **Emergency brake** | Last resort protection against bugs/misconfiguration |
| **No KeepAlive** | Prevents the most common cause of infinite loops |

---

## Customization Options

### Remove Rotation Logic
If you don't need rotation:
1. Set `ROTATION_ITEMS = []` in scheduler.py
2. Remove rotation fields from `SchedulerState`
3. Remove `get_rotation_item()` call in `run()`

### Remove Cost Tracking
1. Remove `total_cost` from `SchedulerState`
2. Remove cost update in `run()`

### Add Custom State Fields
```python
@dataclass
class SchedulerState:
    # ... existing fields ...
    my_custom_field: str = ""
    my_counter: int = 0
```

### Change Schedule
Edit the plist `StartCalendarInterval` - see Schedule Options Reference above.

### Adjust Safeguard Limits
```python
MAX_RETRIES = 3           # Change retry limit
MAX_RUNS_PER_DAY = 10     # Change emergency brake threshold
RETRY_DELAY_SECONDS = 300  # Change delay between retries
```

---

## Migration to Nested Structure (Future)

This template uses a **flat structure** (`src/scheduler.py`) for simplicity. When you need multiple agents, migrate to a **nested structure**.

### Current: Flat Structure

```
src/
+-- scheduler.py          # Single scheduler for one agent
+-- agent/
|   +-- runner.py         # Task logic
|   +-- state.py
+-- tools/
+-- pipelines/
```

**Command:** `python -m src.scheduler run`

### Future: Nested Structure

```
src/
+-- scheduler.py          # Dispatcher (routes to agent schedulers)
+-- agents/
|   +-- content/
|   |   +-- scheduler.py  # Content agent scheduler
|   |   +-- runner.py
|   +-- analytics/
|       +-- scheduler.py  # Analytics agent scheduler
|       +-- runner.py
+-- shared/
    +-- utils.py          # Shared across agents
```

**Commands:**
- `python -m src.scheduler run --agent content`
- `python -m src.scheduler run --agent analytics`

### Migration Path

```
FLAT (Now)                           NESTED (Later)
============                         ==============

scheduler.py          refactor to    scheduler.py (dispatcher)
(monolithic)          ----------->   |
                                     +-- agents/content/scheduler.py
                                     +-- agents/analytics/scheduler.py

ContentAgent          move to        agents/content/
.execute_task()       ----------->   runner.py
```

### Interface Contract (Never Changes)

Whether flat or nested, the task interface stays the same:

```python
class Agent:
    def execute_task(self, task_name: str, dry_run: bool = False) -> TaskResult:
        """This interface NEVER changes. Flat or nested, the contract is the same."""
        pass

@dataclass
class TaskResult:
    success: bool
    cost: float = 0.0
    data: dict = None
    error: str = None
```

### When to Migrate

| Stay Flat | Go Nested |
|-----------|-----------|
| Single agent | Multiple independent agents |
| Simple schedules | Different schedules per agent |
| Shared state | Isolated state per agent |
| < 500 lines in scheduler.py | Growing complexity |

**Key principle:** Design code to read configs and isolate agent logic. Folder structure is secondary; the architecture is what matters for scalability.

---

**End of Template**
