# Environment Setup Guide (Windows, from scratch)

> On a fresh **Windows 11** machine, set up this terminal workflow environment from scratch:
> **Starship (prompt) + WSL/Ubuntu + Zellij (multi-pane) + Claude Code + the agent-workflow Skill**.
> Estimated time: 30–60 minutes (depends on network). Fully local — **no SSH / remote server needed**.

> All companion files are in this package (`dotfiles/`, `.claude/skills/agent-workflow/`) — no cloud account dependency.

---

## What you'll get

- A nice **Pastel Powerline** terminal prompt (Starship + Nerd Font icons)
- A **WSL/Ubuntu** Linux environment to run Claude Code
- **Zellij** one-shot multi-pane layout (several panes in one session)
- **Claude Code** + the pre-installed **multi-agent workflow Skill**

---

## Stage 1 — Install Windows-side tools (15 min)

> All in **PowerShell** (normal privileges, unless noted).

### 1.1 Check winget

```powershell
winget --version
```
Built into Win 11. If missing, install "App Installer" from the Microsoft Store.

### 1.2 Install Starship

```powershell
winget install --id Starship.Starship -e --accept-source-agreements --accept-package-agreements
```

### 1.3 Install JetBrainsMono Nerd Font

```powershell
winget install --id DEVCOM.JetBrainsMonoNerdFont -e --accept-source-agreements --accept-package-agreements
```

### 1.4 Make Windows Terminal use that font

Open Windows Terminal → `Ctrl + ,` → select the PowerShell profile → **Appearance → Font face** → choose `JetBrainsMono Nerd Font` → save.
(Without this font, the prompt icons show as boxes / garbled.)

### 1.5 Configure the PowerShell profile

```powershell
New-Item -ItemType File -Path $PROFILE -Force | Out-Null
Set-Content -Path $PROFILE -Value @"
Invoke-Expression (&starship init powershell)
Import-Module Terminal-Icons
"@
```

### 1.6 Place the Starship config (use `dotfiles/starship.toml` from this package)

```powershell
# Replace <package-path> with where you unpacked this package
New-Item -ItemType Directory -Path "$env:USERPROFILE\.config" -Force | Out-Null
Copy-Item "<package-path>\dotfiles\starship.toml" "$env:USERPROFILE\.config\starship.toml"
```
Or skip the bundled config and generate the official preset:
```powershell
starship preset pastel-powerline -o "$env:USERPROFILE\.config\starship.toml"
```

### 1.7 Install the Terminal-Icons module

```powershell
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force -Scope CurrentUser
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
Install-Module -Name Terminal-Icons -Repository PSGallery -Scope CurrentUser -Force
```

### 1.8 Restart PowerShell to verify

Close all PowerShell windows and open a new one. You should see the Pastel Powerline prompt, and colorful icons before filenames on `ls`.

---

## Stage 2 — Install WSL + Ubuntu (10–30 min, network-dependent)

### 2.1 Open PowerShell **as Administrator**
`Win + X` → "Terminal (Admin)".

### 2.2 Install WSL + Ubuntu 24.04 LTS
```powershell
wsl --install -d Ubuntu-24.04 --web-download
```
> If it hangs at 0%: `Ctrl+C` → `wsl --update --web-download` → retry.

### 2.3 Set the UNIX username / password
After install, an Ubuntu window pops up to set a username and password. Remember the password (needed for `sudo`).

### 2.4 Verify
```bash
whoami && pwd && cat /etc/os-release | head -2
```

---

## Stage 3 — Configure the WSL internal environment (10 min)

> Work inside the **Ubuntu window**.

### 3.1 Install nvm + Node.js LTS
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install --lts && nvm use --lts && nvm alias default 'lts/*'
node --version && npm --version
```

### 3.2 Install Claude Code
```bash
npm install -g @anthropic-ai/claude-code
claude --version
```
The first `claude` run requires **logging into your Anthropic account** (browser authorization).

### 3.3 Install Zellij
```bash
sudo snap install zellij --classic
echo 'export PATH="$PATH:/snap/bin"' >> ~/.bashrc
exec bash -l
zellij --version
```

### 3.4 Place the Zellij config (use `dotfiles/zellij/` from this package)

In WSL, Windows files are at `/mnt/c/...`. Replace `<package-path>` with this package's path under Windows (e.g. `/mnt/c/Users/<your-username>/Desktop/agent-workflow-skill`):
```bash
mkdir -p ~/.config/zellij/layouts
cp "<package-path>/dotfiles/zellij/config.kdl" ~/.config/zellij/
cp "<package-path>/dotfiles/zellij/layouts/my0.kdl" ~/.config/zellij/layouts/
```

### 3.5 Configure bashrc aliases
```bash
cat >> ~/.bashrc << 'EOF'

# === workflow shortcuts ===
# Replace <project-path> with your actual project directory
alias new='cd <project-path> && zellij -s work -n my0'
alias old='cd <project-path> && zellij attach work'
EOF
source ~/.bashrc
```

---

## Stage 4 — Install the multi-agent workflow Skill (5 min)

> This is the core feature of this package. See `README.md` for details.

1. Copy the `.claude/skills/agent-workflow/` folder from this package into your **target project root**:
   ```bash
   mkdir -p <project-path>/.claude/skills
   cp -r "<package-path>/.claude/skills/agent-workflow" <project-path>/.claude/skills/
   ```
2. Start Claude Code in that project:
   ```bash
   cd <project-path> && claude
   ```
3. In Claude Code, run:
   ```
   /agent-workflow init
   ```
   It scaffolds the `.workflow/` state dir, installs the workflow protocol, and asks for your project goal.

After that, start each message with "you are executor / planner / …" to trigger the workflow. Full usage in `README.md`.

---

## Stage 5 — Acceptance Checklist

PowerShell:
- [ ] `starship --version` shows a version
- [ ] `ls` shows colorful file icons
- [ ] The prompt is in Pastel Powerline style

Ubuntu:
- [ ] `node --version` >= 18
- [ ] `claude --version` shows a version
- [ ] `zellij --version` shows a version
- [ ] `new` starts Zellij with a 3-pane layout
- [ ] Running `claude` inside Zellij opens the interactive UI (login on first run)
- [ ] `/agent-workflow init` successfully scaffolds `.workflow/`

All ✅ → setup complete 🎉

---

## Appendix: FAQ

**Q1: `new` reports "There is no active session!"**
→ The alias must use `-n my0`, not `-l my0`. See 3.5.

**Q2: `claude` command not found in WSL**
→ nvm's global packages need a shell restart: `exec bash -l` or open a new Ubuntu window.

**Q3: After Zellij starts, all panes are stuck in `/mnt/c/...`**
→ A hardcoded cwd in the layout; remove it:
```bash
sed -i '/cwd[ =]/d' ~/.config/zellij/layouts/my0.kdl
```
(The `my0.kdl` bundled here already has no such issue — no action needed.)

**Q4: Prompt icons are boxes / garbled**
→ Windows Terminal isn't using a Nerd Font; go back to 1.4.

**Q5: PowerShell profile has no effect**
→ Confirm `Get-Content $PROFILE` has the right content, then reopen the window.
