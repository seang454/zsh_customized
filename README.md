# Enhanced Zsh Setup — Complete Feature Guide

> **Quick start:** Run `bash zsh.bash install`, install a [Nerd Font](https://www.nerdfonts.com) in your terminal, then run `p10k configure`.

---

## Table of Contents

1. [How to Run the Script](#1-how-to-run-the-script)
2. [Installing New Tools](#2-installing-new-tools)
3. [Theme — Powerlevel10k](#3-theme--powerlevel10k)
4. [Tools](#4-tools)
5. [Key Bindings](#5-key-bindings)
6. [Plugins](#6-plugins)
7. [Aliases](#7-aliases)
8. [Functions](#8-functions)
9. [History & Shell Options](#9-history--shell-options)
10. [Completion System](#10-completion-system)
11. [Customization](#11-customization)
12. [Troubleshooting](#12-troubleshooting)

---

## 1. How to Run the Script

### Install everything

```bash
chmod +x zsh.bash
bash zsh.bash install
```

This installs and configures:
- Zsh + Oh My Zsh
- Powerlevel10k theme
- All plugins
- zoxide, bat, eza, delta, thefuck, tldr, fzf, gh
- All Tab completions for installed tools
- Sets Zsh as your default shell

### Remove everything

```bash
bash zsh.bash remove
```

This removes `~/.oh-my-zsh`, `~/.zshrc_custom`, `~/.p10k.zsh`, restores your `.zshrc` backup, and sets Bash as your default shell.

### After install

```bash
# Open a new terminal tab, then:
p10k configure        # set up your prompt theme interactively
exec zsh              # reload if needed
```

---

## 2. Installing New Tools

### The standard workflow (works for ~95% of tools)

```bash
# Step 1 — Install the tool
sudo apt install <toolname>
# or
pipx install <toolname>
# or
npm install -g <toolname>

# Step 2 — Register Tab completion
register_completion <toolname>

# Step 3 — Test it
<toolname> <Tab>
<toolname> --<Tab>
```

### How register_completion works

`register_completion` automatically tries **20 different completion patterns** in order until one works:

```
tool completion zsh          tool completions zsh
tool completion --shell zsh  tool completions --shell zsh
tool completion -s zsh       tool completions -s zsh
tool --completion=zsh        tool --completion zsh
tool --generate=complete-zsh tool --gen-completions=zsh
tool --gen-completions zsh   tool generate-shell-completion zsh
tool generate completion zsh tool shell-completion zsh
tool shell completion zsh    tool init completions zsh
tool complete --shell=zsh    tool complete -s zsh
tool zsh-completion          tool zsh_completion
```

This means it works automatically for **any tool** that supports completion generation — no need to know the tool's specific syntax.

### Real examples

```bash
# kubectl
sudo apt install kubectl
register_completion kubectl
kubectl <Tab>           # shows: get, apply, delete, describe...

# poetry
pipx install poetry
register_completion poetry
poetry <Tab>            # shows: add, remove, run, install...

# uv (fast Python package manager)
pipx install uv
register_completion uv  # automatically uses: uv generate-shell-completion zsh
uv <Tab>                # shows: add, remove, run, sync, lock...

# ripgrep
sudo apt install ripgrep
register_completion rg
rg --<Tab>              # shows all flags with descriptions

# gh (GitHub CLI)
register_completion gh  # automatically uses: gh completion -s zsh
gh <Tab>                # shows: repo, pr, issue, auth...

# helm
register_completion helm
helm <Tab>              # shows: install, upgrade, list, delete...
```

### If register_completion fails

Some tools have completely unique completion syntax that can't be auto-detected. For those, find the correct command manually:

```bash
<toolname> --help | grep -i complet
```

Then run it manually:

```bash
<correct command> > ~/.zsh/completions/_<toolname>
compinit_refresh
```

### Tools that always need manual setup

| Tool | Manual command |
|------|---------------|
| `terraform` | `terraform -install-autocomplete` |
| `aws` | Manual file — no generator available |
| `az` (Azure CLI) | `register-python-argcomplete az > ~/.zsh/completions/_az` |
| `gcloud` | Symlink from SDK — handled automatically on install |
| `nvm` | Handled by `zsh-nvm` plugin automatically |
| `thefuck` | Uses `eval` — no completion file needed |
| `zoxide` | Uses `eval` — no completion file needed |

### Completion management commands

| Command | What it does |
|---------|-------------|
| `register_completion <tool>` | Register one tool — tries all 20 patterns automatically |
| `compinit_refresh` | Refresh all completions for all installed tools |
| `cleanup_completions` | Remove broken/invalid completion files |

```bash
# Install one tool and register it
sudo apt install kubectl
register_completion kubectl

# After installing many tools at once
compinit_refresh

# If something is broken
cleanup_completions
```

---

## 3. Theme — Powerlevel10k

Powerlevel10k (p10k) is a feature-rich Zsh prompt with instant rendering, git awareness, and context-aware segments.

### What it shows

| Segment | Description |
|---|---|
| OS icon | Distro symbol (requires Nerd Font) |
| Directory | Current path, shortened smartly |
| Git branch & status | Branch name, ✔ clean / ✗ modified / ? untracked |
| Exit code | Red indicator when the last command failed |
| Execution time | How long the last command took (shown if > 2s) |
| Background jobs | Count of suspended jobs |
| Node / Python version | Shown when in a relevant project dir |
| Time | Current time (HH:MM) |

### Setup

```bash
p10k configure    # interactive configuration wizard
exec zsh          # reload theme after editing config
p10kconfig        # alias: open ~/.p10k.zsh in editor
```

### Nerd Font (required for icons)

Without a Nerd Font, icons render as boxes or question marks.

1. Download from [nerdfonts.com](https://www.nerdfonts.com) — recommended: **MesloLGS NF** or **JetBrainsMono Nerd Font**
2. Install the font on your system
3. Set it in your terminal emulator's font settings
4. Run `p10k configure` to verify icons render correctly

---

## 4. Tools

### zoxide — Smart cd

zoxide tracks the directories you visit and lets you jump to any of them by typing a short fragment.

```bash
z proj              # jump to best-matching dir containing "proj"
z dow               # jump to ~/Downloads
z code api          # match path containing both "code" and "api"
zi                  # fuzzy-search all known dirs and pick one (fzf)
zoxide query --list --score   # show zoxide database with scores
zoxide remove /old/path       # remove a stale entry
```

**Note:** `cd` is aliased to `z`, so your existing muscle memory works unchanged.

---

### bat — Better cat

`bat` replaces `cat` and `less` with syntax highlighting, line numbers, and git change markers.

```bash
cat file.py                         # syntax highlighted view
cat config.yaml                     # works for any file type
less long_file.log                  # paged, colored view
bat --style=plain file.py           # no line numbers or decorations
bat --language=json data.txt        # force a language
bat --line-range 10:30 file.py      # show only lines 10-30
bat -A file.txt                     # show non-printable characters
man curl                            # colorized man pages (auto-configured)

# List / change themes
bat --list-themes
export BAT_THEME="Dracula"          # add to ~/.zshrc_custom to persist
```

**Note:** On Ubuntu/Debian, bat is installed as `batcat`. The alias `bat='batcat'` is set automatically.

---

### eza — Better ls

`eza` replaces `ls` with icons, colors, git status, and tree views.

```bash
ls          # list with icons, dirs first
ll          # long list: permissions, size, date, git status
la          # long list including hidden files
lt          # tree view, 2 levels deep
lta         # tree view, 3 levels, all files including hidden
l           # compact 1-column list

eza --level=4                   # tree with 4 levels
eza --sort=size                 # sort by file size
eza --sort=modified             # sort by modified date
eza --no-icons                  # disable icons
eza --header                    # show column headers in long view
```

**Git status symbols in ll/la:**

| Symbol | Meaning |
|---|---|
| `M` | Modified |
| `A` | Added (staged) |
| `D` | Deleted |
| `R` | Renamed |
| `?` | Untracked |

---

### delta — Better git diff

`delta` is a syntax-highlighting pager for all git diff commands. Configured automatically.

```bash
git diff                    # auto-used, syntax highlighted
git diff HEAD~3
git log -p
git show abc1234
git stash show -p

# Side-by-side view
git config --global delta.side-by-side true

# Navigate between files in diff (n = next, N = previous)
# Requires delta.navigate=true (already configured)
```

---

### thefuck — Autocorrect

Watches your last command and fixes it. Type `f` or `fuck` to accept the correction.

```bash
git comit -m "fix"
f                           # → git commit -m "fix"

apt install vim
f                           # → sudo apt install vim

git push
f                           # → git push --set-upstream origin <branch>

pyhton script.py
f                           # → python script.py
```

---

### tldr — Quick man pages

Shows practical examples for any command. Much faster than `man`.

```bash
tldr tar
tldr curl
tldr git checkout
tldr rsync
tldr docker
tldr --update               # update local cache
```

---

### fzf — Fuzzy Finder

Interactive filter for any list. Powers key bindings, tab completion previews, and fuzzy functions.

```bash
ls | fzf                                          # pick from any list
fzf --preview 'bat --color=always {}'            # with file preview
ls | fzf -m                                       # multi-select (Tab to mark)
vim $(fzf)                                        # pass selection to command

# Built-in key bindings
# Ctrl+R  → fuzzy history search
# Ctrl+F  → fuzzy file picker
# Alt+C   → fuzzy cd into directory

# Double-star glob completion
cd **<Tab>          # recursive directory picker
vim **<Tab>         # recursive file picker
kill **<Tab>        # process picker
ssh **<Tab>         # host picker (from ~/.ssh/config)
```

---

## 5. Key Bindings

| Key | Action |
|---|---|
| `Ctrl+R` | Fuzzy history search (fzf powered) |
| `Ctrl+F` | Fuzzy file picker — inserts path at cursor |
| `Alt+C` | Fuzzy cd — directory tree preview via eza |
| `↑` / `↓` | History search filtered by what you've typed |
| `Ctrl+→` | Jump forward one word |
| `Ctrl+←` | Jump backward one word |
| `Ctrl+Backspace` | Delete word to the left |
| `Home` | Go to start of line |
| `End` | Go to end of line |
| `Delete` | Delete character to the right |

---

## 6. Plugins

| Plugin | What it does |
|---|---|
| `zsh-autosuggestions` | Ghost-text suggestions from history — press `→` to accept |
| `zsh-syntax-highlighting` | Colors command as you type — green = valid, red = not found |
| `zsh-history-substring-search` | Powers ↑/↓ filtered history search |
| `zsh-autopair` | Auto-closes `(`, `[`, `{`, `"`, `'` |
| `zsh-completions` | Adds hundreds of extra completion definitions |
| `fzf-tab` | Replaces completion menu with fzf picker + preview |
| `forgit` | fzf-powered interactive git commands |
| `zsh-you-should-use` | Reminds you when a shorter alias exists |
| `zsh-better-npm-completion` | Smart completion for npm scripts and package names |
| `zsh-nvm` | Lazy-loads nvm so it doesn't slow down shell startup |
| `git` | Oh My Zsh built-in: `ga`, `gco`, `gcm`, `gp`, `gl`, and ~150 more |

### forgit — interactive git

```bash
glo     # git log — browse commits with diff preview
gad     # git add — interactively stage files
gcf     # git checkout file — restore files interactively
gsp     # git stash pop — pick a stash
gbd     # git branch delete — pick branches to delete
gcb     # git checkout branch — fzf branch switcher
gdf     # git diff — pick files to diff
grh     # git reset HEAD — unstage files
```

---

## 7. Aliases

### Navigation

```bash
..          # cd ..
...         # cd ../..
....        # cd ../../..
~           # cd ~
-           # cd - (go back to previous directory)
```

### File Listing (eza)

```bash
ls          # eza --icons --group-directories-first
ll          # long list + git status
la          # long list + hidden files + git status
lt          # tree, 2 levels
lta         # tree, 3 levels, all files
l           # compact 1-column list
```

### Git

```bash
glog        # pretty graph log with hash, branch, author, relative time
gst         # git status -sb
gdiff       # git diff --stat
gwip        # git add -A && git commit -m "WIP: work in progress"
gundo       # git reset HEAD~1 --soft (undo last commit, keep changes)
gclean      # delete all local branches merged into main/master/develop

# Oh My Zsh git plugin extras
ga          # git add
gaa         # git add --all
gco         # git checkout
gcb         # git checkout -b (new branch)
gcm         # git checkout main
gd          # git diff
gp          # git push
gl          # git pull
gm          # git merge
grb         # git rebase
gsta        # git stash
gstp        # git stash pop
```

### Network

```bash
myip        # show public IP
localip     # show LAN/private IP
ports       # show listening ports (ss -tulpn)
ping        # ping -c 5 (stops after 5 packets)
```

### Disk & Process

```bash
df          # df -hT (human-readable + filesystem type)
du          # du -sh (summary of current dir)
dud         # du -d 1 -h (size of each subdirectory)
psg vim     # search running processes by name
```

### Config & Misc

```bash
zshrc       # open ~/.zshrc in editor + auto-reload
zshcustom   # open ~/.zshrc_custom in editor + auto-reload
p10kconfig  # open ~/.p10k.zsh in editor
reload      # source ~/.zshrc (reload without opening editor)

path        # print each $PATH entry on its own line
week        # print current ISO week number
timestamp   # print current datetime as 20260314_153000
pubkey      # print your SSH public key
grep        # grep --color=auto

# Safety aliases (prompt before overwriting/deleting)
rm          # rm -i
cp          # cp -i
mv          # mv -i
mkdir       # mkdir -pv (create parents, verbose)
```

---

## 8. Functions

### `mkcd` — Make directory and cd into it

```bash
mkcd ~/projects/new-thing
mkcd /tmp/test/nested/dir       # creates all parents automatically
```

### `extract` — Universal archive extractor

```bash
extract archive.tar.gz
extract backup.tar.bz2
extract data.zip
extract package.7z
extract file.rar
```

Supported: `.tar.gz` `.tar.bz2` `.tar.xz` `.tar` `.gz` `.bz2` `.zip` `.7z` `.rar` `.xz`

### `fcd` — Fuzzy directory jump

```bash
fcd                 # search from current directory
fcd ~/code          # search from ~/code
```

### `fkill` — Fuzzy kill a process

```bash
fkill               # kill -9 selected process
fkill 15            # kill -15 (SIGTERM) selected process
```

### `fgit` — Fuzzy git branch switcher

```bash
fgit                # pick a branch with commit log preview and checkout
```

### `up` — Go up N directories

```bash
up              # go up 1 level
up 3            # go up 3 levels (same as cd ../../..)
```

### `backup` — Timestamped file backup

```bash
backup nginx.conf
# → creates nginx.conf.backup.20260314_153000

backup ~/.zshrc
```

### `serve` — Quick HTTP server

```bash
serve               # http://localhost:8000
serve 3000          # http://localhost:3000
```

### `calc` — Quick calculator

```bash
calc 1920 / 1080
calc 2^32
calc 150 * 1.08
calc "sqrt(144)"
calc "scale=5; 1/3"
```

### `weather` — Terminal weather

```bash
weather                 # weather at your current IP location
weather "Bangkok"
weather "London"
```

### `sshkey` — Generate SSH key

```bash
sshkey                  # creates ~/.ssh/id_ed25519
sshkey work_github      # creates ~/.ssh/work_github
```

### `gitignore` — Fetch .gitignore template

```bash
gitignore node
gitignore python
gitignore node,react,vscode
gitignore go,linux,jetbrains > .gitignore    # save to file
```

### `compinit_refresh` — Refresh all completions

```bash
compinit_refresh        # re-register completions for all installed tools
```

Run this after installing multiple tools at once.

### `register_completion` — Register one tool's completion

```bash
register_completion kubectl         # auto-detect method
register_completion poetry          # auto-detect method
register_completion uv              # auto-detect: uses generate-shell-completion
register_completion gh              # auto-detect: uses completion -s zsh

# Manual method override (if auto fails)
register_completion mytool zsh          # force: mytool completion zsh
register_completion mytool completions  # force: mytool completions zsh
register_completion mytool generate     # force: mytool generate-shell-completion zsh
register_completion mytool shell        # force: mytool completion --shell zsh
register_completion mytool flag         # force: mytool --completion=zsh
```

### `cleanup_completions` — Remove broken completion files

```bash
cleanup_completions     # scan and remove invalid completion files
```

---

## 9. History & Shell Options

### History settings

| Setting | Value | Effect |
|---|---|---|
| `HISTSIZE` | 50,000 | Lines kept in memory |
| `SAVEHIST` | 50,000 | Lines saved to `~/.zsh_history` |
| `HIST_IGNORE_DUPS` | on | Don't save consecutive duplicates |
| `HIST_IGNORE_ALL_DUPS` | on | Remove older duplicate entries |
| `HIST_IGNORE_SPACE` | on | Commands starting with a space are not saved |
| `SHARE_HISTORY` | on | History shared across all open terminal sessions |

```bash
# Tip: prefix a command with a space to keep it out of history
 export AWS_SECRET=mysecret    # leading space = not saved
```

### Shell options

| Option | Effect |
|---|---|
| `AUTO_CD` | Type a directory name to cd into it without `cd` |
| `AUTO_PUSHD` | Every `cd` pushes to directory stack; use `popd` to go back |
| `INTERACTIVE_COMMENTS` | Allows `# comments` in interactive shell |
| `GLOB_DOTS` | `*` matches dotfiles without needing `.*` |

```bash
# AUTO_CD example
~/projects          # same as: cd ~/projects

# AUTO_PUSHD example
cd ~/a && cd ~/b && cd ~/c
popd                # back to ~/b
dirs -v             # list directory stack
```

---

## 10. Completion System

### Tab completion features

```bash
# Basic completion
git che<Tab>        # → git checkout

# fzf-tab: any completion opens fzf picker with preview
cd <Tab>            # fzf directory picker with eza preview
vim <Tab>           # fzf file picker with bat preview
kill <Tab>          # fzf process picker
ssh <Tab>           # fzf host picker (from ~/.ssh/config)
apt install <Tab>   # shows packages with flag descriptions
kubectl <Tab>       # shows subcommands

# Double-star glob
vim **<Tab>         # recursive file picker
cd **<Tab>          # recursive directory picker
```

### Completion behavior

- **Case-insensitive:** `cd proj` matches `Projects` and `projects`
- **Partial matching:** `cd pro` matches any directory starting with `pro`
- **Flag completion:** type `--` then Tab to see all flags with descriptions
- **Git-aware:** branch and file completions sorted by relevance

### Completion file locations

```bash
~/.zsh/completions/     # your registered completions live here
ls ~/.zsh/completions/  # see what's registered
```

---

## 11. Customization

All custom configuration lives in `~/.zshrc_custom`. Edit it freely — it will never be overwritten by Oh My Zsh updates.

```bash
zshcustom       # open ~/.zshrc_custom in your editor + auto-reload
```

### Add your own aliases

```bash
# At the bottom of ~/.zshrc_custom:
alias k='kubectl'
alias tf='terraform'
alias dc='docker compose'
```

### Add your own functions

```bash
# In ~/.zshrc_custom:
mkenv() {
    python3 -m venv "${1:-.venv}"
    source "${1:-.venv}/bin/activate"
}
```

### Change default editor

```bash
export EDITOR='vim'         # or 'nano', 'code', 'nvim'
export VISUAL="$EDITOR"
```

### Change bat theme

```bash
bat --list-themes | fzf --preview 'bat --theme={} --color=always ~/.zshrc'
export BAT_THEME="Dracula"  # add to ~/.zshrc_custom
```

---

## 12. Troubleshooting

### Icons show as boxes or question marks

You need a Nerd Font installed and selected in your terminal:
1. Download from [nerdfonts.com](https://www.nerdfonts.com) (recommended: MesloLGS NF)
2. Install on your system
3. Open terminal preferences and change the font
4. Run `exec zsh`

### register_completion fails for a tool

```bash
# Find the correct command
<toolname> --help | grep -i complet

# Run it manually
<correct command> > ~/.zsh/completions/_<toolname>
compinit_refresh
```

### Tab completion shows errors

```bash
cleanup_completions     # remove broken files
compinit_refresh        # rebuild cache
```

### poetry / uv / pipx not found after install

```bash
source ~/.zshrc_custom  # reload PATH
# or open a new terminal tab
```

`~/.local/bin` must be in PATH. Check:
```bash
echo $PATH | tr ':' '\n' | grep local
```

### Zsh is slow to start

```bash
zsh -i -c exit   # time a full startup
```

If `zsh-nvm` is slow, make sure nvm is not also loaded in `~/.bashrc` or elsewhere.

### thefuck / f command not found

```bash
pip3 install thefuck --break-system-packages
exec zsh
```

### zoxide not working

```bash
echo $PATH | tr ':' '\n' | grep local   # check ~/.local/bin is in PATH
export PATH="$HOME/.local/bin:$PATH"    # fix if missing
```

### p10k instant prompt warnings

Move any `echo`/`print` statements in `~/.zshrc_custom` to below the p10k block at the top of the file.

### Reinstall everything from scratch

```bash
bash zsh.bash remove
bash zsh.bash install
```

### Reinstall a single plugin

```bash
rm -rf ~/.oh-my-zsh/custom/plugins/fzf-tab
git clone --depth=1 https://github.com/Aloxaf/fzf-tab.git \
    ~/.oh-my-zsh/custom/plugins/fzf-tab
exec zsh
```

---

## Quick Reference Card

```
SCRIPT         COMMAND                    WHAT IT DOES
──────────────────────────────────────────────────────────────
               bash zsh.bash install      install everything
               bash zsh.bash remove       remove everything

INSTALL TOOL   COMMAND                    EXAMPLE
──────────────────────────────────────────────────────────────
               sudo apt install <tool>    sudo apt install kubectl
               pipx install <tool>        pipx install poetry
               npm install -g <tool>      npm install -g typescript

COMPLETIONS    COMMAND                    WHAT IT DOES
──────────────────────────────────────────────────────────────
               register_completion <tool> register one tool (auto-detects method)
               compinit_refresh           refresh all tools at once
               cleanup_completions        remove broken completion files

TOOLS          COMMAND                    WHAT IT DOES
──────────────────────────────────────────────────────────────
zoxide         z proj                     jump to best-matching "proj" dir
               zi                         interactive fzf directory picker
bat            cat file.py                syntax-highlighted view
               less file                  paged, colored
eza            ll                         long list + git status
               lt                         tree view
delta          git diff                   auto-used, syntax highlighted
thefuck        f                          fix last mistyped command
tldr           tldr tar                   practical examples

KEY BINDINGS   SHORTCUT                   ACTION
──────────────────────────────────────────────────────────────
               Ctrl+R                     fuzzy history search
               Ctrl+F                     fuzzy file picker
               Alt+C                      fuzzy cd
               ↑ / ↓                      filtered history search
               Ctrl+→ / ←                 jump word forward/backward

FUNCTIONS      USAGE                      EXAMPLE
──────────────────────────────────────────────────────────────
mkcd           mkcd dir                   mkdir + cd in one step
extract        extract file               auto-detect and extract archive
fcd            fcd                        fuzzy directory jump
fkill          fkill                      fuzzy process kill
fgit           fgit                       fuzzy git branch switch
up             up 3                       go up 3 directories
backup         backup file                timestamped file copy
serve          serve 8080                 quick HTTP server
calc           calc 2^10                  terminal calculator
weather        weather Bangkok            one-line weather
sshkey         sshkey name                generate ed25519 key
gitignore      gitignore node             fetch .gitignore template

GIT ALIASES    ALIAS                      EQUIVALENT
──────────────────────────────────────────────────────────────
               glog                       pretty graph log
               gst                        git status -sb
               gdiff                      git diff --stat
               gwip                       add -A + commit "WIP"
               gundo                      reset HEAD~1 --soft
               gclean                     delete merged branches
```