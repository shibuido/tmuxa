# TMux Attach - Interactive Session Selector

Tmuxa is an interactive tmux session selector that enhances the `tmux attach` workflow with fuzzy finder integration and multi-server support. It provides elegant, keyboard-driven session selection to replace the traditional `tmux ls` → `tmux attach -t session-name` workflow.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Usage Examples](#usage-examples)
- [Fuzzy Finder Integration](#fuzzy-finder-integration)
- [Multi-Server Support](#multi-server-support)
- [Command Reference](#command-reference)
- [Integration](#integration)
- [Advanced Usage](#advanced-usage)
- [Troubleshooting](#troubleshooting)
- [Technical Details](#technical-details)

## Installation

1. Clone this repository or download the `tmuxa` script
2. Make it executable and place it in your PATH:

```bash
chmod +x tmuxa
mv tmuxa ~/bin/  # or any directory in your PATH
```

3. Install optional fuzzy finders for enhanced experience:

```bash
# Ubuntu/Debian
sudo apt install fzf

# macOS
brew install sk    # skim (fastest)
brew install fzf   # alternative

# Fedora/RHEL  
sudo dnf install fzf

# Arch Linux
sudo pacman -S sk fzf
```

4. Ensure tmux is installed:

```bash
# Ubuntu/Debian
sudo apt install tmux

# macOS
brew install tmux

# Fedora/RHEL
sudo dnf install tmux
```

## Quick Start

```bash
# Start with interactive session selection
tmuxa

# Attach with specific flags
tmuxa -d        # detached mode
tmuxa -dt       # detached with target
tmuxa -x 120    # set terminal width

# Get help
tmuxa --help
```

## Usage Examples

### Basic Session Selection

Replace this workflow:
```bash
$ tmux ls
session-1: 3 windows (created Sat Jun 19 10:15:32 2025)
long-project-name: 5 windows (created Sat Jun 19 09:30:15 2025)
temp-session: 1 windows (created Sat Jun 19 11:45:22 2025)

$ tmux attach -t long-project-name
```

With this:
```bash
$ tmuxa
# Interactive fuzzy selector appears:
# > session-1
#   long-project-name
#   temp-session
# Type "long" → "long-project-name" selected → Enter → attached
```

### Flag Forwarding

All `tmux attach` flags are supported:

```bash
# Detached mode
tmuxa -d

# Set specific terminal dimensions
tmuxa -x 120 -y 40

# Force attach (steal from other clients)
tmuxa -d

# Combined flags work naturally
tmuxa -dt    # detached + target (auto-added)
```

### Multi-Server Scenarios

When multiple tmux servers are running:

```bash
$ tmuxa
# Shows server:session format:
# > default:main
#   default:work
#   ubertmux:sysadmin
#   ubertmux:development
#   custom-server:project-x
```

## Fuzzy Finder Integration

Tmuxa automatically detects and uses the best available fuzzy finder:

### 1. sk (skim) - Preferred

Fast rust-based fuzzy finder with excellent performance:

```bash
# Installation
cargo install sk
# or
brew install sk
```

### 2. fzf - Popular Alternative

Well-established fuzzy finder with extensive features:

```bash
# Installation varies by system
sudo apt install fzf      # Ubuntu/Debian
brew install fzf          # macOS
sudo dnf install fzf      # Fedora/RHEL
```

### 3. Shell Select - Universal Fallback

Built-in bash select menu when no fuzzy finder is available:

```bash
$ tmuxa
Select a session:
1) session-1
2) long-project-name  
3) temp-session
#? 2
Attaching to session: long-project-name
```

## Multi-Server Support

Tmuxa automatically detects multiple tmux servers by scanning socket files in `/tmp/tmux-$UID/`:

### Automatic Detection

```bash
# Servers detected: default, ubertmux, custom-server
$ tmuxa
# Displays: server:session format for clear identification
ubertmux:development
ubertmux:sysadmin
default:main-work
custom-server:special-project
```

### Server Socket Handling

Tmuxa properly handles tmux server sockets:

- **Default server**: Uses standard `tmux attach`
- **Named servers**: Uses `tmux -S socket_path attach`
- **Socket detection**: Scans `/tmp/tmux-$UID/` for socket files

## Command Reference

### Basic Commands

```bash
tmuxa                    # Interactive selection with default flags
tmuxa -d                 # Interactive selection, detached mode
tmuxa -dt                # Interactive selection, detached + target
tmuxa --help             # Show help and usage information
```

### Flag Forwarding Rules

1. **Auto-target addition**: If no `-t`/`--target` flag present, `-t` is added automatically
2. **Flag preservation**: All provided flags are forwarded to `tmux attach`
3. **Inline targets**: `--target=session` format is detected and handled
4. **Combined flags**: Flags like `-dt` are properly parsed

### Target Flag Detection

Tmuxa intelligently detects target flags:

```bash
tmuxa -t          # Target flag detected
tmuxa --target    # Target flag detected  
tmuxa --target=x  # Inline target detected
tmuxa -dt         # Combined flag with target detected
tmuxa -d          # No target → -t added automatically
```

## Integration

### Shell Aliases

Common aliases for enhanced workflow:

```bash
# In ~/.bashrc or ~/.zshrc
alias ta='tmuxa'                    # Quick attach
alias tad='tmuxa -d'               # Quick detached attach
alias tas='tmuxa && echo "Session ended"'  # With feedback
```

### Script Integration

Use tmuxa in scripts:

```bash
#!/bin/bash
# auto-attach.sh - Smart session restoration

if tmux ls >/dev/null 2>&1; then
    # Sessions exist, let user choose
    tmuxa
else
    # No sessions, create default
    tmux new-session -s main
fi
```

### Workflow Automation

Combine with other tools:

```bash
# Morning startup script
#!/bin/bash
# Start common sessions if not running
tmux has-session -t work 2>/dev/null || tmux new-session -d -s work
tmux has-session -t monitoring 2>/dev/null || tmux new-session -d -s monitoring

# Interactive selection
tmuxa
```

## Advanced Usage

### Custom Session Filtering

Use with external tools for advanced filtering:

```bash
# Show only sessions matching pattern (external tool example)
tmux ls | grep "dev\|work" | cut -d: -f1 | fzf | xargs tmux attach -t
```

### Integration with Session Managers

Works seamlessly with tmux session managers:

```bash
# With tmuxinator
tmuxinator start project-x
tmuxa  # Will show the new session

# With tmux-resurrect
tmux resurrect  # Restore sessions
tmuxa           # Select from restored sessions
```

### Scripted Session Selection

Pre-select sessions programmatically:

```bash
# Example: Auto-attach to work session if available
if tmux has-session -t work 2>/dev/null; then
    tmux attach -t work
else
    tmuxa  # Fallback to interactive selection
fi
```

## Troubleshooting

### Common Issues

**Q: "No tmux sessions found"**

A: No active tmux sessions exist. Create one first:
```bash
tmux new-session -s main
# Then use tmuxa
```

**Q: Fuzzy finder not working**

A: Install sk or fzf:
```bash
# Test detection
command -v sk || command -v fzf || echo "No fuzzy finder found"

# Install one
brew install sk    # macOS
sudo apt install fzf  # Ubuntu
```

**Q: Multi-server sessions not showing**

A: Check tmux socket directory:
```bash
ls -la /tmp/tmux-$UID/
# Should show socket files for each server
```

**Q: Flags not being passed correctly**

A: Verify flag syntax with tmux directly:
```bash
# Test the command tmuxa would run
tmux attach --help
```

### Debug Mode

For troubleshooting, add debug output:

```bash
# Temporarily modify the script to add debug info
set -x  # Add this line after #!/usr/bin/env bash
```

### Socket Permissions

If socket access fails:

```bash
# Check socket permissions
ls -la /tmp/tmux-$UID/

# Fix ownership if needed
sudo chown $USER:$USER /tmp/tmux-$UID/*
```

## Technical Details

### Session Detection Algorithm

1. **Socket Discovery**: Scan `/tmp/tmux-$UID/` for socket files
2. **Server Enumeration**: For each socket, run `tmux -S socket ls`
3. **Session Extraction**: Parse session names from tmux output
4. **Presentation**: Format as `server:session` for multi-server scenarios

### Fuzzy Finder Priority

```bash
# Detection order
1. sk (skim)     - Fastest rust implementation
2. fzf           - Popular, feature-rich
3. shell select  - Universal bash fallback
```

### Flag Processing Logic

```bash
# Target flag detection regex patterns
-t|--target                 # Standalone flags
--target=*                  # Inline assignment  
-*t*                       # Combined flags (e.g., -dt)
```

### Exit Codes

- `0` - Success (attached to session)
- `1` - No sessions found
- `3` - No session selected (user cancelled)

### Performance Characteristics

- **Session enumeration**: O(n) where n = number of servers
- **Fuzzy finder**: O(log n) typical case for session selection
- **Memory usage**: Minimal, processes sessions as streams

---

For questions, issues, or contributions, visit the [tmuxa repository](https://github.com/shibuido/tmuxa).