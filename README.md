# 🤖 odoo-agent

> A personal AI development agent for Odoo 17 — automated code auditing, module analysis, and guided deployment via Claude Code + SSH.

---

## The Problem

Odoo development has a hidden bottleneck: **most time is not spent writing code — it's spent understanding what already exists.**

Every module you touch can break three others you don't know about. You have to read inherited models, check deprecated patterns, verify security files, and make sure you're not overwriting something a third-party module already defined. Before writing a single line, you're doing archaeology.

Manual code reviews before merging to production are slow, inconsistent, and easy to skip under pressure. Silent bugs reach production undetected.

## The Solution

An AI agent connected directly to a development environment:

- **Reads the full repository** — models, manifests, inheritance trees, dependencies
- **Connects to Odoo.sh via SSH** — runs updates, checks logs, inspects deployed files
- **Audits before merge** — finds deprecated patterns, duplicate methods, security gaps, and inheritance risks
- **Proposes commits, never executes them** — the developer stays in control

I built this for my own day-to-day Odoo work, and I maintain it.

---

## Live Demo — Branch Audit

**Prompt:**
```
Compare branches dev vs main and audit every module not yet in production.
Check naming conventions, deprecated patterns, security files, inheritance risks.
Report with status: READY / NEEDS WORK / REVIEW REQUIRED
```

**Result in 4 minutes 22 seconds:**

| Module | Status | Blocking Issue |
|--------|--------|----------------|
| custom_room_booking | ✅ READY | None |
| custom_personal_calendar | ⚠️ NEEDS WORK | Empty views (WIP) |
| custom_sale_createdby | ⚠️ NEEDS WORK | Wrong author in manifest |
| custom_product_autocomplete | ⚠️ NEEDS WORK | 3 duplicate `@constrains` + 1 duplicate `@onchange` — silent dead code |
| custom_purchase_from_sale | 🔴 REVIEW REQUIRED | Flagged a possible double-procurement path in `_action_confirm` |

**A real find, and how it played out:** the agent flagged `custom_purchase_from_sale` for a possible double-procurement bug. I traced it by hand: the flagged path was actually a false positive — a `skip_procurement` context already prevented the scenario the agent described. But the audit was right to be suspicious. Reading the method more closely turned up a real, narrower gap: no state guard against re-entrant calls to the confirm method. I verified against production data that it hasn't caused a single duplicate order in practice, and it's now a tracked, documented risk rather than an unknown one.

That's the actual value of this tool: it doesn't replace judgment, it gives you something concrete to investigate. The agent's first read was wrong in its specifics and useful in its instinct.

---

## Quick Start

### Prerequisites
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- SSH access to your Odoo.sh instance
- Git repository with your Odoo modules

### Setup

```bash
# 1. Clone this repo into your Odoo modules directory
git clone https://github.com/jdvillasmil/odoo-agent.git
cd odoo-agent

# 2. Configure your environment
cp .env.example .env
# Edit .env with your Odoo.sh SSH credentials and database name

# 3. Open Claude Code
claude
```

### Configure `.env`

```bash
ODOO_SSH=your_build_id@your-instance.dev.odoo.com
ODOO_DB=your-database-name
```

### Start using

```
# Audit all modules not yet in production
Compare branches dev vs main and audit every module not in main yet.
Check: naming conventions, deprecated patterns, security files, inheritance risks.
Report with status: READY / NEEDS WORK / REVIEW REQUIRED

# Check a specific module
Read custom_purchase_from_sale and find any deprecated Odoo 17 patterns,
inheritance risks, or logic issues before merging to main.

# Deploy and verify
Update module custom_room_booking via SSH, then check logs for errors.
```

---

## Architecture

```
You (requirement)
       ↓
  Claude (claude.ai)
  Strategy · Analysis · Business Logic
       ↓
  Claude Code — odoo-agent
  Reader · Auditor · Coder · Deployer
       ↓
  Odoo.sh Server (SSH)
  odoo-update · logs · file inspection
```

### Rules the agent follows (defined in `CLAUDE.md`)
These are instructions to the model, not a hard technical sandbox — they shape its behavior, they don't enforce it at the infrastructure level.
- Operates only on the development branch — stops and alerts if it detects `main`/`master`
- Only writes to modules under a configured prefix — third-party and native Odoo modules are read-only
- Never executes `git push`, `git merge`, or any destructive command
- Proposes commit messages — the developer runs `git add`/`git commit`
- SSH credentials stay in `.env` — never committed to the repository

---

## A known limitation

Session memory (`LOG_PROGRESO.md`) is meant to give the agent continuity across sessions. In practice, results from an in-chat investigation don't always get persisted before the conversation is compacted, so verified findings from one session can be unavailable in the next. This is an open problem I'm actively working on — see Phase 2 below.

---

## Roadmap

### ✅ Phase 1 — Foundation
- Claude Code connected to repo + Odoo.sh via SSH
- CLAUDE.md with security rules, module hierarchy, business context
- LOG_PROGRESO.md for session memory between runs
- Branch audit: finds deprecated patterns, duplicate methods, inheritance risks

### 🔄 Phase 2 — Reliable memory + database access
- Persist verified findings (query results, confirmed fixes) so they survive session compaction, not just a 3-line summary
- Python proxy script on the server that accepts ORM expressions and returns JSON
- Allows the agent to query live data without interactive shell limitations

### 📋 Phase 3 — Browser Integration (planned)
- Connect `chrome-devtools-mcp` to a dedicated Chrome profile (dev only)
- Agent reads browser console logs, network requests, DOM in real time
- After `odoo-update`, agent opens the instance, verifies views render, reads JS errors

### 🔮 Phase 4 — Multi-Agent Architecture (exploratory)
```
Orchestrator
     ↓
┌────┬──────────┬───────┬────────┐
Reader  Researcher  Coder  Tester
(repo   (GitHub     (edits  (SSH +
+ SSH)  + docs)     files)  browser)
```

---

## Tech Stack

- **Odoo 17** on Odoo.sh
- **Claude Code** (Anthropic)
- **SSH** with ed25519 keys
- **Python** — Odoo ORM, custom module development
- **Git** — branch-based development workflow

---

## Project Structure

```
odoo-agent/
├── CLAUDE.md           ← Agent instructions, security rules, business context
├── .env.example        ← Configuration template
├── .gitignore          ← Keeps .env and sensitive data out of the repo
├── LOG_PROGRESO.md     ← Session memory — agent reads and updates this
└── LICENSE
```

*(`docs/`, `scripts/`, and `examples/` are planned but not built yet — see Roadmap.)*

---

## Contributing

This is a personal project in active development. If you're an Odoo developer and want to adapt it to your own setup, feel free to fork it and open a PR. Feedback, issues, and ideas are welcome.

---

*By [Juan David Villasmil](https://linkedin.com/in/jvillasmil)*
