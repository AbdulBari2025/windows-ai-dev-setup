# Windows Developer Environment Setup Guide
## Python · VS Code · Git · GitHub CLI · Node.js · Claude Code · GitHub MCP
AI Development Environment on Windows using Claude Code and MCP

**Audience:** Anyone setting up a new Windows laptop for AI-assisted development  
**Time required:** 30–45 minutes  
**Tested on:** Windows 10 / Windows 11, PowerShell 5+

---

## Prerequisites

- Windows 10 (version 1809+) or Windows 11
- Admin rights on the machine
- An Anthropic account with Claude Pro, Max, Team, or Enterprise subscription
- A GitHub account

---

## Part 1 — Core Tools

### 1.1 Install Python

Download the latest Python 3.11+ from https://www.python.org/downloads/

Run the installer with these options checked:
- ✅ **Add Python to PATH** (critical — check this before clicking Install)
- ✅ Install for all users (recommended)

Verify in a new PowerShell window:
```powershell
python --version
pip --version
```

### 1.2 Install VS Code

Download from https://code.visualstudio.com/download — choose the **System Installer** (not User Installer) for Windows.

During install check:
- ✅ Add to PATH
- ✅ Register Code as editor for supported file types
- ✅ Add "Open with Code" to Explorer context menu

Verify:
```powershell
code --version
```

Install essential VS Code extensions (run in PowerShell):
```powershell
code --install-extension ms-python.python
code --install-extension ms-python.venv
code --install-extension GitHub.copilot
code --install-extension eamodio.gitlens
```

### 1.3 Install Git for Windows

Download from https://git-scm.com/download/win

During install, key options:
- Default editor: choose **Visual Studio Code**
- ✅ Add Git to PATH (default — keep it)
- Line ending: **Checkout Windows-style, commit Unix-style** (recommended)

Verify:
```powershell
git --version
```

Configure your identity (used in all commits):
```powershell
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

### 1.4 Install Node.js

Download LTS version from https://nodejs.org/en/download

Run the installer with default options. Node.js is required for Claude Code and MCP servers.

Verify:
```powershell
node --version   # should be 18+
npm --version
```

### 1.5 Install GitHub CLI

Download the Windows installer from https://cli.github.com/

Or install via winget (available on Windows 11 and updated Windows 10):
```powershell
winget install --id GitHub.cli
```

Verify:
```powershell
gh --version
```

---

## Part 2 — GitHub Authentication

### 2.1 Create a Personal Access Token (Classic)

Go to: **GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)**

Click **Generate new token (classic)**

Set:
- Note: `claude-code-dev` (or any label you recognise)
- Expiration: 90 days (or No expiration for dev machines)
- Scopes — check these:
  - ✅ `repo` (full — all sub-items)
  - ✅ `read:org`
  - ✅ `read:user`
  - ✅ `gist`

Click **Generate token** and **copy it immediately** — GitHub shows it only once.

> **Save this token** in a password manager or secure note. You will need it in Part 4 for the MCP server. It starts with `ghp_`.

### 2.2 Authenticate GitHub CLI

Run in PowerShell:
```powershell
gh auth login
```

You will be asked a series of questions — answer as follows:
```
Where do you use GitHub?          → GitHub.com
What is your preferred protocol?  → HTTPS
Authenticate Git with credentials? → Yes
How would you like to authenticate? → Login with a web browser
```

GitHub CLI will display an **8-digit one-time code** like `ABCD-1234` and open your browser automatically.

In the browser:
1. Go to https://github.com/login/device (opens automatically)
2. Enter the 8-digit code shown in your terminal
3. Click **Authorise GitHub CLI**
4. You may be asked to confirm with your GitHub password

Back in the terminal you will see:
```
✓ Authentication complete.
✓ Logged in as YourUsername
```

Verify:
```powershell
gh auth status
```

Expected output:
```
github.com
  ✓ Logged in to github.com account YourUsername
  - Active account: true
  - Token scopes: gist, read:org, read:user, repo
```

---

## Part 3 — Claude Code

### 3.1 Install Claude Code

The native installer is the recommended method — no Node.js dependency, auto-updates:

```powershell
irm https://claude.ai/install.ps1 | iex
```

Close PowerShell and open a new window so PATH updates take effect.

Verify:
```powershell
claude --version
```

> If `claude` is not recognised after reopening, add Claude's install directory to PATH manually:  
> Search **"Environment Variables"** in Start → System Properties → Environment Variables → Path → Edit → New → paste the path shown in the install output.

### 3.2 Authenticate Claude Code

Run from inside your project folder:
```powershell
cd D:\projects\YourProject
claude
```

On first launch, Claude Code opens your browser automatically and asks you to sign in to your Anthropic account (the same account as claude.ai).

Sign in with your Anthropic credentials. The browser will show a confirmation page. Return to the terminal — authentication completes automatically.

You should see the Claude Code prompt:
```
▐▛███▜▌   Claude Code v2.x.xxx
▝▜█████▛▘  Sonnet 4.6 · Claude Pro
  ▘▘ ▝▝    D:\projects\YourProject
```

---

## Part 4 — Clone Project from GitHub

### 4.1 Clone your repository

```powershell
cd D:\projects
gh repo clone YourUsername/YourRepoName
cd YourRepoName
```

Or using git directly:
```powershell
git clone https://github.com/YourUsername/YourRepoName.git
cd YourRepoName
```

### 4.2 Set up Python virtual environment

```powershell
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

