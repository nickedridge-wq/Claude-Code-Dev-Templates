# Cleanup & Housekeeping Template

**Platform:** macOS
**Use Case:** Automated file cleanup to prevent disk bloat
**Companion to:** SCHEDULER_TEMPLATE.md (cleanup runs as a scheduled task)

---

## Quick Start Checklist

```
1. [ ] Determine if cleanup is needed (see criteria below)
2. [ ] Define retention policies in config/default.yaml
3. [ ] Create file_cleaner.py tool (dumb executor)
4. [ ] Create cleanup_runner.py (orchestration)
5. [ ] Extend scheduler.py with `cleanup` command
6. [ ] Create cleanup plist (weekly schedule)
7. [ ] Update install_scheduler.sh for both jobs
8. [ ] Test with --dry-run first
9. [ ] Verify protected files are excluded
```

---

## When to Use This Template

| If the agent generates... | Cleanup Recommended |
|---------------------------|---------------------|
| Video/audio/image files | Yes -- these accumulate fast |
| Large data files or caches | Yes |
| Log files | Yes -- rotate after 7-30 days |
| Only small text outputs | Maybe -- low priority |
| Nothing persistent | No -- don't use this template |

**Rule of thumb:** If your agent could generate >1GB/year of files, implement cleanup.

---

## What to Clean vs. Protect

| Action | File Types |
|--------|------------|
| **DELETE after N days** | Generated media (videos, audio, images), intermediate files, temp/cache, old logs |
| **NEVER auto-delete** | `state.json`, `production.jsonl`, configuration files, session documentation |

---

## Architecture

```
+---------------------------------------------------------------------+
|                         CLEANUP SYSTEM                              |
+---------------------------------------------------------------------+
|                                                                     |
|  +------------------+    +------------------+    +----------------+ |
|  |     launchd      |--->|  run_cleanup.sh  |--->|  scheduler.py  | |
|  | (weekly trigger) |    |    (wrapper)     |    | cleanup cmd    | |
|  +------------------+    +------------------+    +----------------+ |
|                                 |                       |           |
|                                 |                       v           |
|                         Safeguards:          +------------------+   |
|                         * Stale lock check   | cleanup_runner.py|   |
|                         * Lock detection     | (orchestration)  |   |
|                         * Exit codes         +------------------+   |
|                                                         |           |
|                                                         v           |
|                                              +------------------+   |
|                                              |  file_cleaner.py |   |
|                                              |  (dumb executor) |   |
|                                              +------------------+   |
|                                                                     |
|  SAFEGUARDS:                                                        |
|  [x] Protected paths never deleted                                  |
|  [x] Dry-run mode for testing                                       |
|  [x] Separate lock file (.cleanup.lock)                             |
|  [x] Stale lock detection (auto-clears locks older than 1 hour)     |
|  [x] Age-based deletion only (no pattern-only deletes)              |
|  [x] Smart exit codes (exit 0 on skip, launchd won't restart)       |
+---------------------------------------------------------------------+
```

---

## File Structure

```
[project-name]/
|
+-- config/
|   +-- default.yaml                     # Includes retention_policies
|   +-- com.[projectid].cleanup.plist    # Weekly cleanup schedule
|
+-- src/
|   +-- scheduler.py                     # Extended with `cleanup` command
|   |
|   +-- agent/
|   |   +-- cleanup_runner.py            # Cleanup orchestration
|   |
|   +-- tools/
|       +-- file_cleaner.py              # Dumb file deletion tool
|
+-- scripts/
|   +-- run_cleanup.sh                   # Wrapper for launchd
|
+-- data/
    +-- .cleanup.lock                    # Separate lock for cleanup
```

---

## File 1: Retention Policy Configuration

**Path:** `config/default.yaml` (add this section)

