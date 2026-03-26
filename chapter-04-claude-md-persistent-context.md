# Chapter 4: CLAUDE.md — Persistent Context & Project Memory

> **CCA Exam Note:** Domain 3 (Claude Code Configuration & Workflows) carries 20% of the Claude Certified Architect exam weight and gets specifically tested on CLAUDE.md hierarchies, skills frontmatter, and plan mode. This chapter directly covers that domain.

---

## 1. The Core Problem CLAUDE.md Solves

Every time you start a new Claude Code session, Claude starts fresh. It has no memory of:
- The architectural decisions your team made
- The naming conventions you follow
- Which files are critical vs boilerplate
- The gotchas specific to your project
- Your team's workflow and review process

Without CLAUDE.md, you repeat this context every single session.

```
Without CLAUDE.md:
  Every session → You = a developer onboarding a new hire from scratch
  Every session → Claude = a brilliant developer who forgot everything overnight

With CLAUDE.md:
  Every session → You = a developer giving a quick task to a senior who knows the codebase
  Every session → Claude = that senior developer, already context-loaded
```

CLAUDE.md is your project's "tech lead document" — it defines architectural rules, naming conventions, and workflow expectations that Claude follows throughout every development session.

---

## 2. The CLAUDE.md Hierarchy — Exam Critical ⚠️

This is one of the most tested specifics on the CCA exam. There are **three levels** of CLAUDE.md, loaded in a specific order with a specific precedence:

```
Level 1: GLOBAL
~/.claude/CLAUDE.md
  └── Applies to ALL projects on your machine
  └── Your personal preferences, tools, style
  └── Loaded first — lowest precedence

Level 2: PROJECT ROOT
/your-project/CLAUDE.md
  └── Applies to this entire project
  └── Architecture, conventions, key files
  └── Loaded second — overrides global on conflicts

Level 3: SUBDIRECTORY
/your-project/src/payments/CLAUDE.md
  └── Applies only when working in that directory
  └── Module-specific rules and context
  └── Loaded last — highest precedence
```

**Precedence rule: More specific = higher precedence.**
Subdirectory overrides project root, which overrides global.

**Exam question pattern:** "Which CLAUDE.md applies when Claude is editing a file in `src/payments/`?"
**Answer:** All three are loaded, but `src/payments/CLAUDE.md` wins on any conflicting instructions.

### Loading Mechanics Internally

When Claude Code bootstraps:
```
1. Reads ~/.claude/CLAUDE.md → loads into system context
2. Reads /project/CLAUDE.md → appends, overrides conflicts
3. As you navigate subdirectories → reads local CLAUDE.md → appends
4. Total combined content → injected at TOP of every prompt's system context
```

> 🔑 **Key insight:** CLAUDE.md content is not sent once at session start — it's included in **every single API call** Claude Code makes during the session. This is why it consistently enforces rules — the instructions are always present.

---

## 3. What to Put in CLAUDE.md — The Full Anatomy

A production-grade CLAUDE.md has six sections:

### Section 1: Project Identity
```markdown
# Project: PayFlow

PayFlow is a B2B invoicing SaaS built on FastAPI (Python 3.11)
with a PostgreSQL database. It handles invoice creation, PDF
generation, Stripe payment processing, and automated email delivery.

Primary business flow:
Invoice created → PDF rendered → Email sent → Payment tracked
via Stripe webhook → Reconciliation report generated nightly.
```

**Why:** Claude needs to understand *what the system does* to make good architectural decisions. Without this, it treats every file in isolation.

### Section 2: Architecture Overview
```markdown
## Architecture

- **API Layer**: FastAPI routers in src/api/
- **Business Logic**: Service classes in src/services/
- **Data Layer**: SQLAlchemy models in src/models/,
  Alembic migrations in migrations/
- **Background Jobs**: Celery workers in src/workers/
- **PDF Generation**: WeasyPrint in src/pdf/
- **External Services**: Stripe (src/integrations/stripe.py),
  SendGrid (src/integrations/email.py)

Key architectural rule: Business logic NEVER goes in routers.
Routers call services. Services call models/integrations.
```

**Why:** Prevents Claude from putting business logic in the wrong layer — one of the most common AI coding mistakes.

### Section 3: Coding Conventions
```markdown
## Conventions

- All service methods must be async
- Use Pydantic v2 models for all request/response schemas
- Database queries always go through the repository pattern
  (src/repositories/)
- Never use raw SQL — always SQLAlchemy ORM
- All monetary values stored as integers (cents), never floats
- Soft deletes only — never hard delete records
- Every API endpoint needs a corresponding test in tests/api/
- Type hints required on all function signatures
```

