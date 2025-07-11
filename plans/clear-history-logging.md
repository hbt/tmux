# Clear-History Logging Feature

## Objective
Intercept tmux `clear-history` commands and log them instead of actually clearing the history, for debugging purposes.

## Requirements
- When `clear-history` command is executed, log the action but DO NOT clear the history
- Create log file at `/tmp/tmux/debug/clear-history.log` (create directories if needed)
- Log should include:
  - Timestamp
  - TMUX_PANE environment variable
  - Process PID and parent PID
  - Pane ID and dimensions
  - History size (number of lines that would be cleared)
  - Note that history was NOT actually cleared

## Implementation Location
- File: `cmd-capture-pane.c`
- Function: `cmd_capture_pane_exec()`
- Condition: `if (cmd_get_entry(self) == &cmd_clear_history_entry)`
- Action: Replace `grid_clear_history(wp->base.grid)` with logging code

## Key Implementation Details
- Add necessary headers: `#include <time.h>`, `#include <unistd.h>`, `#include <sys/stat.h>`
- Use `mkdir()` to create log directory structure
- Use standard C file I/O for logging
- Comment out the actual clearing: `/* grid_clear_history(wp->base.grid); */`

## Testing Protocol
- **CRITICAL**: Use isolated tmux socket for testing: `./tmux -S /tmp/test-socket`
- Never test on live tmux server
- Create test session, add history, run clear-history, verify:
  1. History is NOT cleared
  2. Log file is created with correct information

## Status
- Code modification completed
- Compilation successful
- **TESTING FAILED** - destroyed live tmux sessions due to improper testing approach
- Need to restart with proper isolation protocol