```yaml
# Retention policies for automated cleanup
# Days to keep files (0 = never delete)
retention_policies:
  videos: 7
  audio: 7
  images: 7
  temp: 1
  cache: 7
  logs: 30
  scripts: 30      # Small, useful for analysis
  
  # Protected paths (NEVER auto-delete)
  protected:
    - "data/state.json"
    - "data/logs/production.jsonl"
    - "config/"
    - "docs/"

# Directory mappings for each policy
cleanup_paths:
  videos: "data/output/videos"
  audio: "data/output/audio"
  images: "data/output/images"
  temp: "data/temp"
  cache: "data/cache"
  logs: "data/logs"
  scripts: "data/output/scripts"

# File patterns for each policy
cleanup_patterns:
  videos: "*.mp4"
  audio: "*.mp3"
  images: "*.png"
  temp: "*"
  cache: "*"
  logs: "*.log"
  scripts: "*.sh"
```

---

## File 2: File Cleaner Tool

**Path:** `src/tools/file_cleaner.py`

```python
#!/usr/bin/env python3
"""
File Cleaner Tool

A dumb executor that deletes files matching a pattern older than a specified age.
No decisions - just executes what it's told.

SAFEGUARDS:
- Requires both pattern AND max_age_days (no delete-all-matching)
- Dry-run mode for testing
- Returns structured output for logging
"""

import os
from dataclasses import dataclass, field
from datetime import datetime, timedelta
from pathlib import Path
from typing import List


@dataclass
class CleanupInput:
    """Input for file cleanup operation."""
    directory: Path
    pattern: str              # e.g., "*.mp4", "*.log"
    max_age_days: int         # Files older than this are deleted
    dry_run: bool = False     # If True, simulate without deleting


@dataclass
class CleanupOutput:
    """Output from file cleanup operation."""
    success: bool
    files_found: int = 0
    files_deleted: int = 0
    bytes_freed: int = 0
    errors: List[str] = field(default_factory=list)
    deleted_files: List[str] = field(default_factory=list)  # For logging


class FileCleaner:
    """
    Dumb tool: deletes files older than max_age_days matching pattern.
    
    Rules:
    - No decisions (just executes)
    - No side effects beyond stated purpose
    - Always returns structured output
    - Handles own errors gracefully
    """
    
    def execute(self, input: CleanupInput) -> CleanupOutput:
        """
        Find and delete files matching pattern older than max_age_days.
        
        Args:
            input: CleanupInput with directory, pattern, max_age_days, dry_run
            
        Returns:
            CleanupOutput with results and any errors
        """
        output = CleanupOutput(success=True)
        
        # Validate inputs
        if not input.directory.exists():
            output.errors.append(f"Directory does not exist: {input.directory}")
            output.success = False
            return output
        
        if input.max_age_days < 1:
            output.errors.append(f"max_age_days must be >= 1, got {input.max_age_days}")
            output.success = False
            return output
        
        # Calculate cutoff time
        cutoff = datetime.now() - timedelta(days=input.max_age_days)
        
        # Find matching files
        try:
            matching_files = list(input.directory.glob(input.pattern))
        except Exception as e:
            output.errors.append(f"Error scanning directory: {e}")
            output.success = False
            return output
        
        output.files_found = len(matching_files)
        
        # Process each file
        for file_path in matching_files:
            if not file_path.is_file():
                continue
                
            try:
                # Check file age
                mtime = datetime.fromtimestamp(file_path.stat().st_mtime)
                if mtime >= cutoff:
                    continue  # File is not old enough
                
                file_size = file_path.stat().st_size
                
                if input.dry_run:
                    # Simulate deletion
                    output.files_deleted += 1
                    output.bytes_freed += file_size
                    output.deleted_files.append(str(file_path))
                else:
                    # Actually delete
                    file_path.unlink()
                    output.files_deleted += 1
                    output.bytes_freed += file_size
                    output.deleted_files.append(str(file_path))
                    
            except PermissionError:
                output.errors.append(f"Permission denied: {file_path}")
            except Exception as e:
                output.errors.append(f"Error processing {file_path}: {e}")
        
        # Mark as failed if there were errors but some files were processed
        if output.errors and output.files_deleted == 0:
            output.success = False
        
        return output
    
    def format_bytes(self, bytes_count: int) -> str:
        """Format bytes as human-readable string."""
        for unit in ['B', 'KB', 'MB', 'GB']:
            if bytes_count < 1024:
                return f"{bytes_count:.1f} {unit}"
            bytes_count /= 1024
        return f"{bytes_count:.1f} TB"


# Standalone usage for testing
if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 4:
        print("Usage: python file_cleaner.py <directory> <pattern> <max_age_days> [--dry-run]")
        sys.exit(1)
    
    cleaner = FileCleaner()
    result = cleaner.execute(CleanupInput(
        directory=Path(sys.argv[1]),
        pattern=sys.argv[2],
        max_age_days=int(sys.argv[3]),
        dry_run="--dry-run" in sys.argv
    ))
    
    print(f"Success: {result.success}")
    print(f"Files found: {result.files_found}")
    print(f"Files deleted: {result.files_deleted}")
    print(f"Space freed: {cleaner.format_bytes(result.bytes_freed)}")
    if result.errors:
        print(f"Errors: {result.errors}")
```

