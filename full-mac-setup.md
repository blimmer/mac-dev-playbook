# Full Mac Setup Process

The steps that can't be (or aren't worth) automating when setting up a new
Mac, and the order to run everything in. The ordering matters: Claude Code
must not be launched until its config repo and hook dependencies are in
place.

## Phase 1: Bootstrap (no credentials required)

1. Complete the macOS setup wizard. **Name the user account `blimmer`** —
   the Claude Code hooks reference absolute `/Users/blimmer` paths.
2. Install Ansible (see [README.md](README.md)).
3. Clone this playbook over HTTPS (it's public, so no keys needed):

   ```sh
   git clone https://github.com/blimmer/mac-dev-playbook.git ~/code/mac-dev-playbook
   cd ~/code/mac-dev-playbook
   ansible-galaxy install -r requirements.yml
   ```

4. Run the playbook:

   ```sh
   ansible-playbook main.yml --ask-become-pass
   ```

   This installs Homebrew packages, casks, dotfiles, asdf, the Dock layout,
   and macOS settings. The Claude config step detects that GitHub SSH auth
   isn't set up yet, prints a warning, and skips itself — that's expected on
   the first run.

## Phase 2: GitHub auth via 1Password

1. Open 1Password (installed by the playbook) and sign in.
2. Enable the SSH agent: Settings → Developer → "Use the SSH agent" (and
   "Integrate with 1Password CLI" while you're there).
3. Verify: `ssh -T git@github.com` should greet you by username.

## Phase 3: Claude Code / agents setup

1. Re-run the private portion of the playbook:

   ```sh
   ansible-playbook main.yml --tags claude,repos
   ```

   This clones the private [claude-config](https://github.com/blimmer/claude-config)
   repo into `~/.claude` (settings, hooks, agents, skills, statusline),
   installs the uv-managed tools the hooks depend on (`nah`), and clones
   the day-one personal repos (`notes`, `scripts`, `backup`) into
   `~/code`.

2. Only now launch Claude Code and sign in. The hooks in `settings.json`
   call `~/.local/bin/nah` on nearly every tool event, so launching before
   Phase 3 completes means a broken session and a populated, unmanaged
   `~/.claude` directory.
3. Sign in to `gh` so private plugin marketplaces resolve:

   ```sh
   gh auth login
   ```

4. In Claude Code, confirm plugins load (`/plugins`) — the marketplaces
   include private ContextBridge repos.

## Phase 4: Manual sign-ins and licenses

- App Store account
- Browsers (Firefox, Chrome, Brave) and their sync accounts
- Slack workspaces
- Docker Desktop
- Tailscale
- Spotify, Obsidian sync
- iStat Menus license
- Pastebot accessibility permissions
- System Settings → Privacy & Security → Full Disk Access → enable
  terminal apps (Warp, iTerm)

## Notes

- Nothing in the playbook uninstalls software; it only ensures listed items
  are present.
- `config.yml` is the single source of truth for packages, casks, taps,
  asdf plugins, uv tools, and the Dock. When installing something new that
  should survive a machine move, add it there too.