**Why:** Encodes your team's standards. Claude follows these like laws, not suggestions.

### Section 4: Key Files Reference
```markdown
## Key Files

- `src/core/config.py` — all environment variables and settings
- `src/core/database.py` — database session management
- `src/core/dependencies.py` — FastAPI dependency injection
- `src/services/invoice.py` — main invoice business logic
- `src/workers/pdf_worker.py` — async PDF generation
- `tests/conftest.py` — shared test fixtures

When adding a new feature, always check these files first
for existing patterns to follow.
```

**Why:** Speeds up Claude enormously on large projects. Instead of grepping everything, it goes straight to the right files.

### Section 5: What to Avoid
```markdown
## Do Not

- Do not modify migration files — create new ones instead
- Do not use print() for logging — use the logger from
  src/core/logging.py
- Do not add dependencies to pyproject.toml without noting
  why in your response
- Do not use synchronous database calls in async endpoints
- Do not hardcode configuration values — always use
  src/core/config.py
```

**Why:** Negative constraints are as important as positive ones. This is where you prevent Claude from making your team's most common mistakes.

### Section 6: Workflow Instructions
```markdown
## Workflow

Before making any changes:
1. Explain what you plan to do and which files you'll modify
2. Wait for confirmation before proceeding

After making changes:
1. Run: pytest tests/ -x
2. Run: ruff check src/
3. Report any failures before considering the task done

For database changes:
1. Always create a new Alembic migration
2. Never modify existing migration files
3. Test migration up AND down
```

**Why:** Encodes your development process directly into Claude's behavior. It becomes your team's process enforcer.

---

## 4. Global CLAUDE.md — Your Personal Developer Profile

The global `~/.claude/CLAUDE.md` applies to every project on your machine:

```markdown
# Global Developer Preferences

## My Style
- I prefer explicit over clever code
- Always add type hints
- Prefer composition over inheritance
- Keep functions under 30 lines when possible

## Tools I Use
- Python 3.11+ for backend work
- pytest for testing
- ruff for linting
- black for formatting (line length 88)

## How I Like to Work
- Always show me diffs before applying changes
- Explain your reasoning for non-obvious decisions
- When in doubt, ask rather than assume
- Flag any security concerns you notice

## Things I Hate
- Magic numbers without explanation
- Functions that do more than one thing
- Commented-out code left in files
- print() statements left in production code
```

> If a project CLAUDE.md contradicts the global one, the project CLAUDE.md wins.

---

## 5. Subdirectory CLAUDE.md — Module-Level Context

For large codebases, put CLAUDE.md files inside specific modules:

```
src/
├── CLAUDE.md          ← project root (architecture)
├── payments/
│   └── CLAUDE.md      ← payment module specific
├── pdf/
│   └── CLAUDE.md      ← PDF generation specific
└── api/
    └── CLAUDE.md      ← API layer specific
```

Example `src/payments/CLAUDE.md`:
```markdown
# Payments Module

This module handles all Stripe integration and payment processing.

## Critical Rules
- ALL Stripe API calls must go through StripeClient
  in stripe_client.py — never call stripe directly
- Every payment operation must be idempotent
  (use idempotency keys)
- All amounts are in cents (integer), never dollars (float)
- Payment failures must be logged to payment_errors table
- Webhook events must be verified with Stripe signature
  before processing

## Testing
- Use stripe-mock for unit tests (already configured in conftest)
- Never call real Stripe API in tests
- All payment tests live in tests/payments/
```

When Claude is working in `src/payments/`, it loads all three levels: global + project root + this subdirectory file.

---

## 6. Programmatic Enforcement vs Prompt-Based Guidance

> ⚠️ **This is the single most tested concept on the CCA exam.** When a system behavior needs to be guaranteed, the exam consistently rewards the programmatic solution over the "add it to the prompt" solution.

### Prompt-Based Guidance (Weak)
```markdown
# In CLAUDE.md
Always validate user input before processing.
Never use floats for monetary values.
```

This is a **suggestion**. Claude will usually follow it but:
- Can be overridden by a conflicting instruction in your prompt
- Doesn't fail-safe if Claude forgets or misinterprets
- Has no enforcement mechanism

### Programmatic Enforcement (Strong)
```python
# In your actual code
class MonetaryValue:
    def __init__(self, cents: int):
        if not isinstance(cents, int):
            raise TypeError("Monetary values must be integers (cents)")
        self.cents = cents
```

Now it's **impossible** to use a float for money — the code enforces it, not the prompt.

### The Decision Framework