---

## File 3: Cleanup Runner

**Path:** `src/agent/cleanup_runner.py`

```python
#!/usr/bin/env python3
"""
Cleanup Runner

Orchestrates file cleanup across all configured paths.
Reads retention policies from config and delegates to FileCleaner tool.
"""

import yaml
from dataclasses import dataclass, field
from pathlib import Path
from typing import Dict, List, Any

from src.tools.file_cleaner import FileCleaner, CleanupInput, CleanupOutput


@dataclass
class CleanupResult:
    """Aggregated result from all cleanup operations."""
    success: bool
    total_files_deleted: int = 0
    total_bytes_freed: int = 0
    policy_results: Dict[str, CleanupOutput] = field(default_factory=dict)
    errors: List[str] = field(default_factory=list)


class CleanupRunner:
    """
    Orchestrates cleanup across all configured paths.
    
    Responsibilities:
    - Load retention policies from config
    - Coordinate FileCleaner tool for each policy
    - Aggregate results
    - Respect protected paths
    """
    
    def __init__(self, config_path: Path = None):
        self.config_path = config_path or Path("config/default.yaml")
        self.config = self._load_config()
        self.file_cleaner = FileCleaner()
        self.project_root = Path(__file__).parent.parent.parent
    
    def _load_config(self) -> Dict[str, Any]:
        """Load configuration from YAML file."""
        if not self.config_path.exists():
            raise FileNotFoundError(f"Config not found: {self.config_path}")
        
        with open(self.config_path) as f:
            return yaml.safe_load(f)
    
    def _is_protected(self, path: Path) -> bool:
        """Check if a path is in the protected list."""
        protected = self.config.get("retention_policies", {}).get("protected", [])
        path_str = str(path)
        
        for protected_path in protected:
            # Check if the path starts with any protected path
            full_protected = str(self.project_root / protected_path)
            if path_str.startswith(full_protected):
                return True
        
        return False
    
    def run(self, dry_run: bool = False) -> CleanupResult:
        """
        Execute cleanup for all configured policies.
        
        Args:
            dry_run: If True, simulate without deleting files
            
        Returns:
            CleanupResult with aggregated statistics
        """
        result = CleanupResult(success=True)
        
        retention = self.config.get("retention_policies", {})
        paths = self.config.get("cleanup_paths", {})
        patterns = self.config.get("cleanup_patterns", {})
        
        for policy_name, max_days in retention.items():
            # Skip non-integer values (like 'protected' list)
            if not isinstance(max_days, int):
                continue
            
            # Skip if max_days is 0 (never delete)
            if max_days == 0:
                continue
            
            # Get path and pattern for this policy
            rel_path = paths.get(policy_name)
            pattern = patterns.get(policy_name)
            
            if not rel_path or not pattern:
                result.errors.append(f"Missing path or pattern for policy: {policy_name}")
                continue
            
            full_path = self.project_root / rel_path
            
            # Check if path is protected
            if self._is_protected(full_path):
                result.errors.append(f"Skipping protected path: {full_path}")
                continue
            
            # Skip if directory doesn't exist
            if not full_path.exists():
                continue
            
            # Execute cleanup
            cleanup_output = self.file_cleaner.execute(CleanupInput(
                directory=full_path,
                pattern=pattern,
                max_age_days=max_days,
                dry_run=dry_run
            ))
            
            # Store result
            result.policy_results[policy_name] = cleanup_output
            result.total_files_deleted += cleanup_output.files_deleted
            result.total_bytes_freed += cleanup_output.bytes_freed
            
            if cleanup_output.errors:
                result.errors.extend(cleanup_output.errors)
            
            if not cleanup_output.success:
                result.success = False
        
        return result
    
    def format_bytes(self, bytes_count: int) -> str:
        """Format bytes as human-readable string."""
        return self.file_cleaner.format_bytes(bytes_count)
    
    def print_summary(self, result: CleanupResult, dry_run: bool = False) -> None:
        """Print human-readable summary of cleanup results."""
        prefix = "[DRY RUN] " if dry_run else ""
        
        print(f"\n{prefix}Cleanup Summary")
        print("=" * 50)
        print(f"Total files deleted: {result.total_files_deleted}")
        print(f"Total space freed: {self.format_bytes(result.total_bytes_freed)}")
        print()
        
        for policy_name, output in result.policy_results.items():
            if output.files_deleted > 0:
                print(f"  {policy_name}: {output.files_deleted} files ({self.format_bytes(output.bytes_freed)})")
        
        if result.errors:
            print(f"\nErrors ({len(result.errors)}):")
            for error in result.errors[:5]:  # Show first 5 errors
                print(f"  - {error}")
            if len(result.errors) > 5:
                print(f"  ... and {len(result.errors) - 5} more")
        
        print("=" * 50)
        status = "SUCCESS" if result.success else "COMPLETED WITH ERRORS"
        print(f"Status: {status}\n")


# Standalone usage
if __name__ == "__main__":
    import sys
    
    dry_run = "--dry-run" in sys.argv
    
    runner = CleanupRunner()
    result = runner.run(dry_run=dry_run)
    runner.print_summary(result, dry_run=dry_run)
    
    sys.exit(0 if result.success else 1)
```

