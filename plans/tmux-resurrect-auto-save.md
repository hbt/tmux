# Tmux-Resurrect Auto-Save Protection

## Problem
tmux-resurrect plugin only saves manually (prefix + Ctrl-s), leading to potential data loss when tmux server crashes or gets killed accidentally.

## Solution
Implement automatic periodic saving of tmux sessions to prevent work loss.

## Current State
- tmux-resurrect plugin installed and configured in `.tmux.conf`
- Last manual save: October 31, 2024 (8+ months old)
- Recent sessions lost due to accidental `tmux kill-server`

## Implementation Options

### 1.0 Tmux Hook-Based Auto-Save
```bash
# Add to .tmux.conf
set-hook -g after-new-window 'run-shell "~/.tmux/plugins/tmux-resurrect/scripts/save.sh"'
set-hook -g after-kill-window 'run-shell "~/.tmux/plugins/tmux-resurrect/scripts/save.sh"'
set-hook -g session-created 'run-shell "~/.tmux/plugins/tmux-resurrect/scripts/save.sh"'
```

### 2.0 Cron-Based Auto-Save
```bash
# Add to crontab
*/15 * * * * /home/hassen/.tmux/plugins/tmux-resurrect/scripts/save.sh >/dev/null 2>&1
```

### 3.0 Background Process Auto-Save
Create a background script that runs while tmux is active:
```bash
#!/bin/bash
while tmux list-sessions >/dev/null 2>&1; do
    ~/.tmux/plugins/tmux-resurrect/scripts/save.sh
    sleep 900  # 15 minutes
done
```

### 4.0 Tmux Continuous Save Plugin
Research/install `tmux-continuum` plugin which provides automatic saves.

## Recommended Approach
- **Primary**: Install tmux-continuum plugin for seamless auto-save
- **Backup**: Add tmux hooks for session events
- **Safety net**: Cron job every 15 minutes

## File Locations
- Resurrect saves: `~/.tmux/resurrect/`
- Save script: `~/.tmux/plugins/tmux-resurrect/scripts/save.sh`
- Restore script: `~/.tmux/plugins/tmux-resurrect/scripts/restore.sh`

## Recovery Commands
```bash
# Manual save
tmux run-shell '~/.tmux/plugins/tmux-resurrect/scripts/save.sh'

# Manual restore (kills current sessions)
tmux run-shell '~/.tmux/plugins/tmux-resurrect/scripts/restore.sh'
```

## Priority
**HIGH** - Implement immediately after session restoration to prevent future data loss.

## Status
- Planning phase
- Need to implement after current work is restored