```
For behavior that must ALWAYS happen    → Programmatic enforcement
For behavior that should usually happen → Prompt-based guidance
For behavior that's a preference        → CLAUDE.md convention
```

**Exam question pattern:**
> "You need to ensure Claude always validates JSON output before returning it. What is the most reliable approach?"

❌ Wrong: "Add 'always validate JSON output' to CLAUDE.md"
✅ Right: "Implement a validation loop with JSON schema that retries on failure"

---

## 7. Skills Frontmatter — Advanced CLAUDE.md Feature

Skills are reusable, composable instruction sets. Each skill file has a **frontmatter header**:

```markdown
---
name: api-endpoint-creator
description: Use when creating new FastAPI endpoints
triggers: ["new endpoint", "add route", "create API"]
version: 1.0
---

# Skill: API Endpoint Creator

When creating a new endpoint, always follow this exact pattern:

1. Create router function in src/api/{module}.py
2. Create Pydantic schema in src/schemas/{module}.py
3. Create service method in src/services/{module}.py
4. Add test in tests/api/test_{module}.py
5. Update API documentation
```

### Skills Frontmatter Fields

| Field | Purpose |
|---|---|
| `name` | Unique identifier for the skill |
| `description` | When Claude should use this skill |
| `triggers` | Phrases that auto-activate this skill |
| `version` | For tracking and updating skills |

Skills live in `.claude/skills/` in your project directory. They're loaded on-demand, not upfront — keeping your base context lean while giving Claude deep expertise on demand.

---

## 8. Plan Mode — A CLAUDE.md-Controlled Behavior

Plan mode changes how Claude executes tasks:

### Normal Mode (Default)
```
Task received → Plan → Act → Verify → Done
```
Fast but less controllable for complex tasks.

### Plan Mode
```
Task received → Plan → STOP → Show plan → Wait for approval → Act → Verify → Done
```
Claude plans, shows you the complete plan, and waits for your go-ahead before touching anything.

### Enabling Plan Mode

**In CLAUDE.md:**
```markdown
## Workflow Mode

Use plan mode for:
- Any task modifying more than 3 files
- Database schema changes
- Any refactoring task
- Changes to authentication/authorization code

For simple tasks (single file, bug fix), proceed directly.
```

**Via CLI flag:**
```bash
claude --plan "refactor the authentication module"
```

**Why it matters for the exam:** Plan mode is a **process control** mechanism. Questions about when to use it test your judgment about risk vs. speed tradeoffs in production systems.

---

## 9. CLAUDE.md Anti-Patterns — What Not to Do

### ❌ Anti-Pattern 1: The Novel
```markdown
# CLAUDE.md (BAD)
This project is a very comprehensive and feature-rich application
that was built over many years by a talented team... [500 more words]
```
**Problem:** Token waste. Every word of CLAUDE.md consumes context window on every prompt. Be ruthlessly concise.

### ❌ Anti-Pattern 2: Vague Instructions
```markdown
# CLAUDE.md (BAD)
Write good code.
Follow best practices.
Be careful with security.
```
**Problem:** Meaningless without specifics. Say exactly what you mean.

### ❌ Anti-Pattern 3: Security by Prompt
```markdown
# CLAUDE.md (BAD)
Never expose user passwords in API responses.
Always sanitize SQL inputs.
```
**Problem:** Security constraints must be programmatic. CLAUDE.md is advisory. A prompt can override it. Your code cannot be.

### ❌ Anti-Pattern 4: Stale CLAUDE.md
If your architecture evolves but CLAUDE.md doesn't, you create contradictions. Claude gets confused when CLAUDE.md says "use MongoDB" but the codebase uses PostgreSQL.

**Fix:** Treat CLAUDE.md like code — version control it, review it in PRs, update it when architecture changes.

### ❌ Anti-Pattern 5: One Giant CLAUDE.md
A 2000-token CLAUDE.md covering everything is worse than a 400-token one covering essentials.

**Fix:** Use the hierarchy. Put global preferences in `~/.claude/CLAUDE.md`, project architecture in `/project/CLAUDE.md`, and module details in subdirectory files.

---

## 10. Internal Mechanics — How CLAUDE.md Is Processed

```
Claude Code starts
       ↓
File system walk:
  ~/.claude/CLAUDE.md → read → tokenize
  /project/CLAUDE.md  → read → tokenize
  /project/src/CLAUDE.md → read (if in src/) → tokenize
       ↓
Conflict resolution:
  Same instruction at multiple levels? → Most specific wins
  Complementary instructions?         → All kept, appended in order
       ↓
System prompt assembly:
  [Base Claude Code instructions]
  + [Global CLAUDE.md content]
  + [Project CLAUDE.md content]
  + [Subdirectory CLAUDE.md content]
  = Final system prompt
       ↓
This system prompt is sent with EVERY API call in the session
```