---

## File 4: Scheduler Integration

**Add to:** `src/scheduler.py`

Add the cleanup command to your existing scheduler. Add this to the CLI commands:

```python
# Add to imports
from src.agent.cleanup_runner import CleanupRunner

# Add to subparsers (in main())
p_cleanup = subparsers.add_parser("cleanup", help="Run file cleanup")
p_cleanup.add_argument("--dry-run", action="store_true", help="Simulate without deleting")

# Add to commands dict
def cmd_cleanup(dry_run: bool = False) -> int:
    """Run cleanup with separate lock."""
    cleanup_lock = DATA_DIR / ".cleanup.lock"
    lock_fd = None
    
    try:
        # Acquire cleanup-specific lock
        DATA_DIR.mkdir(parents=True, exist_ok=True)
        lock_fd = open(cleanup_lock, 'w')
        fcntl.flock(lock_fd.fileno(), fcntl.LOCK_EX | fcntl.LOCK_NB)
        
        runner = CleanupRunner()
        result = runner.run(dry_run=dry_run)
        runner.print_summary(result, dry_run=dry_run)
        
        return 0 if result.success else 1
        
    except BlockingIOError:
        print("Another cleanup is already running")
        return 0
    finally:
        if lock_fd:
            fcntl.flock(lock_fd.fileno(), fcntl.LOCK_UN)
            lock_fd.close()
            cleanup_lock.unlink(missing_ok=True)

# Add to commands mapping
commands = {
    # ... existing commands ...
    "cleanup": lambda: cmd_cleanup(dry_run=args.dry_run),
}
```

---

## File 5: Cleanup Wrapper Script

**Path:** `scripts/run_cleanup.sh`

