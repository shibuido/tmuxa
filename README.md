# TMux Attach

> Interactive tmux session selector with fuzzy finder support

Tmuxa enhances tmux session management by providing an interactive session selector with fuzzy finder integration. Skip typing session names and navigate your tmux sessions with elegant, keyboard-driven selection.

## Key Features

- **🔍 Fuzzy Finding** - Uses sk/fzf for lightning-fast session selection
- **🖥️ Multi-Server Support** - Handles multiple tmux servers seamlessly  
- **⌨️ Smart Defaults** - Automatically adds `-t` flag when needed
- **🔧 Flag Forwarding** - All tmux attach flags supported
- **📱 Graceful Fallbacks** - Works with sk, fzf, or shell select menu

## Quick Start

```bash
# Install
chmod +x tmuxa && mv tmuxa ~/bin/

# Basic usage
tmuxa                    # Interactive session selection
tmuxa -d                 # Attach in detached mode  
tmuxa -dt                # Combined flags
```

## Example Workflow

```bash
# Instead of this:
tmux ls
tmux attach -t long-session-name-here

# Use this:
tmuxa
# → Interactive fuzzy selection
# → Instant attach to chosen session
```

## Session Selection

Tmuxa automatically detects the best available selector:

1. **sk** (skim) - Ultra-fast fuzzy finder (preferred)
2. **fzf** - Popular fuzzy finder (fallback)
3. **shell select** - Built-in menu (universal fallback)

## Multi-Server Support

When multiple tmux servers are running, tmuxa shows server:session pairs:

```bash
$ tmuxa
# Shows: 
# default:main-session
# ubertmux:development  
# ubertmux:sysadmin
```

## Documentation

**📖 [Complete Documentation](tmuxa.README.md)** - Detailed usage, configuration, and advanced features

**⚡ [Quick Reference](tmuxa.README.md#usage-examples)** - Common usage patterns

**🔧 [Integration Guide](tmuxa.README.md#integration)** - Shell aliases and workflow tips

## Requirements

- tmux (any recent version)
- Optional: sk or fzf for fuzzy finding
- Bash shell environment

## Part of shibuido

Tmuxa is part of the [shibuido](https://github.com/shibuido) collection - tools following "the way of subtle, unobtrusive beauty in utility."

---

**Ready to streamline your tmux workflow?** → [Read the full documentation](tmuxa.README.md)