# Chapter 2: Installation & Setup

---

## 1. Prerequisites — What You Need First

Before installing Claude Code, you need two things:

### Node.js
Claude Code is built on Node.js. You need **Node.js version 18 or higher**.

Check if you have it:
```bash
node --version   # Should show v18.0.0 or higher
npm --version    # Should show alongside Node
```

If not installed, get it from **nodejs.org** — download the LTS version.

### An Anthropic Account
You need either:
- A **Claude.ai account** (Pro, Team, or Enterprise plan) — for billing through claude.ai
- An **Anthropic API key** — for billing directly through the API

> ⚠️ Free Claude.ai accounts **do not** include Claude Code access.

---

## 2. Installation

Claude Code is installed as a global npm package:

```bash
npm install -g @anthropic-ai/claude-code
```

That's it. One command. This installs the `claude` binary globally so you can run it from any directory.

**Verify the installation:**
```bash
claude --version
```

### Platform-Specific Notes

| Platform | Notes |
|---|---|
| **macOS** | Works out of the box. No extra steps. |
| **Linux** | Works out of the box on most distributions. |
| **Windows** | Requires **Git for Windows** to be installed first (provides the bash shell layer). Install from gitforwindows.org, then run npm install inside Git Bash. |
| **WSL** | Fully supported and recommended Windows experience. Treat it like Linux. |

### Auto-Updates
Claude Code updates itself automatically when new versions are released. To force an update manually:
```bash
npm update -g @anthropic-ai/claude-code
```

---

## 3. Authentication

When you run `claude` for the first time, it prompts you to authenticate. Two paths:

### Path A — Claude.ai Account (Recommended for most)
```bash
claude
# First-run wizard launches
# Choose: "Login with Claude.ai"
# Browser opens → you authorize → done
```

This ties Claude Code to your Claude.ai subscription. Usage costs come out of your plan.

### Path B — API Key
```bash
claude config set apiKey YOUR_API_KEY
```

Or set it as an environment variable (useful for CI/CD):
```bash
export ANTHROPIC_API_KEY="your-key-here"
```

Claude Code automatically picks up the `ANTHROPIC_API_KEY` environment variable — no config command needed.

**Check your current auth status:**
```bash
claude config list
```

---

## 4. Your First `claude` Session

Navigate to any project directory (or create a test one):

```bash
mkdir my-test-project
cd my-test-project
echo "print('hello world')" > app.py
claude
```

You'll see something like:

```
╭─────────────────────────────────────╮
│ ✻ Welcome to Claude Code!           │
│                                     │
│ /help for commands, /exit to quit   │
╰─────────────────────────────────────╯

claude>
```

You're now in an **interactive session**. Claude Code has already scanned your directory. Try:

```
claude> What files are in this project and what do they do?
```

Claude will read your files and respond. Watch closely — you'll see it use the **Read** tool in real time, shown in the terminal output before its answer.

---

## 5. The Terminal Interface — A Full Tour

### The Prompt Area
```
claude>
```
Accepts: natural language tasks, slash commands (start with `/`), and follow-up messages mid-task.

### The Tool Use Display
When Claude acts, it shows you what it's doing:

```
● Reading file: app.py
● Running: python app.py
● Editing: app.py
  ┌─ diff ──────────────────┐
  │ - print('hello world')  │
  │ + print('Hello, World!') │
  └──────────────────────────┘
  Apply this change? [y/n]
```

### Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Enter` | Send your message |
| `Shift + Enter` | New line without sending (multi-line input) |
| `Escape` | Interrupt Claude mid-task |
| `Escape × 2` | Undo last action / restore checkpoint |
| `Ctrl + C` | Exit session |
| `↑ / ↓` arrows | Scroll through prompt history |

> 💡 **The Escape key is your safety net.** If Claude starts doing something unexpected mid-task, press Escape immediately. It stops the agent loop. You can then redirect, undo with `Escape × 2`, or start fresh with `/clear`.

