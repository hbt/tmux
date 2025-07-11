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

## Implementation Locations

### 1. Direct Command (`clear-history`)
- File: `cmd-capture-pane.c`
- Function: `cmd_capture_pane_exec()`
- Lines: 205-223
- Condition: `if (cmd_get_entry(self) == &cmd_clear_history_entry)`
- Action: Replace `grid_clear_history(wp->base.grid)` with logging code

### 2. Escape Sequence (`ESC[3J`)
- File: `input.c`
- Function: `input_csi_dispatch()`
- Lines: 1500-1523
- Condition: `case 3:` (CSI sequence handling)
- Action: Replace `screen_write_clearhistory(sctx)` with logging code

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
- Code modification completed in TWO locations:
  1. **cmd-capture-pane.c**: Direct `clear-history` command logging
  2. **input.c**: ESC[3J escape sequence logging  
- Compilation successful
- **TESTING SUCCESSFUL** - Both clear-history methods now logged and disabled:
  - Direct command: `./tmux clear-history` → logs timestamp, preserves scroll buffer
  - Escape sequence: `ESC[3J` from applications → logs timestamp, preserves scroll buffer
- Scroll buffer preservation verified with test commands