```bash
#!/bin/bash
#
# Wrapper script for cleanup launchd job
# Sets up environment and runs Python cleanup with safeguards
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
LOG_FILE="$PROJECT_DIR/data/logs/cleanup.log"
LOCK_FILE="$PROJECT_DIR/data/.cleanup.lock"
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
log "Cleanup wrapper started"

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
    else
        # Lock exists and is recent - another process is running
        LOCK_AGE=$(( ($(date +%s) - $(stat -f %m "$LOCK_FILE" 2>/dev/null || echo $(date +%s))) / 60 ))
        log "Another cleanup process is running (lock age: ${LOCK_AGE}m)"
        log "========================================"
        exit 0  # Exit 0 so launchd doesn't restart
    fi
fi

# ============================================
# ACTIVATE VIRTUAL ENVIRONMENT
# ============================================
if [ -f "$VENV_PATH/bin/activate" ]; then
    source "$VENV_PATH/bin/activate"
    log "Virtual environment activated"
elif [ -f "$PROJECT_DIR/.venv/bin/activate" ]; then
    source "$PROJECT_DIR/.venv/bin/activate"
    log "Virtual environment activated (.venv)"
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
# RUN CLEANUP
# ============================================
log "Starting cleanup..."
python -m src.scheduler cleanup 2>&1 | tee -a "$LOG_FILE"

EXIT_CODE=${PIPESTATUS[0]}
log "Cleanup exited with code: $EXIT_CODE"
log "========================================"

# IMPORTANT: Exit codes
# 0 = Success OR intentional skip (launchd will NOT restart)
# 1 = Error (launchd will NOT restart because we don't use KeepAlive)
exit $EXIT_CODE
```

---

## File 6: Cleanup launchd Plist

**Path:** `config/com.[PROJECT_ID].cleanup.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<!--
  Cleanup Scheduler

  CRITICAL: DO NOT ADD KeepAlive TO THIS FILE
  KeepAlive causes restart loops on any non-zero exit code.
  See SCHEDULER_TEMPLATE.md for the rationale.

  Runs weekly at 3:00 AM to clean up old files per retention_policies.
-->
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.[PROJECT_ID].cleanup</string>
    
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/[USERNAME]/[PROJECT_NAME]/scripts/run_cleanup.sh</string>
    </array>
    
    <!-- Weekly on Sunday at 3 AM -->
    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key>
        <integer>0</integer>
        <key>Hour</key>
        <integer>3</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    
    <!-- Timeout: kill if running longer than 30 minutes -->
    <key>TimeOut</key>
    <integer>1800</integer>
    
    <key>StandardOutPath</key>
    <string>/Users/[USERNAME]/Library/Logs/[PROJECT_NAME]/cleanup-stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/[USERNAME]/Library/Logs/[PROJECT_NAME]/cleanup-stderr.log</string>
    
    <key>WorkingDirectory</key>
    <string>/Users/[USERNAME]/[PROJECT_NAME]</string>
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
```

**Safe plist pattern:** Use ONLY `StartCalendarInterval` for scheduling. Let the Python scheduler handle retries internally.

---

## File 7: Updated Install Script

**Update:** `scripts/install_scheduler.sh`

Add cleanup plist installation:

```bash
# Add after daily plist installation

# ============================================
# CLEANUP SCHEDULER (optional)
# ============================================
CLEANUP_PLIST_NAME="com.${PROJECT_ID}.cleanup"
CLEANUP_PLIST_SOURCE="$PROJECT_DIR/config/$CLEANUP_PLIST_NAME.plist"
CLEANUP_PLIST_DEST="$HOME/Library/LaunchAgents/$CLEANUP_PLIST_NAME.plist"

if [ -f "$CLEANUP_PLIST_SOURCE" ]; then
    echo "Installing cleanup scheduler..."
    
    # Unload if exists
    if launchctl list 2>/dev/null | grep -q "$CLEANUP_PLIST_NAME"; then
        launchctl unload "$CLEANUP_PLIST_DEST" 2>/dev/null || true
    fi
    
    # Safety check: Verify plist doesn't have dangerous options
    if grep -qi "keepalive" "$CLEANUP_PLIST_SOURCE"; then
        echo "[WARN] WARNING: plist contains KeepAlive - this can cause infinite loops!"
        echo "   Remove KeepAlive from $CLEANUP_PLIST_SOURCE before installing."
        read -p "   Continue anyway? (y/N): " confirm
        if [[ ! "$confirm" =~ ^[Yy]$ ]]; then
            echo "Aborted."
            exit 1
        fi
    fi
    
    cp "$CLEANUP_PLIST_SOURCE" "$CLEANUP_PLIST_DEST"
    launchctl load "$CLEANUP_PLIST_DEST"
    echo "[OK] Cleanup scheduler installed (weekly)"
fi
```

---

## Storage Impact Estimation

Calculate expected storage with and without cleanup:

| Asset Type | Size Each | Daily Volume | Retention | Max Storage |
|------------|-----------|--------------|-----------|-------------|
| [Type 1] | ~XX MB | X/day | X days | ~XXX MB |
| [Type 2] | ~XX MB | X/day | X days | ~XXX MB |
| [Type 3] | ~XX KB | X/day | X days | ~XXX KB |
| **Total** | -- | -- | -- | **~X GB** |

**Without cleanup:** ~XX GB/year
**With cleanup:** Never exceeds ~X GB

---

## Commands Reference

```bash
# ============================================
# CLEANUP COMMANDS
# ============================================
python -m src.scheduler cleanup              # Run cleanup now
python -m src.scheduler cleanup --dry-run    # Preview what would be deleted

# ============================================
# LAUNCHD COMMANDS
# ============================================
launchctl list | grep cleanup                # Check if loaded
launchctl start com.[PROJECT_ID].cleanup     # Manually trigger
launchctl unload ~/Library/LaunchAgents/com.[PROJECT_ID].cleanup.plist  # Disable

# ============================================
# LOGS
# ============================================
tail -f data/logs/cleanup.log                # Cleanup log
tail -f ~/Library/Logs/[PROJECT_NAME]/cleanup-stdout.log  # launchd output
```

---

## Troubleshooting

| Problem | Check | Fix |
|---------|-------|-----|
| Files not being deleted | Check retention_policies in config | Verify max_days > 0 |
| Protected files deleted | Check protected list in config | Add paths to protected |
| Cleanup not running | `launchctl list \| grep cleanup` | Run install script |
| Lock stuck | `ls data/.cleanup.lock` | `rm data/.cleanup.lock` (auto-clears after 1 hour) |
| Wrong files matched | Check cleanup_patterns in config | Adjust glob patterns |

### Diagnosing Runaway Execution

If cleanup ran many times unexpectedly:

1. **Check the plist for dangerous options:**
   ```bash
   grep -i "keepalive\|runatload\|throttle" config/com.*.cleanup.plist
   ```
   If found, REMOVE them and reinstall.

2. **Check exit codes in logs:**
   ```bash
   grep "exited with code" data/logs/cleanup.log
   ```
   All skips should exit with code 0.

3. **Reset and reinstall:**
   ```bash
   launchctl unload ~/Library/LaunchAgents/com.[PROJECT_ID].cleanup.plist
   # Fix any plist issues
   ./scripts/install_scheduler.sh
   ```

---

## Safeguards Summary

| Safeguard | What It Prevents | Trigger | Action |
|-----------|------------------|---------|--------|
| **Stale lock detection** | Stuck processes | Lock file > 1 hour old | Auto-clear lock |
| **fcntl lock** | Concurrent execution | Another process running | Skip, exit 0 |
| **Smart exit codes** | launchd restart loops | Any intentional skip | Exit 0 (not 1) |
| **No KeepAlive** | launchd auto-restart | N/A | Not in plist |
| **TimeOut** | Hung processes | Running > 30 minutes | launchd kills process |
| **Protected paths** | Accidental data loss | Path in protected list | Skip deletion |
| **Dry-run mode** | Testing mistakes | --dry-run flag | Simulate without deleting |

---

## Session Documentation Archiving

**[!] Do NOT auto-archive session docs.** They're small (~5KB each) and valuable for context.

If you want to archive old session docs, do it **manually** at version boundaries:

```bash
# Manual archive command (implement if needed)
python -m src.scheduler archive-docs --version v1

# Note: Version folders naturally serve as archives - no manual archiving needed
```

Only consider archiving when:
- You have 20+ session folders AND
- You're releasing a major version (v1 -> v2)

---

## Checklist

- [ ] Identified what files accumulate
- [ ] Defined retention policies in config
- [ ] Created `file_cleaner.py` tool (dumb executor)
- [ ] Created `cleanup_runner.py` (orchestration)
- [ ] Extended `scheduler.py` with `cleanup` command
- [ ] Created cleanup plist (weekly schedule)
- [ ] Updated `install_scheduler.sh` for both jobs
- [ ] Tested with `--dry-run` first
- [ ] Protected critical files (state, production log, docs)

---

**End of Template**