Verify the venv is active — your prompt should show `(.venv)`:
```
(.venv) PS D:\projects\YourRepoName>
```

---

## Part 5 — GitHub MCP Server for Claude Code

The MCP server lets Claude Code read issues, create PRs, check commits, and interact with your GitHub repo directly from the terminal.

### 5.1 Install the GitHub MCP package globally

```powershell
npm install -g @modelcontextprotocol/server-github
```

> Note: This package is deprecated as of April 2025 but remains functional. It uses the standard GitHub REST API with your PAT — no Copilot subscription needed.

### 5.2 Add GitHub MCP to Claude Code config

The `claude mcp add` CLI has known Windows parsing bugs with `npx -y` and JSON flags. The most reliable method is to edit the config file directly.

**Open the config file in VS Code:**
```powershell
code $env:USERPROFILE\.claude.json
```

Find the end of the file (just before the final closing `}`) and add the `mcpServers` block. If a `mcpServers` key already exists, add the `github` entry inside it. If it does not exist, add the whole block:

```json
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx"
      }
    }
  }
```

Replace `ghp_xxxx` with the classic PAT you created in Part 2.1.

> The `cmd /c` wrapper is required on Windows — it routes the npx call through cmd.exe, bypassing the PowerShell argument parsing bug that causes `error: unknown option '-y'`.

**Validate the JSON before saving:**

Use **Ctrl+Shift+P → Format Document** in VS Code. If VS Code reports a JSON syntax error, fix it before saving (usually a missing comma or mismatched bracket).

### 5.3 Verify MCP connection

Restart Claude Code, then run inside the Claude Code session:
```
/mcp
```

Expected output:
```
github · ✔ connected · 26 tools
```

If it shows `✘ Failed to connect`, check:
1. Your PAT has `repo`, `read:org`, `read:user` scopes
2. The PAT has not expired
3. `npm list -g @modelcontextprotocol/server-github` confirms global install

### 5.4 Test GitHub MCP is reading your repo

Ask Claude Code:
```
Show me the open issues in my repo
```

A response listing issues (or confirming none exist) means everything is working.

---

## Part 6 — CLAUDE.md Project File

Every project should have a `CLAUDE.md` file in its root. Claude Code reads this automatically at session start — it is Claude's permanent memory about your project.

Create it in your project root:
```powershell
code CLAUDE.md
```

Minimum contents:
```markdown
# Project Name

## What this is
Brief description. Stack: Python, ...

## Frozen files — never modify without explicit instruction
- path/to/frozen_file.py

## Rules
- No deletions unless explicitly asked
- Always incremental — new code alongside existing
- Run tests after any change to core logic
- Commit format: [FEATURE|FIX|TEST] short description

## Project structure
- src/          — core source code
- tests/        — test suite
- config/       — configuration

## Common commands
pip install -r requirements.txt
python -m pytest tests/ -v

## Phase status
Phase 1: COMPLETE
Phase 2: IN PROGRESS
```

---

## Part 7 — Personal Access Token for Kaggle Notebooks

When you move work to a Kaggle notebook (for GPU access), you need the same GitHub PAT to clone your private repo.

**Add the PAT as a Kaggle Secret:**

1. Go to https://www.kaggle.com/settings/connected-accounts — verify GitHub is connected
2. Open any Kaggle notebook → **Add-ons → Secrets → Add Secret**
3. Name: `GITHUB_TOKEN`
4. Value: your `ghp_xxxx` PAT

In the notebook, access it:
```python
from kaggle_secrets import UserSecretsClient
token = UserSecretsClient().get_secret("GITHUB_TOKEN")

import subprocess
subprocess.run([
    "git", "clone",
    f"https://{token}@github.com/YourUsername/YourRepoName.git"
])
```

> Use the same classic PAT with `repo` scope — it works identically from Kaggle.

---

## Quick Reference — Daily Commands

```powershell
# Start a session
cd D:\projects\YourProject
.venv\Scripts\activate
claude

# Inside Claude Code
/mcp                          # check MCP server connections
/status                       # confirm CLAUDE.md loaded, git branch

# Git via GitHub CLI
gh repo view                  # repo summary
gh issue list                 # open issues
gh pr list                    # open pull requests
gh pr create --title "title"  # create PR for current branch

# Common git
git status
git add .
git commit -m "[PHASE2] description"
git push
git pull
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `claude` not recognised after install | Open new PowerShell window; PATH updates require restart |
| `error: unknown option '-y'` in mcp add | Edit `~\.claude.json` directly using the cmd wrapper approach in Part 5.2 |
| GitHub MCP `✘ Failed to connect` | Check PAT scopes (`repo`, `read:org`, `read:user`) and expiry |
| `api.githubcopilot.com` returns 401 | That endpoint requires Copilot subscription — use the npm package approach instead |
| `(.venv)` not showing in prompt | Run `.venv\Scripts\activate` from project root |
| VS Code Python interpreter wrong | Ctrl+Shift+P → **Python: Select Interpreter** → choose `.venv` |
| `pip install` fails with permission error | Never use `sudo` on Windows; ensure venv is activated first |
| Claude Code auth loop | Run `claude logout` then `claude` again to re-authenticate |
