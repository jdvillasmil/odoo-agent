# 🤖 odoo-agent

> AI-powered development agent for Odoo 17 — automated code auditing, module analysis, and deployment via Claude Code + SSH.

---

## The Problem

Odoo development has a hidden bottleneck: **most time is not spent writing code — it's spent understanding what already exists.**

Every module you touch can break three others you don't know about. You have to read inherited models, check deprecated patterns, verify security files, and make sure you're not overwriting something a third-party module already defined. Before writing a single line, you're doing archaeology.

Manual code reviews before merging to production are slow, inconsistent, and easy to skip under pressure. Silent bugs reach production undetected.

## The Solution

An AI agent connected directly to your development environment:

- **Reads the full repository** — models, manifests, inheritance trees, dependencies
- **Connects to Odoo.sh via SSH** — runs updates, checks logs, inspects deployed files
- **Audits before you merge** — finds deprecated patterns, duplicate methods, security gaps, and inheritance risks
- **Suggests commits, never executes them** — you stay in control

One prompt. One agent. Real diagnosis in minutes.

---

## Live Demo — Branch Audit

**Prompt:**
```
Compare branches AIT vs main and audit every module not yet in production.
Check naming conventions, deprecated patterns, security files, inheritance risks.
Report with status: READY / NEEDS WORK / REVIEW REQUIRED
```

**Result in 4 minutes 22 seconds:**

| Module | Status | Blocking Issue |
|--------|--------|----------------|
| proyelec_salas | ✅ READY | None |
| proyelec_calendar_personal | ⚠️ NEEDS WORK | Empty views (WIP) |
| proyelec_sale_createdby | ⚠️ NEEDS WORK | Wrong author in manifest |
| proyelec_snp_autocomplete | ⚠️ NEEDS WORK | 3 duplicate `@constrains` + 1 duplicate `@onchange` — silent dead code |
| proyelec_so_to_po | 🔴 REVIEW REQUIRED | Double-procurement risk in `_action_confirm` + deprecated `digits=(5,2)` |

**Bugs found that would have reached production undetected:**
- A method defined **3 times** in the same class — Python silently uses only the last one
- A `_action_confirm` override triggering `_action_launch_stock_rule()` twice — risking **duplicate Purchase Orders per confirmed sale line**

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
Compare branches AIT vs main and audit every module not in main yet.
Check: naming conventions, deprecated patterns, security files, inheritance risks.
Report with status: READY / NEEDS WORK / REVIEW REQUIRED

# Check a specific module
Read proyelec_so_to_po and find any deprecated Odoo 17 patterns,
inheritance risks, or logic issues before merging to main.

# Deploy and verify
Update module proyelec_salas via SSH, then check logs for errors.
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

### Safety Rules (enforced via CLAUDE.md)
- ✅ Only operates on development branch — never touches production
- ✅ Only modifies your custom modules — third-party and native Odoo modules are read-only
- ✅ Never executes `git push`, `git merge`, or any destructive command
- ✅ Suggests commits — you always have the final call
- ✅ SSH credentials stay in `.env` — never committed to the repository

---

## Roadmap

### ✅ Phase 1 — Foundation (Complete)
- Claude Code connected to repo + Odoo.sh via SSH
- CLAUDE.md with security rules, module hierarchy, business context
- LOG_PROGRESO.md for session memory between runs
- Branch audit: finds deprecated patterns, duplicate methods, inheritance risks

### 🔄 Phase 2 — Database Access
- Python proxy script on the server that accepts ORM expressions and returns JSON
- Allows the agent to query live data without interactive shell limitations
- Example: `env['sale.order'].search_count([('state','=','sale')])`

### 📋 Phase 3 — Browser Integration
- Connect `chrome-devtools-mcp` to a dedicated Chrome profile (dev only)
- Agent reads browser console logs, network requests, DOM in real time
- After `odoo-update`, agent opens the instance, verifies views render, reads JS errors
- Closes the loop: write → deploy → verify → iterate

### 🔮 Phase 4 — Multi-Agent Architecture
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
├── docs/
│   ├── SETUP.md        ← Step-by-step connection guide for Odoo.sh
│   ├── ARCHITECTURE.md ← Multi-agent design and vision
│   └── ROADMAP.md      ← Phases, current status, what's next
├── scripts/
│   └── db_proxy.py     ← (Phase 2) ORM query proxy via SSH
└── examples/
    ├── audit-prompt.md ← Ready-to-use audit prompts
    └── sample-output.md ← Real audit output example
```

---

## Contributing

This project is in active development. If you're an Odoo developer and want to contribute or adapt it to your setup, feel free to fork it and open a PR.

Feedback, issues, and ideas are welcome.

---

*Built at Proyelec International · AIT Department · 2026*  
*By [Juan David Villasmil](https://linkedin.com/in/jvillasmil)*
