# Setup Guide — odoo-agent

Step-by-step guide to connect odoo-agent to your Odoo.sh instance.

---

## Prerequisites

- [Node.js](https://nodejs.org) v18 or higher
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- An Odoo.sh account with access to your project
- Git repository with your Odoo custom modules
- Windows (PowerShell), macOS, or Linux terminal

---

## Step 1 — Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:
```bash
Claude --version
```

---

## Step 2 — Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "your@email.com"
```

Press Enter to accept the default path. Leave passphrase empty (or set one if you prefer).

View your public key:
```bash
cat ~/.ssh/id_ed25519.pub
```

---

## Step 3 — Register SSH Key in Odoo.sh

1. Go to [odoo.sh](https://odoo.sh) → your profile (top right) → **Settings**
2. Under **SSH Keys** → **Add a key manually**
3. Paste your public key (`ssh-ed25519 ...`) and click **Add**

---

## Step 4 — Get Your SSH Connection String

1. In Odoo.sh, go to your project → select your **development branch**
2. Look for the SSH button — it shows something like:
   ```
   ssh 12345678@yourproject-dev-12345678.dev.odoo.com
   ```
3. Copy the part after `ssh ` — that's your `ODOO_SSH` value

---

## Step 5 — Test the Connection

```bash
ssh your_build_id@yourproject-dev-12345678.dev.odoo.com "pwd"
```

Expected output: `/home/odoo`

If it works, your SSH is configured correctly.

---

## Step 6 — Clone and Configure odoo-agent

**Option A — Standalone repo (recommended for portfolio/sharing):**
```bash
git clone https://github.com/jdvillasmil/odoo-agent.git
cd odoo-agent
```

**Option B — Add to your existing Odoo modules repo:**
```bash
# Copy CLAUDE.md, .env.example, .gitignore, LOG_PROGRESO.md
# to the root of your existing repo
```

Configure your environment:
```bash
cp .env.example .env
```

Edit `.env`:
```bash
ODOO_SSH=your_build_id@yourproject-dev-12345678.dev.odoo.com
ODOO_DB=yourproject-dev-12345678
MODULE_PREFIX=your_prefix
DEV_BRANCH=your_dev_branch
```

---

## Step 7 — Open Claude Code

```bash
claude
```

Claude Code will automatically read `CLAUDE.md` and `LOG_PROGRESO.md`.

Test it:
```
What branch am I on and what modules exist in this repo?
```

---

## Step 8 — Verify SSH from Claude Code

Ask Claude Code to run:
```
Run this via SSH and tell me what you see:
ssh $ODOO_SSH "ls /home/odoo/src/user/"
```

If it lists your modules, everything is connected.

---

## Monthly Staging Renewal

Odoo.sh staging instances renew monthly. When yours renews:

1. Go to Odoo.sh → your project → development branch → SSH button
2. Copy the new connection string
3. Update your `.env`:
   ```bash
   ODOO_SSH=new_build_id@yourproject-dev-newid.dev.odoo.com
   ODOO_DB=yourproject-dev-newid
   ```
4. Test with `ssh $ODOO_SSH "pwd"`

Only `.env` changes — `CLAUDE.md` and everything else stays the same.

---

## Security Best Practices

### Close Chrome when not using Claude Code
When you launch Chrome with `--remote-debugging-port=9222`, any process on your 
machine can connect to that port and control the browser.

**Rule:** Only open Chrome with the debug port when actively working with Claude Code.
Close it when you're done.

```powershell
# Open Chrome for Claude Code session
& "C:\Program Files\Google\Chrome\Application\chrome.exe" `
  --remote-debugging-port=9222 `
  --user-data-dir="C:\Users\YOUR_USER\AppData\Local\Google\Chrome\User Data\Profile 1"

# When done — close Chrome completely
```

This is a simple habit that eliminates the most immediate local security risk.

---

## Troubleshooting

**`Connection closed` immediately after connecting:**
Normal behavior — Odoo.sh doesn't open an interactive shell. Use command execution:
```bash
ssh $ODOO_SSH "your command here"
```

**`Permission denied (publickey)`:**
Your SSH key isn't registered. Repeat Step 3.

**`CONFIG NOT FOUND` in terminal:**
Unrelated to odoo-agent — this is an Oh My Posh configuration issue. Run:
```bash
oh-my-posh init pwsh | Invoke-Expression
```

**Claude Code can't find `.env`:**
Make sure `.env` is in the same directory where you run `claude`.

---

*For issues or questions → [github.com/jdvillasmil/odoo-agent/issues](https://github.com/jdvillasmil/odoo-agent/issues)*