---

## 6. All Core Slash Commands

### Session Management
| Command | What it does |
|---|---|
| `/help` | Lists all slash commands with descriptions |
| `/exit` | Ends the current session cleanly |
| `/clear` | Clears conversation history and starts fresh (keeps files) |
| `/compact` | Summarizes conversation to free up context window space |
| `/reset` | Full reset — clears history AND any in-memory context |

### Model & Configuration
| Command | What it does |
|---|---|
| `/model` | Shows current model being used |
| `/model claude-opus-4-5` | Switches to a specific model mid-session |
| `/config` | Opens configuration settings |

### Memory & Context
| Command | What it does |
|---|---|
| `/memory` | Shows what Claude currently has in its active memory |
| `/add-dir <path>` | Adds an additional directory to Claude's context |

### Task & Workflow
| Command | What it does |
|---|---|
| `/todo` | Shows the current task list Claude is tracking |
| `/review` | Asks Claude to review recent changes it made |

### Recovery & Undo
| Command | What it does |
|---|---|
| `/rewind` | Reverts to a previous checkpoint |
| `Escape × 2` | Quick undo of the last change |

### Feedback & Debug
| Command | What it does |
|---|---|
| `/bug` | Opens a bug report to send to Anthropic |
| `/insights` | Shows stats on your session (tokens used, tools called, etc.) |
| `/status` | Shows current session status and health |

### Git Shortcuts
| Command | What it does |
|---|---|
| `/commit` | Asks Claude to stage and commit current changes |
| `/pr` | Asks Claude to create a pull request |
| `/diff` | Shows git diff of current changes |

---

## 7. CLI Flags — Running Claude Without Interactive Mode

```bash
# One-shot task (no interactive session)
claude "write a function that validates email addresses"

# Run on a specific directory without cd-ing
claude --dir /path/to/project "explain this codebase"

# Use a specific model
claude --model claude-opus-4-5 "refactor my auth module"

# Run without any permission prompts (auto-approves edits)
claude --yes "fix all lint errors"

# Print output to stdout (good for piping)
claude --print "summarize main.py"

# Run in non-interactive mode (for CI/CD pipelines)
claude --no-interactive "run tests and report failures"

# Limit how many tokens Claude can use
claude --max-tokens 4000 "review this file"
```

> ⚠️ The `--yes` flag auto-approves all changes without asking. Use carefully — fine for low-risk tasks, risky for anything destructive.

---

## 8. Configuration — `claude config`

Claude Code stores configuration in `~/.claude/` on your system:

```bash
claude config list          # View all config values
claude config set theme dark  # Set a value
claude config get apiKey    # Get a specific value
claude config reset         # Reset to defaults
```

### Key Configuration Options

| Config Key | What it controls |
|---|---|
| `apiKey` | Your Anthropic API key |
| `model` | Default model for all sessions |
| `theme` | Terminal color theme (`dark` / `light`) |
| `autoApprove` | Skip confirmation on file edits (default: false) |
| `maxTokens` | Default token limit per request |
| `editor` | Preferred editor for opening files |

---

## 9. Internal Mechanics — How Claude Code Bootstraps

Here's exactly what happens in the first few seconds after you type `claude`:

```
Step 1: DIRECTORY SCAN
   └── Claude reads your current directory tree
   └── Identifies file types, languages, frameworks
   └── Notes presence of: package.json, pyproject.toml,
       Makefile, .git, etc.

Step 2: CLAUDE.md CHECK
   └── Looks for CLAUDE.md in current dir
   └── Looks for global CLAUDE.md in ~/.claude/
   └── Loads instructions into system context if found

Step 3: GIT CONTEXT
   └── Runs `git status` and `git log --oneline -10`
   └── Understands what's changed, what branch you're on

Step 4: SYSTEM PROMPT CONSTRUCTION
   └── Combines: base instructions + project context +
       CLAUDE.md contents + git context
   └── This becomes the "world" Claude operates in

Step 5: SESSION OPENS
   └── Interactive prompt appears
   └── Claude is ready, already knowing your project's shape
```

