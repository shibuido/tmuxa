# TMux Attach - Interactive Session Selector

Tmuxa is an interactive tmux session selector that enhances the `tmux attach` workflow with fuzzy finder integration and multi-server support. It provides elegant, keyboard-driven session selection to replace the traditional `tmux ls` → `tmux attach -t session-name` workflow.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Usage Examples](#usage-examples)
- [Pre-filling the Query](#pre-filling-the-query)
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
tmuxa -d        # detach other clients from the session
tmuxa -dt       # same, combined form
tmuxa -r        # read-only

# Start the finder with a query already typed in
tmuxa -q work

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
# Take the session over: detach every other client
tmuxa -d

# Same, and send SIGHUP to the other clients' parent shells
tmuxa -x

# Read-only client
tmuxa -r

# Start the session in a given working directory
tmuxa -c ~/project

# Combined flags work naturally
tmuxa -dt    # detach others + target (session appended after -t)
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

## Pre-filling the Query

`-q QUERY` / `--query QUERY` opens the finder with `QUERY` already typed in. It
works like `sk -q` and `fzf -q`: the list starts narrowed, you can still edit
the query, and you still press Enter to choose.

**If exactly one session matches, tmuxa attaches to it straight away**, without
showing the finder (it passes `--select-1` to sk/fzf). So `tmuxa -q medic`
goes directly to the only session matching "medic".

```bash
tmuxa -q work            # finder opens with "work" typed in
tmuxa --query work       # same
tmuxa --query=work       # same
tmuxa -qwork             # same
tmuxa -d -q work         # combine with attach flags, in any order
tmuxa -q work -d         # same
tmuxa -q 'ubertmux:'     # with several servers, narrow to one server
```

Rules:

* `-q` belongs to tmuxa. It is removed before the remaining flags go to
  `tmux attach`, so tmux never sees it. (`tmux attach` has no `-q` of its own.)
* Give it as its own argument. A cluster like `-dq work` is not split apart;
  write `-d -q work`.
* `-q` with no value is an error (exit 2). If `-q` appears twice, the last one
  wins.
* Without sk or fzf, the `select` menu shows only sessions whose name contains
  `QUERY`, ignoring case. This is a plain substring match, not fuzzy. One
  match attaches directly. If none match, tmuxa prints
  `No tmux sessions match query: QUERY` and exits 1.

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
tmuxa -d                 # Interactive selection, detach other clients
tmuxa -dt                # Interactive selection, detached + target
tmuxa -q work            # Interactive selection, query pre-filled
tmuxa --help             # Show help and usage information
```

### tmuxa's Own Options

| Option | Meaning |
|--------|---------|
| `-h`, `--help`, `help` | Show help and exit. Must be the first argument. |
| `-q QUERY`, `--query QUERY`, `-qQUERY`, `--query=QUERY` | Start the finder with `QUERY` typed in; a single match attaches directly. Removed before forwarding. See [Pre-filling the Query](#pre-filling-the-query). |

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

Start with the list already narrowed:

```bash
tmuxa -q dev             # finder starts filtered to "dev"
```

sk and fzf read their usual query syntax, so `-q` accepts it too:

```bash
tmuxa -q "'work"         # names containing exactly "work" (no fuzzy matching)
tmuxa -q '^dev'          # sessions whose name starts with "dev"
tmuxa -q 'dev | work'    # "dev" or "work"
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
- `1` - No sessions found, or none match `-q` (select-menu fallback only)
- `2` - `-q`/`--query` given without a value
- `3` - No session selected (user cancelled)

### Performance Characteristics

- **Session enumeration**: O(n) where n = number of servers
- **Fuzzy finder**: O(log n) typical case for session selection
- **Memory usage**: Minimal, processes sessions as streams

---

For questions, issues, or contributions, visit the [tmuxa repository](https://github.com/shibuido/tmuxa).