# Developer Guide to Claude Code

A comprehensive, structured guide to mastering Claude Code — from first install to advanced agentic workflows and Claude Certified Architect (CCA) exam preparation.

---

## How to Use This Guide

Each chapter builds on the last. Read sequentially for the full learning experience, or jump to a specific topic using the index below. Every chapter covers:

- **Concepts** — what it is and why it matters
- **Internal Mechanics** — how Claude Code implements it under the hood
- **Hands-On Practice** — step-by-step exercises
- **Common Mistakes** — what to watch out for

---

## Table of Contents

### Part 1 — Foundations (Beginner)

| Chapter | Topic | CCA Domains |
|---|---|---|
| [Chapter 1](./chapter-01-what-is-claude-code.md) | What Is Claude Code? | Domain 1 |
| [Chapter 2](./chapter-02-installation-and-setup.md) | Installation & Setup | Domain 3 |
| [Chapter 3](./chapter-03-basic-interaction-and-core-commands.md) | Basic Interaction & Core Commands | Domain 1, 4 |

### Part 2 — Core Workflows (Intermediate)

| Chapter | Topic | CCA Domains |
|---|---|---|
| [Chapter 4](./chapter-04-claude-md-persistent-context.md) | CLAUDE.md — Persistent Context & Project Memory | Domain 3, 5 |
| Chapter 5 *(coming soon)* | File Editing & Multi-File Tasks | Domain 1, 4 |
| Chapter 6 *(coming soon)* | Debugging & Bug Fixing | Domain 1 |
| Chapter 7 *(coming soon)* | Git Integration | Domain 3 |

### Part 3 — Advanced Usage

| Chapter | Topic | CCA Domains |
|---|---|---|
| Chapter 8 *(coming soon)* | Context Window Management | Domain 5 |
| Chapter 9 *(coming soon)* | MCP — Model Context Protocol | Domain 2 |
| Chapter 10 *(coming soon)* | Multi-Agent & Parallel Workflows | Domain 1, 2 |
| Chapter 11 *(coming soon)* | Checkpoints & Session Recovery | Domain 3 |
| Chapter 12 *(coming soon)* | CI/CD Automation | Domain 3 |

### Part 4 — Expert & Internal Architecture

| Chapter | Topic | CCA Domains |
|---|---|---|
| Chapter 13 *(coming soon)* | The Claude Agent SDK | Domain 2 |
| Chapter 14 *(coming soon)* | Security, Trust & Safety | Domain 6 |
| Chapter 15 *(coming soon)* | Output Quality & Verification | Domain 4 |
| Chapter 16 *(coming soon)* | Advanced CLAUDE.md & Custom Commands | Domain 3 |

---

## Claude Certified Architect (CCA) Exam Coverage

This guide is structured to cover all six CCA exam domains:

| Domain | Topic | Exam Weight |
|---|---|---|
| Domain 1 | Agentic Architecture & Tool Use | 27% |
| Domain 2 | Multi-Agent Systems & MCP | 18% |
| Domain 3 | Claude Code Configuration & Workflows | 20% |
| Domain 4 | Prompt Engineering & Structured Output | 15% |
| Domain 5 | Context & Memory Management | 15% |
| Domain 6 | Security, Safety & Responsible AI | 5% |

---

## Quick Reference — Commands

### CLI Entry Points
```bash
claude                          # Start interactive session
claude "your task here"         # One-shot task
claude --help                   # All flags and options
claude --version                # Installed version
claude --model <model-name>     # Specify model
claude --yes "task"             # Auto-approve all edits
claude --plan "task"            # Plan mode (shows plan before acting)
claude --print "task"           # Print output to stdout
claude --no-interactive "task"  # Non-interactive mode (CI/CD)
claude --dir /path "task"       # Run on specific directory
claude --max-tokens 4000 "task" # Limit token usage
```

### Slash Commands — Session
```
/help          List all commands
/exit          End session
/clear         Clear history (keep files)
/compact       Compress history to save context
/reset         Full reset
```

### Slash Commands — Context & Memory
```
/memory        Show active context and token usage
/add-dir       Add directory to context
/todo          Show Claude's current task list
```

### Slash Commands — Review & Quality
```
/review        Claude self-audits its changes
/diff          Show all git changes this session
/insights      Session stats (tokens, tools, cost)
/status        Session health check
```

### Slash Commands — Recovery
```
/rewind        Restore to a previous checkpoint
Escape × 2     Quick undo of last action
```

### Slash Commands — Git
```
/commit        Stage and commit changes
/pr            Create pull request
/diff          Show git diff
```

### Slash Commands — Model & Config
```
/model                    Show current model
/model claude-opus-4-5    Switch model mid-session
/config                   Open configuration
/bug                      Report a bug to Anthropic
```

---

## Key Concepts Quick Reference

| Concept | One-liner |
|---|---|
| Agent loop | Plan → Tool → Observe → Repeat |
| CLAUDE.md hierarchy | Global → Project → Subdirectory (most specific wins) |
| Programmatic vs prompt | Hard constraints must be code, not instructions |
| Context window | Everything accumulates; use `/compact` on long sessions |
| Plan mode | Claude shows plan and waits for approval before acting |
| Checkpoints | Auto-saved after every prompt, retained 30 days |
| `.claudeignore` | Skip irrelevant dirs to speed up large project scanning |
| MCP | Open protocol for connecting Claude to external services |

---

## Contributing

This guide is a living document. As new chapters are added or Claude Code updates, this repo will be updated accordingly. Feel free to open issues for corrections or suggestions.

---

*Guide version: March 2026 | Claude Code version: latest*
