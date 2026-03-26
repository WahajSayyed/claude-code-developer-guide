# Chapter 3: Basic Interaction & Core Commands

---

## 1. The Fundamental Mindset Shift

Before learning *how* to talk to Claude Code, internalize one idea:

**You are not prompting an AI. You are directing a developer.**

When you onboard a new developer, you don't say:
> *"Please utilize your programming expertise to potentially implement a solution for the authentication challenge we are experiencing."*

You say:
> *"The login endpoint is broken. It's returning 401 even with valid credentials. Fix it."*

Claude Code responds to the same directness. Vague, polite, over-qualified prompts produce vague results. Direct, specific, task-framed prompts produce precise action.

---

## 2. The Four Interaction Modes

### Mode 1: Question / Exploration
You want to *understand* something. Claude reads and explains, makes no changes.

```
What does the PaymentProcessor class do?
How does authentication flow through this app?
Why is this function O(n²)?
What dependencies does this project have and are any outdated?
```

### Mode 2: Task Execution
You want Claude to *do* something. It plans, acts, and verifies.

```
Add rate limiting to the /api/login endpoint
Write tests for the UserService class
Refactor this function to use async/await
Fix the failing test in test_payments.py
```

### Mode 3: Iterative Collaboration
You work back and forth, refining. Each message builds on the last.

```
You:     Add a caching layer to the database queries
Claude:  [adds Redis caching]
You:     Good, but the TTL should be configurable via environment variable
Claude:  [updates implementation]
You:     Also add a cache invalidation method
Claude:  [adds the method]
```

### Mode 4: One-Shot CLI
No interactive session. One command, one result, done.

```bash
claude "add docstrings to every function in utils.py"
claude "what is the database schema for this project?"
claude --print "summarize main.py" > summary.txt
```

---

## 3. Prompt Structure — What Makes a Great Prompt

Every strong Claude Code prompt has up to four components:

```
[CONTEXT] + [TASK] + [CONSTRAINTS] + [OUTPUT FORMAT]
```

### Component 1: Context
What Claude needs to know that it can't infer:
```
The app uses soft deletes — records are never truly deleted,
just marked with deleted_at timestamp.
```

### Component 2: Task
What you actually want done — verb-first, specific:
```
Add a scope to the User model that filters out soft-deleted records
by default on all queries.
```

### Component 3: Constraints
Boundaries it must respect:
```
Don't modify the existing migration files.
Keep backward compatibility with the existing API.
Use the same pattern as the Product model's soft delete implementation.
```

### Component 4: Output Format
How you want the result:
```
Show me the diff before applying.
Add comments explaining the changes.
Also update the tests.
```

**Full example combined:**
```
The app uses soft deletes (deleted_at timestamp). Add a default scope
to the User model that filters out soft-deleted records on all queries.
Don't touch existing migrations. Follow the same pattern as the Product
model. Show the diff before applying and update the relevant tests.
```

One prompt. Everything Claude needs. Right first time.

---

## 4. Prompt Patterns — Your Repeatable Toolkit

### The "Explain First" Pattern
Forces Claude to demonstrate understanding before acting. Prevents confident wrong action.
```
Before making any changes, explain how the current authentication
system works and what you plan to do. Then wait for my confirmation.
```

### The "Follow Existing Patterns" Pattern
Critical for consistency in real codebases:
```
Add input validation to the registration endpoint.
Follow the exact same pattern used in the login endpoint.
```

Without this, Claude invents its own approach which may not match your codebase conventions.

### The "Scope Limiter" Pattern
Prevents Claude from helpfully doing too much:
```
Only change user_service.py. Don't touch any other files.
```

### The "Verification First" Pattern
Ask Claude to find the problem before fixing it:
```
Find all the places in the codebase where we're not handling
database connection errors. List them. Don't fix anything yet.
```

### The "Step by Step" Pattern
For complex multi-part tasks, break them into explicit stages:
```
Do this in three steps:
1. First, add the database migration
2. Then update the model
3. Finally update the API endpoint

Complete each step and show me before moving to the next.
```

### The "Reference Implementation" Pattern
Point Claude at something that already works:
```
Write a new EmailNotificationService using the exact same
structure as the existing SMSNotificationService in
src/notifications/sms.py
```

### The "Rubber Duck" Pattern
Use Claude to think through a problem without writing code yet:
```
I need to add real-time notifications to this app. Talk me through
the different approaches I could take given this tech stack.
Don't write any code yet.
```

---

## 5. Reading Claude's Output — What to Watch For