---

## 11. CCA Exam — Practice Questions

**Q1:** Your team wants Claude Code to always run `pytest` after making changes, but only for the backend module. Where should this instruction go?
- A) Global CLAUDE.md
- B) Project root CLAUDE.md
- C) Backend subdirectory CLAUDE.md ✅
- D) CLI flag at runtime

**Q2:** A developer added "never expose API keys in logs" to CLAUDE.md. Is this sufficient for a production system?
- A) Yes, CLAUDE.md instructions are always followed
- B) No, this must be enforced programmatically in the logging layer ✅
- C) Yes, if placed in the global CLAUDE.md
- D) No, use a CLI flag instead

**Q3:** Claude is following outdated architecture conventions from CLAUDE.md even though the codebase was refactored 3 months ago. What is the root cause?
- A) CLAUDE.md precedence hierarchy bug
- B) CLAUDE.md was not updated to reflect the architectural change ✅
- C) Subdirectory CLAUDE.md is overriding project root
- D) Global CLAUDE.md is conflicting

**Q4:** Which CLAUDE.md structure is most appropriate for a large monorepo with 8 microservices?
- A) One large CLAUDE.md at the repo root
- B) Global CLAUDE.md + one per microservice directory ✅
- C) Only subdirectory CLAUDE.md files, no root
- D) Duplicate CLAUDE.md in every directory

**Q5:** A developer wants Claude to pause and show its plan before modifying authentication code. What is the best approach?
- A) Remind Claude verbally before each task
- B) Add plan mode instructions to CLAUDE.md for auth-related tasks ✅
- C) Use `/clear` before each auth task
- D) Use a different model for auth tasks

---

## 12. Practical Exercise — Build a Production CLAUDE.md

**Step 1:** Create the global file:
```bash
nano ~/.claude/CLAUDE.md
```
Add your personal preferences, tools, and working style.

**Step 2:** Create the project file using all six sections:
```bash
cd your-project
nano CLAUDE.md
```

**Step 3:** Test it:
```bash
claude
What is this project and how is it structured?
```
Claude should answer using your CLAUDE.md context, not just by reading files.

**Step 4:** Verify it loaded:
```
/memory
```
You should see CLAUDE.md listed as loaded with its token count.

**Step 5:** Test a constraint:
Write a convention in CLAUDE.md, then ask Claude to do something that would violate it. Verify it refuses or flags the violation.

**Step 6:** Test programmatic vs prompt enforcement:
- Add a rule to CLAUDE.md: "Never use floats for prices"
- Ask Claude to add a price field to a model
- Then add actual type validation in your code
- Notice the difference in reliability

---

## Chapter 4 Summary

| Concept | Key Takeaway | CCA Exam Weight |
|---|---|---|
| CLAUDE.md hierarchy | Global → Project → Subdirectory, most specific wins | High |
| Six sections | Identity, Architecture, Conventions, Key Files, Do Not, Workflow | High |
| Skills frontmatter | `name`, `description`, `triggers`, `version` fields | Medium |
| Plan mode | Pause-before-act for high-risk tasks | Medium |
| Programmatic vs prompt | Hard constraints must be code, not instructions | Very High |
| Anti-patterns | Vague, stale, oversized, security-by-prompt | High |
| Internal mechanics | Injected into every API call, not just session start | Medium |

### CCA Domain Coverage

| CCA Domain | Concepts Covered Here |
|---|---|
| **Domain 3: Claude Code Config & Workflows (20%)** | CLAUDE.md hierarchy, skills frontmatter, plan mode |
| **Domain 1: Agentic Architecture (27%)** | Programmatic vs prompt enforcement |
| **Domain 5: Context Management (15%)** | Token efficiency, CLAUDE.md sizing |

---

## ✅ Before Chapter 5

- [ ] Written a global CLAUDE.md with your personal preferences
- [ ] Written a project CLAUDE.md using all six sections
- [ ] Verified it loaded with `/memory`
- [ ] Tested at least one constraint from CLAUDE.md
- [ ] Internalized the programmatic vs prompt enforcement distinction
- [ ] Answered all five practice questions correctly

---

*Previous: [Chapter 3 — Basic Interaction & Core Commands](./chapter-03-basic-interaction-and-core-commands.md)*
*Next: Chapter 5 — File Editing & Multi-File Tasks (coming soon)*