### The `~/.claude/` Directory

```
~/.claude/
├── config.json          ← your settings
├── CLAUDE.md            ← your global instructions (applies everywhere)
├── sessions/            ← session history logs
├── checkpoints/         ← undo snapshots (30 day retention)
└── mcp/                 ← MCP server configs (Chapter 9)
```

---

## 10. The `.claudeignore` File

If your project has large auto-generated directories (like `node_modules`, `dist`, `.next`), Claude wastes time scanning them. Create a `.claudeignore`:

```
node_modules/
dist/
.next/
build/
*.log
*.lock
```

Same syntax as `.gitignore`. Claude skips these during its directory scan — significantly speeding up startup on large projects.

---

## 11. Undo System — `Escape × 2` vs `/rewind`

These two are different and commonly confused:

| | `Escape × 2` | `/rewind` |
|---|---|---|
| **Scope** | One action | Any checkpoint |
| **Speed** | Instant | Shows a menu |
| **Granularity** | Single tool call | Whole session snapshots |
| **Use when** | Claude just did one wrong thing | Claude went down a wrong path for a while |
| **Conversation** | Not affected | Can restore or keep |

**`Escape × 2`** = *"undo that one thing you just did"*
**`/rewind`** = *"take me back to before this whole approach"*

Checkpoints are created automatically after every prompt. Retained for 30 days in `~/.claude/checkpoints/`.

---

## 12. Common Setup Problems & Fixes

| Problem | Cause | Fix |
|---|---|---|
| `claude: command not found` | npm global bin not in PATH | `export PATH="$PATH:$(npm bin -g)"` |
| Auth loop keeps repeating | Browser didn't complete auth | `claude config set apiKey YOUR_KEY` instead |
| `EACCES` permission error on install | npm global directory permissions | `sudo npm install -g` or fix npm permissions |
| Windows: `bash not found` | Git for Windows not installed | Install Git for Windows first |
| Claude seems slow to start | Large directory being scanned | Add a `.claudeignore` file |

---

## 13. Practical Exercise

**Step 1:** Create a test project:
```bash
mkdir claude-practice && cd claude-practice && git init
```

Create `calculator.py`:
```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    return a / b  # Bug: no zero division check
```

**Step 2:** Start Claude Code:
```bash
claude
```

**Step 3:** Try each of these prompts and observe the tool usage:
```
What's in this project?
Is there anything wrong with calculator.py?
Fix the bug and add proper error handling
Add a test file for these functions
```

**Step 4:** Run the slash commands:
```
/diff
/todo
/insights
```

**Step 5:** Practice interrupting:
```
Rewrite the entire calculator to use a class-based approach
```
→ While it's running, press `Escape`
→ Then try `Escape × 2` to undo

---

## Chapter 2 Summary

| Topic | Key Takeaway |
|---|---|
| Installation | `npm install -g @anthropic-ai/claude-code` — one command |
| Auth | claude.ai account or `ANTHROPIC_API_KEY` env variable |
| Starting | `cd your-project && claude` |
| Interface | Tool use is visible, diffs shown before applying |
| Escape key | Your interrupt and undo safety net |
| Slash commands | `/help`, `/clear`, `/compact`, `/rewind`, `/insights` — memorize these |
| Bootstrap | Claude scans dir + git + CLAUDE.md before you type anything |
| Config | Lives in `~/.claude/config.json` |
| `.claudeignore` | Speeds up large projects by skipping irrelevant dirs |

---

*Previous: [Chapter 1 — What Is Claude Code?](./chapter-01-what-is-claude-code.md)*
*Next: [Chapter 3 — Basic Interaction & Core Commands](./chapter-03-basic-interaction-and-core-commands.md)*