### Tool Use Indicators
```
● Reading: src/auth/login.py          ← reading a file
● Searching: "def validate_token"     ← grepping the codebase
● Running: python -m pytest tests/    ← executing a command
● Writing: src/auth/login.py          ← making an edit
```

Watch these carefully. If Claude is reading files you didn't expect, it's either exploring smartly or going off course. If it's running commands you didn't authorize, press `Escape`.

### The Thinking Block
Sometimes Claude shows its reasoning before acting:
```
I need to understand how tokens are currently validated before
adding rate limiting. Let me read the auth module first...
```
This is Claude planning. A good sign — it means it's not blindly executing.

### Diff Display
Before applying any file edit:
```
  Editing: src/auth/login.py
  ┌──────────────────────────────────────┐
  │   def login(username, password):     │
  │ -     return authenticate(username,  │
  │ -                          password) │
  │ +     user = authenticate(username,  │
  │ +                          password) │
  │ +     if not user:                   │
  │ +         raise AuthError("Invalid") │
  │ +     return user                    │
  └──────────────────────────────────────┘
  Apply this change? [y/n/e]
```

`y` = apply, `n` = skip, `e` = open in editor to modify manually before applying.

> ⚠️ **Always read diffs.** This is your primary quality gate.

---

## 6. Slash Commands — Deep Practical Usage

### `/clear` — When to Use It
Clears conversation history. Files are untouched.

**Use when:**
- Finished one task, starting a completely unrelated one
- Conversation has gone in the wrong direction
- Claude seems confused by accumulated context

**Don't** use it mid-task — you'll lose the context Claude needs to complete what it started.

### `/compact` — The Context Saver
Compresses conversation history into a summary, freeing up token space while keeping important context.

**Use when:**
- Sessions run long
- Claude's responses are getting slower or less precise
- You want to continue a long session without starting over

Unlike `/clear`, `/compact` **keeps the project understanding** — it just shrinks how it's represented.

### `/memory` — Inspect What Claude Knows
```
/memory

Active context:
- Project: FastAPI invoicing backend
- CLAUDE.md: loaded (247 tokens)
- Files read this session: 8
- Conversation turns: 14
- Estimated tokens used: 18,400 / 200,000
```

Use to understand why Claude might be missing context, or to check how close you are to the context limit.

### `/todo` — Task Tracking
Claude maintains an internal todo list for multi-step tasks:
```
/todo

Current tasks:
[✓] Read existing auth implementation
[✓] Identify rate limiting insertion point
[ ] Add rate limiting middleware
[ ] Write tests
[ ] Update API documentation
```

Particularly useful for long tasks. If Claude seems to have forgotten a step, `/todo` shows the state of its internal plan.

### `/review` — Ask Claude to Self-Audit
After Claude makes changes, ask it to review its own work:
```
/review
```
Claude re-reads everything it changed in the session and gives you a critical assessment. It often catches things it missed the first time.

### `/insights` — Session Analytics
```
/insights

Session statistics:
- Duration: 34 minutes
- Prompts sent: 12
- Tools called: 47
- Files read: 15
- Files modified: 4
- Tokens used: 42,800
- Estimated cost: $0.18
```

### `/diff` — See All Changes This Session
Shows a combined git diff of every change Claude has made during the session. Your master view of what's changed before you commit.

---

## 7. Full Slash Command Reference

### Session Management
| Command | What it does |
|---|---|
| `/help` | Lists all slash commands with descriptions |
| `/exit` | Ends the current session cleanly |
| `/clear` | Clears conversation history, keeps files |
| `/compact` | Summarizes conversation to free context space |
| `/reset` | Full reset — clears history AND in-memory context |

### Model & Configuration
| Command | What it does |
|---|---|
| `/model` | Shows current model |
| `/model <name>` | Switches model mid-session |
| `/config` | Opens configuration settings |

### Memory & Context
| Command | What it does |
|---|---|
| `/memory` | Shows active memory and token usage |
| `/add-dir <path>` | Adds an additional directory to context |

### Task & Workflow
| Command | What it does |
|---|---|
| `/todo` | Shows current task list |
| `/review` | Claude self-audits its changes |

### Recovery & Undo
| Command | What it does |
|---|---|
| `/rewind` | Reverts to a previous checkpoint |
| `Escape × 2` | Quick undo of the last change |

### Feedback & Debug
| Command | What it does |
|---|---|
| `/bug` | Opens a bug report to Anthropic |
| `/insights` | Session stats (tokens, tools, cost) |
| `/status` | Current session status and health |

### Git Shortcuts
| Command | What it does |
|---|---|
| `/commit` | Stage and commit current changes |
| `/pr` | Create a pull request |
| `/diff` | Show git diff of all session changes |

---

## 8. Handling Claude When It Goes Wrong

### When it misunderstands:
```
That's not what I meant. Stop and let me clarify.

I don't want you to rewrite the whole function. I just want
you to add a null check at line 12. Only that.
```

Explicitly say "stop" and redirect. Don't pile on more instructions — that compounds confusion.

### When the approach is wrong:
```
/rewind
```
Pick the checkpoint before it went wrong. Restart with a better-scoped prompt.

### When one edit is wrong:
`Escape × 2` — undo that one action and redirect.

### When you want to reject a diff:
At the `Apply this change? [y/n/e]` prompt:
- `n` — skip this edit, Claude continues planning
- `e` — open in editor, you manually fix it, then Claude continues

### When Claude is confidently wrong:
```
I don't think that's right. The function is called from three
places — read all three call sites before deciding how to
change the signature.
```

Push back directly with specific reasoning. Claude responds well to being challenged.

---

## 9. Multi-turn Conversation — Keeping Context Sharp

### Re-anchor When Switching Topics
```
We're done with the auth refactor. Now switching to a completely
different problem: the PDF generation in src/pdf/generator.py
is running out of memory on large documents.
```

### Reference Specific Files and Lines
```
In src/payments/processor.py, line 84, the retry logic
is missing exponential backoff. Add it.
```

### Confirm Before Long Tasks
```
Before you start, tell me which files you're going to modify.
```

This forces Claude to plan explicitly. You can catch wrong assumptions before any code is written.

---

## 10. Internal Mechanics — How Claude Processes Your Prompt

```
Your prompt arrives
       ↓
Tokenization: your words → tokens
       ↓
Context assembly:
  [System prompt]
  + [CLAUDE.md contents]
  + [Conversation history]
  + [Your new prompt]
  = Full context sent to model
       ↓
Model generates a response
(mix of: text to show you + tool calls to execute)
       ↓
Tool calls extracted and executed:
  - Read file → file contents returned
  - Run command → stdout/stderr returned
  - Edit file → diff shown to you
       ↓
Tool results appended to context
       ↓
Model generates next response
(loop continues until task done or model responds to you)
```

### Why This Matters Practically

**Everything is in the context window.** Every file Claude reads, every command output, every conversation turn accumulates in one large context. When that context fills up, Claude starts losing early information first (recency bias).

This is why:
- `/compact` matters on long sessions
- Specific prompts work better than vague ones (less back-and-forth = less context used)
- Pointing Claude at specific files is more efficient than letting it explore freely

**Tool calls are synchronous within the loop.** Claude can't do two things at once. It reads a file, gets the result, decides what to do next, reads another file — sequentially.

---

## 11. Practical Exercise — The Full Interaction Workout

Use the calculator project from Chapter 2 or any project you have.

**Exploration prompts:**
```
Explain the overall structure of this project
What's the most complex function here and why?
Are there any obvious code quality issues?
```

**Task prompts:**
```
Add type hints to all functions
Add a modulo operation function following the same pattern as the others
Write a comprehensive docstring for each function
```

**Pattern practice:**
```
Before changing anything, explain what changes you'd make to add
logging to every function. Then wait for my approval.
```

**Slash command workout — run each one:**
```
/todo
/diff
/memory
/insights
/compact
/review
```

**Recovery practice:**
- Let Claude start editing something
- Press `Escape` to interrupt
- Use `Escape × 2` to undo
- Use `/rewind` to go back further

---

## Chapter 3 Summary

| Topic | Key Takeaway |
|---|---|
| Mindset | Direct a developer, don't prompt an AI |
| Interaction modes | Question / Task / Iterative / One-shot |
| Prompt structure | Context + Task + Constraints + Output format |
| Key patterns | Explain first, follow existing, scope limiter, step by step |
| Reading output | Watch tool use indicators and always read diffs |
| Daily slash commands | `/clear`, `/compact`, `/memory`, `/todo`, `/review`, `/diff`, `/insights` |
| When things go wrong | Escape, Escape×2, /rewind, redirect with specific language |
| Internal mechanics | Everything is context; tools run sequentially in a loop |

---

*Previous: [Chapter 2 — Installation & Setup](./chapter-02-installation-and-setup.md)*
*Next: [Chapter 4 — CLAUDE.md: Persistent Context & Project Memory](./chapter-04-claude-md-persistent-context.md)*
